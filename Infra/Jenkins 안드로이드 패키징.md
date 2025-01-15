
### TL; DR

1. Jenkins는 JAVA 11 지원 X -> JDK 17
   

```bat
@echo off
setlocal enabledelayedexpansion

:: Java 11 Setting -
set JAVA_HOME=C:\Program Files\Android\Android Studio\jre
set PATH=%JAVA_HOME%\bin;%PATH%
set JAVA_EXE="%JAVA_HOME%\bin\java.exe"
:: Gradle 설정
set GRADLE_USER_HOME=%USERPROFILE%\.gradle_java11
:: Java 옵션 설정
set _JAVA_OPTIONS=-Djava.specification.version=11

:: 기본 경로 설정
set ENGINE_PATH=D:\Develop\GolfzonMEngine
set PROJECT_PATH=C:\Users\psyche95\Desktop\jenkins_build\GolfzonM\U2Client
set PROJECT_NAME=U2Client
set OUTPUT_DIR=C:\Users\psyche95\Desktop\Golfzon\M_Packaging

:: Android SDK 및 NDK 설정
set ANDROID_HOME=C:\Users\psyche95\AppData\Local\Android\Sdk
set ANDROID_NDK_PATH=%ANDROID_HOME%\ndk\25.2.9519653
set NDKROOT=%ANDROID_NDK_PATH%

:: 출력 폴더 설정
for /f "tokens=2 delims==" %%I in ('wmic os get localdatetime /value') do set TIMESTAMP=%%I
set NEW_OUTPUT_DIR=%OUTPUT_DIR%\Build_Android_%TIMESTAMP:~0,8%_%TIMESTAMP:~8,6%

:: 필요한 디렉토리 생성
if not exist "%NEW_OUTPUT_DIR%" mkdir "%NEW_OUTPUT_DIR%"

:: 프로젝트 파일 확인
if not exist "%PROJECT_PATH%\%PROJECT_NAME%.uproject" (
    echo Error: Project file not found!
    exit /b 1
)

echo Starting Android packaging for %PROJECT_NAME%...

:: Project Build
call "%ENGINE_PATH%\Engine\Build\BatchFiles\RunUAT.bat" ^
BuildCookRun ^
-nocompileeditor ^
-installed -nop4 ^
-project=%PROJECT_PATH%\%PROJECT_NAME%.uproject ^
-cook -stage -archive -archivedirectory="%NEW_OUTPUT_DIR%" ^
-package ^
-ue4exe="%ENGINE_PATH%\Engine\Binaries\Win64\UE4Editor-Cmd.exe" ^
-compressed ^
-ddc=InstalledDerivedDataBackendGraph ^
-pak ^
-prereqs ^
-targetplatform=Android -cookflavor=ASTC ^
-build ^
-CrashReporter ^
-target=%PROJECT_NAME% ^
-clientconfig=Development ^
-utf8output ^
-AndroidSDK="%ANDROID_HOME%" ^
-AndroidNDK="%NDKROOT%"

if %ERRORLEVEL% neq 0 (
    echo Error: Build failed with exit code %ERRORLEVEL%
    exit /b %ERRORLEVEL%
)

echo Android packaging completed successfully.
echo Output directory: %NEW_OUTPUT_DIR%
exit /b 0

```

### RunUAT 인자 역할

- `-nocompileeditor`: 에디터 컴파일을 건너뜀
- `-installed`: 엔진 바이너리로 빌드
- `-nop4`: Perforce 연동을 비활성화
- `-project`: 프로젝트 파일의 경로를 지정
- `-cook`: 콘텐츠 쿠킹을 수행
- `-stage`: 패키징된 게임을 스테이징 디렉토리로 복사
- `-archive`: 패키지된 게임을 아카이브
- `-archivedirectory`: 아카이브 디렉토리를 지정
- `-package`: 게임을 패키징
- `-ue4exe`: 사용할 UE4 Editor 실행 파일의 경로를 지정
- `-compressed`: 패키지를 압축
- `-ddc`: 사용할 파생 데이터 캐시(DDC) 설정을 지정

	- DDC는 파생 데이터 캐시를 의미합니다. 이는 에셋 처리 결과를 저장하는 캐시 시스템입니다. 예를 들어, 텍스처의 압축 버전이나 머티리얼의 컴파일된 버전 등이 여기에 저장됩니다. DDC를 사용하면 에셋 처리 시간을 줄이고, 여러 사용자 간에 처리된 데이터를 공유할 수 있습니다.

