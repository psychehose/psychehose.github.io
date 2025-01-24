#unreal #error 
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

### 해결 방법

1. 프로젝트 세팅에서 비동기화 로딩 스레드 끔 -> 앱 크래쉬는 나지 않으나, 게임스레드가 모든 Asset을 로딩해서 속도 이슈가 있을 수 있음 
2. ApplicationHasEnteredForegroundDelegate를 사용하지 말기(?) 

IOSAppDelegate.cpp
1. applicationDidEnterBackground
2. applicationWillEnterForeground
3. applicationDidBecomeActive

가 존재합니다. 각각 백그라운드로 갔을 때, Foreground로 돌아올 때 그리고 화면이 완전히 Active 되었을 때 호출되고 이는 각자 순서대로 호출함

1. FIOSCoreDelegates::ApplicationWillEnterBackgroundDelegate
2. FIOSCoreDelegates::ApplicationHasEnteredForegroundDelegate
3. FIOSCoreDelegates.OnDidBecomeActive

`ApplicationHasEnteredForegroundDelegate`를 사용하지 말고 `OnDidBecomeActive` 하면됨.
