자동화 시스템 구축

## Jenkins 설치

1. 젠킨스에서 Java 11은 2024.07.31까지 지원 Deprecated 될 예정
2. OpenJDK 21로 구성 완료

## 설정 이슈 및 해결 과정

### 플러그인 다운로드, 업데이트 안됨

#### sun.security.provider.certpath.SunCertPathBuilderException

```plain-text

sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target
  at java.base/sun.security.provider.certpath.SunCertPathBuilder.build(SunCertPathBuilder.java:148)
  at java.base/sun.security.provider.certpath.SunCertPathBuilder.engineBuild(SunCertPathBuilder.java:129)
  at java.base/java.security.cert.CertPathBuilder.build(CertPathBuilder.java:297)
  at java.base/sun.security.validator.PKIXValidator.doBuild(PKIXValidator.java:383)
Caused: sun.security.validator.ValidatorException: PKIX path building failed
  at java.base/sun.security.validator.PKIXValidator.doBuild(PKIXValidator.java:388)
  at java.base/sun.security.validator.PKIXValidator.engineValidate(PKIXValidator.java:271)
  at java.base/sun.security.validator.Validator.validate(Validator.java:256)
  at java.base/sun.security.ssl.X509TrustManagerImpl.checkTrusted(X509TrustManagerImpl.java:230)
  at java.base/sun.security.ssl.X509TrustManagerImpl.checkServerTrusted(X509TrustManagerImpl.java:132)
  at java.base/sun.security.ssl.CertificateMessage$T13CertificateConsumer.checkServerCerts(CertificateMessage.java:1302)
Caused: javax.net.ssl.SSLHandshakeException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target
  at java.base/sun.security.ssl.Alert.createSSLException(Alert.java:130)
  at java.base/sun.security.ssl.TransportContext.fatal(TransportContext.java:378)
  at java.base/sun.security.ssl.TransportContext.fatal(TransportContext.java:321)
  at java.base/sun.security.ssl.TransportContext.fatal(TransportContext.java:316)
  at java.base/sun.security.ssl.CertificateMessage$T13CertificateConsumer.checkServerCerts(CertificateMessage.java:1318)
  at java.base/sun.security.ssl.CertificateMessage$T13CertificateConsumer.onConsumeCertificate(CertificateMessage.java:1195)
  at java.base/sun.security.ssl.CertificateMessage$T13CertificateConsumer.consume(CertificateMessage.java:1138)
  at java.base/sun.security.ssl.SSLHandshake.consume(SSLHandshake.java:393)
  at java.base/sun.security.ssl.HandshakeContext.dispatch(HandshakeContext.java:476)
  at java.base/sun.security.ssl.HandshakeContext.dispatch(HandshakeContext.java:447)
  at java.base/sun.security.ssl.TransportContext.dispatch(TransportContext.java:201)
  at java.base/sun.security.ssl.SSLTransport.decode(SSLTransport.java:172)
  at java.base/sun.security.ssl.SSLSocketImpl.decode(SSLSocketImpl.java:1506)
  at java.base/sun.security.ssl.SSLSocketImpl.readHandshakeRecord(SSLSocketImpl.java:1421)
  at java.base/sun.security.ssl.SSLSocketImpl.startHandshake(SSLSocketImpl.java:455)
  at java.base/sun.security.ssl.SSLSocketImpl.startHandshake(SSLSocketImpl.java:426)
  at java.base/sun.net.www.protocol.https.HttpsClient.afterConnect(HttpsClient.java:586)
  at java.base/sun.net.www.protocol.https.AbstractDelegateHttpsURLConnection.connect(AbstractDelegateHttpsURLConnection.java:187)
  at java.base/sun.net.www.protocol.http.HttpURLConnection.followRedirect0(HttpURLConnection.java:2909)
  at java.base/sun.net.www.protocol.http.HttpURLConnection.followRedirect(HttpURLConnection.java:2818)
  at java.base/sun.net.www.protocol.http.HttpURLConnection.getInputStream0(HttpURLConnection.java:1929)
  at java.base/sun.net.www.protocol.http.HttpURLConnection.getInputStream(HttpURLConnection.java:1599)
  at java.base/sun.net.www.protocol.https.HttpsURLConnectionImpl.getInputStream(HttpsURLConnectionImpl.java:223)
  at hudson.model.UpdateCenter$UpdateCenterConfiguration.download(UpdateCenter.java:1321)
Caused: java.io.IOException: Failed to load https://updates.jenkins.io/download/plugins/mailer/470.vc91f60c5d8e2/mailer.hpi to C:\ProgramData\Jenkins\.jenkins\plugins\mailer.jpi.tmp
  at hudson.model.UpdateCenter$UpdateCenterConfiguration.download(UpdateCenter.java:1332)
Caused: java.io.IOException: Failed to download from https://updates.jenkins.io/download/plugins/mailer/470.vc91f60c5d8e2/mailer.hpi (redirected to: https://get.jenkins.io/plugins/mailer/470.vc91f60c5d8e2/mailer.hpi)
  at hudson.model.UpdateCenter$UpdateCenterConfiguration.download(UpdateCenter.java:1366)
  at hudson.model.UpdateCenter$DownloadJob._run(UpdateCenter.java:1923)
  at hudson.model.UpdateCenter$InstallationJob._run(UpdateCenter.java:2235)
  at hudson.model.UpdateCenter$DownloadJob.run(UpdateCenter.java:1897)
  at java.base/java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:572)
  at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:317)
  at hudson.remoting.AtmostOneThreadExecutor$Worker.run(AtmostOneThreadExecutor.java:121)
  at java.base/java.lang.Thread.run(Thread.java:1583)
```