- `-pak`: 콘텐츠를 PAK 파일로 패키징
- `-prereqs`: 필요한 선행 요구사항을 포함
- `-targetplatform`: 대상 플랫폼을 Android로 설정
- `-cookflavor`: 쿠킹 설정을 ASTC(텍스처 압축 형식)로 지정
- `-build`: 코드를 빌드
- `-CrashReporter`: 크래시 리포터를 포함
- `-target`: 빌드할 프로젝트 이름을 지정
- `-clientconfig`: 클라이언트 구성을 Development or Shipping
- `-utf8output`: 출력을 UTF-8로 인코딩
- `-AndroidSDK`: Android SDK 경로를 지정
- `-AndroidNDK`: Android NDK 경로를 지정
### Issue
#### Incredibuild 사용시 패키징 실패
```
 Execution of commandlet took:  123.49 seconds
  LogShaderCompilers: Display: === FShaderJobCache stats ===
  LogShaderCompilers: Display: Total job queries 0, among them cache hits 0 (0.00%)
  LogShaderCompilers: Display: Tracking 0 distinct input hashes that result in 0 distinct outputs (0.00%)
  LogShaderCompilers: Display: RAM used: 0.00 MB (0.00 GB) of 1638.40 MB (1.60 GB) budget. Usage: 0.00%
  LogShaderCompilers: Display: ================================================
  LogShaderCompilers: Display: Shaders left to compile 0
  LogShaderCompilers: Display: Shaders left to compile 0
  OptickLog: Display: OptickPlugin UnLoaded!
  LogHttp: Display: cleaning up 0 outstanding Http requests.
  LogContentStreaming: Display: There are 1 unreleased StreamingManagers
Took 140.1294547s to run UE4Editor-Cmd.exe, ExitCode=0
********** COOK COMMAND COMPLETED **********
********** BUILD COMMAND STARTED **********
Running: D:\Develop\GolfzonMEngine\Engine\Binaries\DotNET\UnrealBuildTool.exe U2Client Android Development -Project=C:\Users\psyche95\Desktop\jenkins_build\GolfzonM\U2Client\U2Client.uproject -Manifest=C:\Users\psyche95\Desktop\jenkins_build\GolfzonM\U2Client\Intermediate\Build\Manifest.xml -nobuilduht -NoHotReload -xgeexport  C:\Users\psyche95\Desktop\jenkins_build\GolfzonM\U2Client\U2Client.uproject -NoUBTMakefiles  -remoteini="C:\Users\psyche95\Desktop\jenkins_build\GolfzonM\U2Client" -skipdeploy -log="C:\Windows\system32\config\systemprofile\AppData\Roaming\Unreal Engine\AutomationTool\Logs\D+Develop+GolfzonMEngine\UBT-U2Client-Android-Development.txt"
  Engine Directory:D:\Develop\GolfzonMEngine\Engine
  Project Directory:C:\Users\psyche95\Desktop\jenkins_build\GolfzonM\U2Client
  Adjust SDK found in C:\Users\psyche95\Desktop\jenkins_build\GolfzonM\U2Client\Plugins\Adjust\Source\Adjust\../ThirdParty/Android
  Engine Directory:D:\Develop\GolfzonMEngine\Engine
  Project Directory:C:\Users\psyche95\Desktop\jenkins_build\GolfzonM\U2Client
  Engine Directory:D:\Develop\GolfzonMEngine\Engine
  Project Directory:C:\Users\psyche95\Desktop\jenkins_build\GolfzonM\U2Client
  D:\Develop\GolfzonMEngine\Engine\Source\Runtime\Engine\Engine.Build.cs: warning: Referenced directory 'D:\Develop\GolfzonMEngine\Engine\Source\Launch\Public' does not exist.
  PLATFORM_ANDROID_NDK_VERSION = 250300
  NDK toolchain: r25c, NDK version: 33, GccVersion: 4.9, ClangVersion: 14.0.7
  Parsing headers for U2Client
    Running UnrealHeaderTool "C:\Users\psyche95\Desktop\jenkins_build\GolfzonM\U2Client\U2Client.uproject" "C:\Users\psyche95\Desktop\jenkins_build\GolfzonM\U2Client\Intermediate\Build\Android\U2Client\Development\U2Client.uhtmanifest" -LogCmds="loginit warning, logexit warning, logdatabase error" -Unattended -WarningsAsErrors -abslog="C:\Windows\system32\config\systemprofile\AppData\Roaming\Unreal Engine\AutomationTool\Logs\D+Develop+GolfzonMEngine\UHT-U2Client-Android-Development.txt" -installed
  LogInit: Display: Loading text-based GConfig....
  Reflection code generated for U2Client in 3.4027725 seconds
  Compiling Native 64-bit code with NDK API 'android-33'
  Writing manifest to C:\Users\psyche95\Desktop\jenkins_build\GolfzonM\U2Client\Intermediate\Build\Manifest.xml
  XGEEXPORT: Exported 'D:\Develop\GolfzonMEngine\Engine\Intermediate\Build\UBTExport.000.xge.xml'
  Total execution time: 6.14 seconds
Took 6.2589684s to run UnrealBuildTool.exe, ExitCode=0
Running: C:\Program Files (x86)\Incredibuild\xgConsole.exe "C:\Windows\system32\config\systemprofile\AppData\Roaming\Unreal Engine\AutomationTool\Logs\D+Develop+GolfzonMEngine\UAT_XGE.xml" /Rebuild /NoLogo /ShowAgent /ShowTime /no_watchdog_thread
  Fatal Error: File not found: C:\Windows\system32\config\systemprofile\AppData\Roaming\Unreal Engine\AutomationTool\Logs\D+Develop+GolfzonMEngine\UAT_XGE.xml
Took 0.0520074s to run xgConsole.exe, ExitCode=3
BUILD FAILED: Command failed (Result:3): C:\Program Files (x86)\Incredibuild\xgConsole.exe "C:\Windows\system32\config\systemprofile\AppData\Roaming\Unreal Engine\AutomationTool\Logs\D+Develop+GolfzonMEngine\UAT_XGE.xml" /Rebuild /NoLogo /ShowAgent /ShowTime /no_watchdog_thread. See logfile for details: 'xgConsole-2024.07.31-10.14.49.txt'
AutomationTool exiting with ExitCode=1 (Error_Unknown)
Error: AutomationTool execution failed. Check log: C:\Windows\system32\config\systemprofile\UnrealEngine\BuildTemp\UAT_Log.txt
Build step 'Execute Windows batch command' marked build as failure
Finished: FAILURE
```

