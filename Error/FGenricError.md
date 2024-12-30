## auto FGenericPlatformMisc::RequestExit(bool)::$_27::operator()



2c90

```
          Crashed: Thread
0  libsystem_kernel.dylib         0xc2ec __pthread_kill + 8
1  libsystem_pthread.dylib        0x7c0c pthread_kill + 268
2  libsystem_c.dylib              0x75ba0 abort + 180
3  U2Client                       0x2417084 auto FGenericPlatformMisc::RequestExit(bool)::$_27::operator()<FLogCategoryLogGenericPlatformMisc, char16_t [31], bool>(FLogCategoryLogGenericPlatformMisc const&, char16_t const (&) [31], bool const&) const + 653 (GenericPlatformMisc.cpp:653)
4  U2Client                       0x24d2e54 FIOSErrorOutputDevice::Serialize(char16_t const*, ELogVerbosity::Type, FName const&) + 31 (IOSErrorOutputDevice.cpp:31)
5  U2Client                       0x254cf6c FOutputDevice::LogfImpl(char16_t const*, ...) + 178 (UnrealMemory.h:178)
6  U2Client                       0x250e55c FDebug::ProcessFatalError() + 482 (AssertionMacros.cpp:482)
7  U2Client                       0x277e668 UObjectBase::~UObjectBase() + 212 (UObjectArray.h:212)
8  U2Client                       0x26d9534 FAsyncPurge::TickDestroyGameThreadObjects(bool, float, double) + 366 (GarbageCollection.cpp:366)
9  U2Client                       0x26d8f3c FAsyncPurge::TickPurge(bool, float, double) + 489 (GarbageCollection.cpp:489)
10 U2Client                       0x26ce4b4 IncrementalPurgeGarbage(bool, float) + 1783 (GarbageCollection.cpp:1783)
11 U2Client                       0x4626ef4 UEngine::ConditionalCollectGarbage() + 1378 (IConsoleManager.h:1378)
12 U2Client                       0x421ca50 UWorld::Tick(ELevelTick, float) + 1691 (LevelTick.cpp:1691)
13 U2Client                       0x41064a4 UGameEngine::Tick(float, bool) + 224 (CoreGlobals.h:224)
14 U2Client                       0x106c3c4 FEngineLoop::Tick() + 4902 (LaunchEngineLoop.cpp:4902)
15 U2Client                       0x27e4410 -[IOSAppDelegate MainAppThread:] + 429 (IOSAppDelegate.cpp:429)
16 Foundation                     0xde428 __NSThread__start__ + 732
17 libsystem_pthread.dylib        0x606c _pthread_start + 136
18 libsystem_pthread.dylib        0x10d8 thread_start + 8

```


9f40

