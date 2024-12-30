

insight를 위한 명령어를 넣어줘어야 함.

ADB로 Trace를 위한 포트 오픈 (1980)

리버스로 해야 들어온다. 포워드로 하면 반대로 가겠지

Unreal Insight 실행해서 AutoStart 설정해두면 게임 실행시 Insight 프로세스가 자동실행된다.

쭉 추적하고, Trace Sessions에 들어온 Insight 파일로 프로파일링 진행한다.

```cmd
UE4Editor.exe "D:\Develop\GolfzonM\U2Client\U2Client.uproject" -game -WINDOWED -ResX=1280 -ResY=720 -trace=cpu,gpu,memory,log,frame -engine="D:\Develop\GolfzonMEngine\Engine"
```

PowerShell로 할 경우 인자 추가가 안됨 (cpu,gpu...) -> Cmd로 할 것




이 명령어는 언리얼 엔진 4에서 Unreal Insights를 사용하기 위한 실행 인자들입니다. 각 인자에 대해 설명

1. -NetTrace=1
   네트워크 트레이싱을 활성화합니다.

2. -trace=cpu,frame,gpu,memory,log,counters
   트레이스할 데이터 유형을 지정합니다. 여기서는 CPU, 프레임, GPU, 메모리, 로그, 카운터 데이터를 수집합니다.

3. -tracehost=127.0.0.1
   트레이스 데이터를 전송할 호스트 IP를 지정합니다. 여기서는 로컬호스트를 사용합니다.

4. -Messaging
   메시징 시스템을 활성화합니다. Unreal Insights와의 통신에 필요합니다.

5. -SessionId=
   세션 ID를 지정합니다. 여기서는 비어 있어 자동 생성됩니다.

6. -SessionOwner=""
   세션 소유자의 이름을 지정합니다.

7. -SessionName=""
   세션의 이름을 지정합니다. 여기서는 비어 있어 자동 생성됩니다.

8. -iterative
   반복적 빌드 모드를 활성화합니다. 개발 중 빠른 재컴파일을 위해 사용됩니다.

9. -fileopenlog
   파일 열기 작업에 대한 로깅을 활성화합니다.

10. -tracefile=PathToSaveTraceTo.utrace
    트레이스 데이터를 저장할 파일 경로를 지정합니다. .utrace 확장자를 사용합니다.

이 명령어는 Unreal Insights를 사용하여 상세한 성능 데이터를 수집하고 분석하는 데 사용됩니다. CPU, GPU, 메모리 사용량, 프레임 정보 등 다양한 성능 지표를 추적하며, 네트워크 활동도 모니터링합니다. 

개발자 이름(chin)을 지정하고 있어, 팀 환경에서 누가 어떤 세션을 실행했는지 추적하는 데 유용할 수 있습니다. 또한 트레이스 파일을 지정된 경로에 저장하므로, 나중에 이 데이터를 Unreal Insights 툴에서 분석할 수 있습니다.

언리얼 엔진 4에서 Unreal Insights와 관련된 추가적인 명령 인자들을 알려드리겠습니다. 이 목록은 주요 인자들을 포함하고 있지만, 모든 인자를 망라하지는 않을 수 있습니다.

1. -TraceGameThread=1
   게임 스레드의 트레이싱을 활성화합니다.

2. -TraceRenderThread=1
   렌더링 스레드의 트레이싱을 활성화합니다.

3. -TraceAudioThread=1
   오디오 스레드의 트레이싱을 활성화합니다.

4. -TraceTasks=1
   태스크 시스템의 트레이싱을 활성화합니다.

5. -TraceMemory=1
   메모리 할당 및 해제의 트레이싱을 활성화합니다.

6. -TraceUI=1
   사용자 인터페이스 관련 활동의 트레이싱을 활성화합니다.

7. -TraceFrames=1
   프레임 정보의 트레이싱을 활성화합니다.

8. -TraceAnim=1
   애니메이션 시스템의 트레이싱을 활성화합니다.

9. -TraceLoadTime=1
   자산 로딩 시간의 트레이싱을 활성화합니다.

10. -TraceRHI=1
    렌더링 하드웨어 인터페이스(RHI) 호출의 트레이싱을 활성화합니다.

11. -InsightsBufferSize=
    Insights 트레이스 버퍼의 크기를 설정합니다. (예: -InsightsBufferSize=100)

12. -statnamedevents
    네임드 이벤트에 대한 통계를 활성화합니다.

13. -NoInsightsConsole
    Insights 콘솔 출력을 비활성화합니다.

14. -InsightsLatencyCompensation=
    네트워크 지연 보상 값을 설정합니다. (밀리초 단위)

15. -InsightsCollectorPort=
    Insights 콜렉터가 사용할 포트 번호를 지정합니다.

16. -InsightsCollectorHost=
    Insights 콜렉터의 호스트 주소를 지정합니다.

17. -TraceBookmarks=1
    북마크 이벤트의 트레이싱을 활성화합니다.

18. -TraceLiveInterval=
    라이브 트레이싱 업데이트 간격을 설정합니다. (초 단위)

19. -TraceMaxFileSize=
    트레이스 파일의 최대 크기를 설정합니다. (MB 단위)

20. -TraceScreenshots=1
    스크린샷 캡처를 트레이스에 포함시킵니다.