suncertpathbuilderexception unable to find valid certification path to requested target를 인터넷에 검색했을 때 -Https를 사용하는 웹사이트에 연결을 시도할 때 Java에서 신뢰하는 인증서 목록에 해당 웹사이트의 인증서가 존재하지 않아서 발생하는 문제라고 한다.

그래서 예전에 전달받은 인증서를 jdk(jre)/lib/securites/cacert에 keytool 명령어를 이용해 현재 사용하는 자바인증서에 인증서를 넣었음.


```
keytool -importcert -alias {alias name} -keystore "C:\Program Files\Java\jdk-21.0.2\lib\security\cacerts" -storepass changeit -file C:\Users\psyche95\Desktop\CERT\{.cer}
```

suncertpathbuilderexception unable to find valid certification path to requested target는 해결 되었으나 아래의 문제가 발생
#### java.security.InvalidAlgorithmParameterException: the trustAnchors parameter must be non-empty

```
java.security.InvalidAlgorithmParameterException: the trustAnchors parameter must be non-empty
```

검색하니, openjdk에서 발생할 수 있다고 함 -> oracle jdk의 cacert로 교체하면 된다고 해서 교체 -> 같은 에러 반복 cacert 문제가 아님을 확인

환경변수 JAVA_HOME 확인-> java8 사용하고 있었음. 
JAVA_HOME java21로 변경

이로써 플러그인 업데이트 가능하게 됨. + 배치파일 작성 후 빌드 가능

그래도 계속 안된다면, java cacert에서 회사 사내 인증서 없애고 JAVA_HOME 설정 다시 할 것

```
java.security.InvalidAlgorithmParameterException: the trustAnchors parameter must be non-empty
  
	at java.base/java.security.cert.PKIXParameters.setTrustAnchors(PKIXParameters.java:200)
  
	at java.base/java.security.cert.PKIXParameters.<init>(PKIXParameters.java:120)
  
	at java.base/java.security.cert.PKIXBuilderParameters.<init>(PKIXBuilderParameters.java:104)
  
	at java.base/sun.security.validator.PKIXValidator.<init>(PKIXValidator.java:94)
  
Caused: java.lang.RuntimeException: Unexpected error
```

해결한 줄 알았는데, 다시 발생해서 재설치 진행함.

jdk 21은 젠킨스에 도입된 지 얼마 되지 않아서 jdk 17로 교체.

sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target

그냥 check certificate 하지 않는 게 정신건강에 이로울 거 같아서 skip-certificate-check 플러그인을 hpi로 받음. 그러고나서 C:\ProgramData\Jenkins\.jenkins\plugins에 직접 hpi를 넣고 설치 젠킨스 재시작

이로써 해결 완료되었고 현재까지는 잘 작동함.


#### auto commit