```
          Crashed: Thread
0  libsystem_kernel.dylib         0x7674 __pthread_kill + 8
1  libsystem_pthread.dylib        0x71ac pthread_kill + 268
2  libsystem_c.dylib              0x20c8c abort + 180
3  U2Client                       0x2417084 auto FGenericPlatformMisc::RequestExit(bool)::$_27::operator()<FLogCategoryLogGenericPlatformMisc, char16_t [31], bool>(FLogCategoryLogGenericPlatformMisc const&, char16_t const (&) [31], bool const&) const + 653 (GenericPlatformMisc.cpp:653)
4  U2Client                       0x24d2e54 FIOSErrorOutputDevice::Serialize(char16_t const*, ELogVerbosity::Type, FName const&) + 31 (IOSErrorOutputDevice.cpp:31)
5  U2Client                       0x254cf6c FOutputDevice::LogfImpl(char16_t const*, ...) + 178 (UnrealMemory.h:178)
6  U2Client                       0x250e55c FDebug::ProcessFatalError() + 482 (AssertionMacros.cpp:482)
7  U2Client                       0x2624598 FAsyncLoadingThread::FlushLoading(int) + 7003 (AsyncLoading.cpp:7003)
8  U2Client                       0x2787174 LoadPackageInternal(UPackage*, char16_t const*, unsigned int, FLinkerLoad*, FArchive*, FLinkerInstancingContext const*) + 1147 (UObjectGlobals.cpp:1147)
9  U2Client                       0x2785ddc LoadPackage(UPackage*, char16_t const*, unsigned int, FArchive*, FLinkerInstancingContext const*) + 1469 (UObjectGlobals.cpp:1469)
10 U2Client                       0x2784960 ResolveName(UObject*&, FString&, bool, bool, unsigned int, FLinkerInstancingContext const*) + 791 (UObjectGlobals.cpp:791)
11 U2Client                       0x2785fe4 StaticLoadObjectInternal(UClass*, UObject*, char16_t const*, char16_t const*, unsigned int, UPackageMap*, bool, FLinkerInstancingContext const*) + 853 (UObjectGlobals.cpp:853)
12 U2Client                       0x2774930 StaticLoadObject(UClass*, UObject*, char16_t const*, char16_t const*, unsigned int, UPackageMap*, bool, FLinkerInstancingContext const*) + 928 (UObjectGlobals.cpp:928)
13 U2Client                       0x278662c StaticLoadClass(UClass*, UObject*, char16_t const*, char16_t const*, unsigned int, UPackageMap*) + 1322 (UObjectGlobals.h:1322)
14 U2Client                       0x1dc2248 UUIScreenController::CreatePopupUI(EU2UIScreenLayerType, FString, bool) + 1332 (UObjectGlobals.h:1332)
15 U2Client                       0x1dc285c UUIScreenController::OpenPopupUI(FString, bool) + 294 (UIScreenController.cpp:294)
16 U2Client                       0x1dc2950 UUIScreenController::OpenPopupUI(FString, FAnchors const&, FVector2D const&, bool) + 306 (UIScreenController.cpp:306)
17 U2Client                       0x1bf67d8 UUIErrorPopup* UUIScreenController::OpenPopup<UUIErrorPopup>(EUIComponentPopup, bool) + 149 (UIScreenController.h:149)
18 U2Client                       0x1dcea48 UU2NetworkManager::MessageManagerEnum(EU2MsgType, UMsgC_Base*) + 105 (U2NetworkManager.cpp:105)
19 U2Client                       0x1e5f708 TBaseUObjectMethodDelegateInstance<false, UManagerObject, void (EU2MsgType, UMsgC_Base*), FDefaultDelegateUserPolicy>::ExecuteIfSafe(EU2MsgType, UMsgC_Base*) const + 598 (DelegateInstancesImpl.h:598)
20 U2Client                       0x1e5f8a8 UE4Function_Private::TFunctionRefCaller<UMessageManager::SendThreadSafeMsgManager(EU2MsgType, UMsgC_Base*)::$_49, void ()>::Call(void*) + 955 (DelegateSignatureImpl.inl:955)
21 U2Client                       0x1079fdc TGraphTask<TFunctionGraphTaskImpl<void (), (ESubsequentsMode::Type)0>>::ExecuteTask(TArray<FBaseGraphTask*, TSizedDefaultAllocator<32>>&, ENamedThreads::Type) + 681 (Function.h:681)
22 U2Client                       0x23f7be0 FNamedTaskThread::ProcessTasksUntilIdle(int) + 711 (TaskGraph.cpp:711)
23 U2Client                       0x2bc2d9c FlushRenderingCommands(bool) + 1264 (RenderingThread.cpp:1264)
24 U2Client                       0x27eb754 invocation function for block in FIOSApplication::OrientationChanged(UIInterfaceOrientation) + 465 (SharedPointerInternals.h:465)
25 U2Client                       0x24d1fcc -[FIOSAsyncTask CheckForCompletion] + 69 (IOSAsyncTask.cpp:69)
26 U2Client                       0x24d2044 +[FIOSAsyncTask ProcessAsyncTasks] + 107 (IOSAsyncTask.cpp:107)
27 U2Client                       0x27e424c -[IOSAppDelegate MainAppThread:] + 297 (CoreGlobals.h:297)
28 Foundation                     0x5b518 __NSThread__start__ + 716
29 libsystem_pthread.dylib        0x16cc _pthread_start + 148
30 libsystem_pthread.dylib        0xba4 thread_start + 8
```

```
          FAsyncLoadingThread
0  libsystem_kernel.dylib         0x1268 __semwait_signal + 8
1  libsystem_c.dylib              0x57d8 nanosleep + 220
2  libsystem_c.dylib              0x64a4 usleep + 68
3  U2Client                       0x261e12c FAsyncLoadingThread::Run() + 4772 (AsyncLoading.cpp:4772)
4  U2Client                       0x2440214 FRunnableThreadPThread::Run() + 25 (PThreadRunnableThread.cpp:25)
5  U2Client                       0x24236a8 FRunnableThreadPThread::_ThreadProc(void*) + 186 (PThreadRunnableThread.h:186)
6  libsystem_pthread.dylib        0x16cc _pthread_start + 148
7  libsystem_pthread.dylib        0xba4 thread_start + 8
```