인크레디빌드 사용시에 패키징 에러가 발생함.
로그에서 실패한 이유가 File Not found인데 실제 경로에 가보니 해당 파일이 있었다.
그래서 권한을 주는 등 다양한 방법을 적용해봤으나 계속해서 같은 이유로 실패했다.

인크레디빌드 Agent를 끄고 패키징을 하면 Incredibuild 관련 부분을 통과할 수 있었다. 또는 RunUAT 실행 인자에 -noxge를 추가해도 Incredibuild가 꺼져서 관련 부분을 통과할 수 있었다. 

이를 통해, Incredibuild 관련 이슈라 판단하고 incredibuild support에 질문함.

![[inc_support.png]]

윈도우 젠킨스 설치시에 Windows Service에 등록된다.

젠킨스를 이용해서 빌드를 할 때 Window Service에 등록된 사용자로 빌드를 하는데 기본 값이 SYSTEM 사용자다. 그래서 이를 바꿔줘야한다.

WIN + R를 누르고 services.msc를 입력해서 서비스를 연다.

Jenkins Service를 돌릴 윈도우 계정이 필요한데 회사 도메인을 이용한 사내아이디를 사용 해야했다.
```
Jenkins - 속성 - 로그온 - 찾아보기
```


아이디 검색 - 비밀번호 입력하고 젠킨스 중지

C:\\Program Files\\Jenkins 로 이동해서 jenkins.xml을 수정한다. xml을 열어서 <arguments\> 있는 곳에 다음 값을 추가한다. (젠킨스 서비스가 시작할 때 domain 아이디로 로그인 하겠다고 알려줘야함)

