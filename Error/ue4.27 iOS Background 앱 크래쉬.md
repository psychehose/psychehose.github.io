
### 순서

1. 게임을 실행하고 플레이한다.
2. 앱을 백그라운드로 전환한다.
3. 일정 시간 지난 후에 (시간은 각각 다양합니다.) 앱을 켭니다.
4. 앱 크래쉬 발생


##### 배경 지식
* 렌더 스레드를 Flush 하고 suspend 하는 생성자
* 생성자 - 렌더링 스레드를 없애고 재생성할 지, 단순 일시정지인 지 결정

```cpp
// 렌더링 스레드가 일시 중단되는 동안 렌더링 명령 대기열을 시작하지 않도록 비동기 로드 스레드를 일시 중단합니다.
if (IsAsyncLoadingMultithreaded())
{
	SuspendAsyncLoading();
}

```

Async Loading Thread는 렌더링에 필요한 리소스를 비동기적으로 로딩하고 사용하지 않는 리소스들을 언로드한다.

앱이 백그라운드 전환시에 렌더링 스레드 일시 중지 -> 비동기 로딩 중단 -> 비동기 로딩 기다림 -> 크래쉬

### 앱 죽는 이유 분석

앱이 Background 갈 때 FAppEntry::Suspend(true) -> 스레드 중지  
앱이 Foreground로 다시 돌아와서 팝업을 열어야 하는데 Async Loading Thread가 중지되어 있는 상태

팝업을 여는 코드에서 Load Package를 하는 부분이 있는데 이때 ALT를 이용. 따라서 ALT가 돌지 않고 메인스레드는 ALT를 기다리고 있기 때문에 Stall 발생

### 해결 방법?

1. UE5 5.3.2 git commit 적용 -> 앱 시작하자마자 크래쉬 -> 프로젝트 세팅에서 비동기화 로딩 스레드 끔 -> 앱 크래쉬는 나지 않으나, 게임스레드가 모든 Asset을 로딩해서 속도 이슈가 있을 수 있음 (골엠은 맵이 크고 그래서 켜는 게 더 나을 것 같음, 그리고 기존에 켠 이유가 있다고 하셔서)  그리고 커밋을 제안한 버전이 ue5.1

2. ApplicationHasEnteredForegroundDelegate를 사용하지 말기(?) 

**Step 1: app goes to background****
1) **FCoreDelegates::ApplicationWillDeactivateDelegate.Broadcast();
2) FCoreDelegates::ApplicationWillEnterBackgroundDelegate.Broadcast();
3) Suspend: RenderThread/ALT/GameThread

**Step 2: app going to foreground**
1) FCoreDelegates::ApplicationHasEnteredForegroundDelegate.Broadcast();    //"Fatal error!" if FlushAsyncLoading is called. Because ALT is still being suspended
2) Resume: RenderThread/ALT/GameThread
3) FCoreDelegates::ApplicationHasReactivatedDelegate.Broadcast();

Is it standard rule to not call any asset loading in `FCoreDelegates::ApplicationHasEnteredForegroundDelegate.Broadcast(); `

- applicationDidBecomeActive에서 FAppEntry::Resume(true)가 있음. -> package를 로드하려면 여기에서 해야할 것 같은 느낌.

IOSAppDelegate.cpp에  

1. applicationDidEnterBackground
2. applicationWillEnterForeground
3. applicationDidBecomeActive

가 존재합니다. 각각 백그라운드로 갔을 때, Foreground로 돌아올 때 그리고 화면이 완전히 Active 되었을 때 호출되고 이는 각자  

1. FIOSCoreDelegates::ApplicationWillEnterBackgroundDelegate
2. FIOSCoreDelegates::ApplicationHasEnteredForegroundDelegate
3. FIOSCoreDelegates.OnDidBecomeActive

를 브로드캐스팅 합니다.골엠에서 GSInstance에 이 Delegate에 바인딩을 걸어놓고 있는데 팝업을 여는게 ApplicationHasEnteredForegroundDelegate, ApplicationHasReactivatedDelegate에 걸려있습니다. (NetworkManager의 bBackgroud를 바꿔서)의심 정황: 이 부분에서 atomic하게 처리 되지 않아서 쓰레드가 살아나지 않고 packageload를 하면 죽고, 약간의 지연이 발생해서 쓰레드가 살아나면 package load하면 튕기지 않는 것 같습니다.그래서 ApplicationHasEnteredForegroundDelegate를 사용하지 않고 OnDidBecomeActive를 사용하면 해결될 가능성이 있는 것 같습니다.