```
          FAsyncPurge
0  libsystem_kernel.dylib         0x167c __psynch_cvwait + 8
1  libsystem_pthread.dylib        0x806c _pthread_cond_wait + 1232
2  U2Client                       0x2418fc4 FPThreadEvent::Wait(unsigned int, bool) + 443 (GenericPlatformProcess.cpp:443)
3  U2Client                       0x26d9134 FAsyncPurge::Run() + 537 (GarbageCollection.cpp:537)
4  U2Client                       0x2440214 FRunnableThreadPThread::Run() + 25 (PThreadRunnableThread.cpp:25)
5  U2Client                       0x24236a8 FRunnableThreadPThread::_ThreadProc(void*) + 186 (PThreadRunnableThread.h:186)
6  libsystem_pthread.dylib        0x16cc _pthread_start + 148
7  libsystem_pthread.dylib        0xba4 thread_start + 8
```

Tip.FAsyncPurge는 언리얼엔진에서 가비지 컬렉션을 담당하는 비동기 스레드 -더 이상 사용되지 않는 객체들을 식별하고 정리(Purge)함


#### LoadClass(StaticLoadObject) 과정에서 FlushLoading이 필요한 이유

1. 비동기 로딩 프로세스:
```cpp
StaticLoadObject
-> LoadPackage  // 패키지 로딩 시작
-> AsyncLoading 큐에 작업 등록
-> FlushLoading // 로딩 완료 대기
-> 로딩된 객체 반환
```

2. FlushLoading이 필요한 이유:
- LoadClass는 동기(Synchronous) 함수임 - 호출한 즉시 결과(UClass*)를 반환해야 함
- 하지만 실제 리소스 로딩은 비동기로 처리
- 따라서 비동기 로딩이 완료될 때까지 기다려야 함
- 이 "기다림"을 위해 FlushLoading 사용

3. FlushLoading의 역할:
```cpp
void FAsyncLoadingThread::FlushLoading(int32 PackageId)
{
    // 1. 현재 진행중인 비동기 로딩 작업들이 완료될 때까지 대기
    // 2. 로딩된 객체들의 초기화/링킹 작업
    // 3. 참조 관계 설정
    // 4. 모든 작업이 완료되면 리턴
}
```

4. 만약 FlushLoading이 없다면:
```cpp
UClass* LoadedClass = LoadClass(...);  // 비동기 로딩 시작
// 이 시점에서 실제 로딩이 안 된 상태로 반환될 수 있음
CreateWidget(LoadedClass);  // 크래시! 아직 로딩 안 된 Class 사용 시도
```
UE의 FlushLoading은:

큐에 있는 모든 비동기 로딩 작업을 처리할 때까지 대기
각 작업의 완료를 보장
모든 작업이 완료될 때까지 호출 스레드를 블록

즉, "Flush"는 파이프라인이나 큐에 있는 모든 작업을 강제로 처리 완료시키는 동작을 의미
즉, FlushLoading은 비동기 로딩 시스템에서 동기적 결과가 필요할 때 사용되는 "동기화 포인트(Synchronization Point)"
이는 UE의 리소스 로딩 시스템에서 필수적인 메커니즘





1. 크래쉬 발생 경로 

    ```
    UUIScreenController::CreatePopupUI()
    -> StaticLoadClass() 
    -> LoadPackage()
    -> FAsyncLoadingThread::FlushLoading()
    -> Fatal Error (Cannot Flush Async Loading while async loading is suspended)
    ```

    ```
    상황:
    1. ALT Status: Suspended (작업은 있지만 실행 못하는 상태)
    [Task1] -> [Task2] -> [LoadClass Task] -> [Task4]
    하지만 실행 불가 (Suspended)
    
    2. CreatePopupUI에서 LoadClass 호출
    -> StaticLoadObject 
    -> LoadPackage
    -> FlushLoading 호출
   
   1. 교착 상태 발생:
   FlushLoading: "모든 작업이 완료될 때까지 기다린다"
   ALT: "Suspended 상태라 작업을 처리할 수 없다"
   결과: ⚠️ 영원히 대기하게 됨
    ```


2. 이유
   
   AsyncLoadingThread가 suspend 상태일 때 LoadClass를 호출
   LoadClass는 내부적으로 리소스 로딩을 위해 FlushLoading 호출
   UE는 ALT가 Suspend 상태일 때 FlushLoading을 명시적으로 금지
   ```cpp
   UE_CLOG(IsAsyncLoadingSuspendedInternal(), LogStreaming, Fatal, 
        TEXT("Cannot Flush Async Loading while async loading is suspended (%d)"), 
        GetAsyncLoadingSuspendedCount());
   ```

3. UE가 이를 명시적으로 금지하는 이유

    교착 상태 방지

    리소스 로딩의 안전성 보장

    명확한 에러 메시지로 개발자에게 문제 상황 전달


4. 해결방안

    ALT 상태를 먼저 체크하고 Suspend 상태일 때 지연 로딩을 하기