```xml
<arguments>-Xrs -Xmx256m -Dhudson.lifecycle=hudson.lifecycle.WindowsServiceLifecycle -jar "C:\Program Files\Jenkins\jenkins.war" --httpPort=12000 --webroot="%ProgramData%\Jenkins\war --serviceLogonAccount={yourdomain\yourid} --serviceLogonPassword={your_passward}"</arguments>
```

domain과 id를 하는 법은 cmd를 열고 whoami 입력

![[whoami.png]]

젠킨스 다시 시작하고 젠킨스 빌드시에 Incredibuild에서 에러나는 부분을 넘어갈 수 있다.


### Gradle 문제 (JAVA 버전)

```

  
====2024-07-31 오후 2:17:26====PERFORMING FINAL APK PACKAGE OPERATION=====-arm64===========================================
Copied file C:\Users\psyche95\Desktop\jenkins_build\GolfzonM\U2Client\Intermediate\Android\arm64\gradle\app\src\main\jniLibs\arm64-v8a\libUE4.so.
Copied file C:\Users\psyche95\Desktop\jenkins_build\GolfzonM\U2Client\Intermediate\Android\arm64\gradle\app\src\main\assets\main.obb.png.
[FirebaseGoodies] Crashlytics debug symbols upload enabled. Adding native libraries...

Creating rungradle.bat to work around commandline length limit (using unused drive letter Z:)
Making .apk with Gradle...
To honour the JVM settings for this build a single-use Daemon process will be forked. See [https://docs.gradle.org/7.5/userguide/gradle_daemon.html#sec:disabling_the_daemon](https://docs.gradle.org/7.5/userguide/gradle_daemon.html#sec:disabling_the_daemon).
Daemon will be stopped at the end of the build

FAILURE: Build failed with an exception.

* Where:
Build file 'Z:\build.gradle' line: 14

* What went wrong:
A problem occurred evaluating root project 'app'.
> Could not open dsl generic class cache for script 'Z:\buildscriptAdditions.gradle' (C:\Users\psyche95\.gradle\caches\7.5\scripts\cp2i11tnu00nybyfrpbih872t).
   > BUG! exception in phase 'semantic analysis' in source unit '_BuildScript_' Unsupported class file major version 65

* Try:
> Run with --stacktrace option to get the stack trace.
> Run with --info or --debug option to get more log output.
> Run with --scan to get full insights.


Deprecated Gradle features were used in this build, making it incompatible with Gradle 8.0.

You can use '--warning-mode all' to show the individual deprecation warnings and determine if they come from your own scripts or plugins.

See [https://docs.gradle.org/7.5/userguide/command_line_interface.html#sec:command_line_warnings](https://docs.gradle.org/7.5/userguide/command_line_interface.html#sec:command_line_warnings)
* Get more help at [https://help.gradle.org](https://help.gradle.org/)

BUILD FAILED in 4s
ERROR: cmd.exe failed with args /c "C:\Users\psyche95\Desktop\jenkins_build\GolfzonM\U2Client\Intermediate\Android\arm64\gradle\rungradle.bat" :app:assembleDebug
       (see C:\Users\psyche95\AppData\Roaming\Unreal Engine\AutomationTool\Logs\D+Develop+GolfzonMEngine\Log.txt for full exception trace)
AutomationTool exiting with ExitCode=1 (Error_Unknown)
BUILD FAILED
Error: Build failed with exit code 1
Build step 'Execute Windows batch command' marked build as failure
Finished: FAILURE
```

젠킨스 설정시에 java 버전을 21로 올렸기 때문에 발생하는 에러다. 

현재 gradle 7.5를 사용하고 있는데 gradle 7.5가 지원하는 상방은 java 18이다. 그래서 패키징시에 명시적으로 java 11 path를 넣고 패키징을 했다.

```bat

:: batch 파일 상단

:: Java 11 Setting -
set JAVA_HOME=C:\Program Files\Android\Android Studio\jre
set PATH=%JAVA_HOME%\bin;%PATH%
set JAVA_EXE="%JAVA_HOME%\bin\java.exe"
:: Gradle 설정
set GRADLE_USER_HOME=%USERPROFILE%\.gradle_java11
:: Java 옵션 설정
set _JAVA_OPTIONS=-Djava.specification.version=11

```