Unreal Engine4로 작업을 하고 있는데, ugs를 사용하지 않아 개발자가 주기적으로 binary를 depot에 올려줘야 하는 상황 (perforce helix core 사용)

아래 batch 파일을 통해 일정 시간에 컴파일하고 만들어진 binary를 submit 함.


```
@echo off
setlocal enabledelayedexpansion

set ENGINE_PATH=C:\Users\owner\Perforce\psyche95_GZ-PSYCHE9503_8014\GolfzonMEngine
set PROJECT_PATH=C:\Users\owner\Perforce\psyche95_GZ-PSYCHE9503_8014\WaveM\U2Client_Wave

:: 접근 권한 설정
icacls "%ENGINE_PATH%" /grant Everyone:(F)
icacls "%PROJECT_PATH%" /grant Everyone:(F)

:: 필요한 폴더와 파일 삭제
echo Deleting unnecessary files and folders...
if exist "%PROJECT_PATH%\.vs" rmdir /s /q "%PROJECT_PATH%\.vs"
if exist "%PROJECT_PATH%\DerivedDataCache" rmdir /s /q "%PROJECT_PATH%\DerivedDataCache"
if exist "%PROJECT_PATH%\Intermediate" rmdir /s /q "%PROJECT_PATH%\Intermediate"
if exist "%PROJECT_PATH%\Saved" rmdir /s /q "%PROJECT_PATH%\Saved"
if exist "%PROJECT_PATH%\U2Client.sln" del /f /q "%PROJECT_PATH%\U2Client.sln"

:: Unreal Engine을 이용해 비주얼 스튜디오 프로젝트 파일 생성
echo Generating Visual Studio project files...
"%ENGINE_PATH%\Engine\Binaries\DotNET\UnrealBuildTool.exe" -projectfiles -project="%PROJECT_PATH%\U2Client.uproject" -game -progress
if %errorlevel% neq 0 (
    echo Failed to generate Visual Studio project files.
    curl -X POST -H "Content-type: application/json" --data "{\"text\":\"Failed to generate Visual Studio project files\", \"channel\":\"#dev_build\"}" "https://hooks.slack.com/services/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"
    exit /b %errorlevel%
)

:: 비주얼 스튜디오를 이용해 U2Client.sln 빌드
echo Building the U2Client.sln...
"C:\Program Files (x86)\Microsoft Visual Studio\2019\Professional\Common7\IDE\devenv.com" "%PROJECT_PATH%\U2Client.sln" /Build "Development Editor|Win64" /Project "U2Client"
echo Errorlevel after build: %errorlevel%
if %errorlevel% neq 0 (
    echo Build process failed, checking log...
    curl -X POST -H "Content-type: application/json" --data "{\"text\":\"Build process failed\", \"channel\":\"#dev_build\"}" "https://hooks.slack.com/services/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"
    exit /b %errorlevel%
)

:: 변경된 PDB 및 DLL 파일 검사 및 Perforce로 체크아웃
pushd "%PROJECT_PATH%\Binaries\Win64"
set CHANGES_EXIST=
for %%f in (*.pdb *.dll) do (
    :: 파일 접미사 체크 및 변경 검사
    echo %%f | findstr /R /C:".*-[0-9][0-9][0-9][0-9]\." > nul
    if errorlevel 1 (
        :: 파일이 마지막으로 변경된 시간 가져오기
        for /f "tokens=2 delims==" %%t in ('wmic datafile where name^="%%f" get lastmodified /value') do set LASTMOD=%%t
        :: Perforce에 저장된 파일의 수정 시간 비교
        p4 -u psyche95 fstat -T headTime %%f | findstr /C:"headTime"
        set HEADTIME=!headTime!
        if "!LASTMOD:~0,14!" neq "!HEADTIME!" (
            set CHANGES_EXIST=true
            p4 -u psyche95 edit %%f
        )
    )
)
popd

:: 변경점이 있으면 제출하고, 없으면 되돌리기
if defined CHANGES_EXIST (
    echo Changes detected. Submitting to Perforce...
    p4 -u psyche95 submit -d "automate submit"
) else (
    echo No changes detected. Reverting changes...
    pushd "%PROJECT_PATH%\Binaries\Win64"
    for %%f in (*.pdb *.dll) do (
        p4 -u psyche95 revert %%f
    )
    popd
)

echo Finished: SUCCESS

```
