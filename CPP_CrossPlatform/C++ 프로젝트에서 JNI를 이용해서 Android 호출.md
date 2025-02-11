
C++ 기반의 프로젝트에서 안드로이드 함수를 사용 해야하는 경우가 있다. 안드로이드 게임을 타겟으로 하는 언리얼 엔진 프로젝트가 예시이다.

어떻게 하면 C++ 프로젝트에서 안드로이드 개발자가 정의한 함수를 호출할 수 있을까? JNI를 이용하면 된다. JNI는 Java Native Interface의 준말로 안드로이드 앱(자바/코틀린)과 네이티브 코드(C/C++) 사이를 연결해주는 인터페이스다. C++ 기반 코드에서 `JNIEnv*`라는 환경 포인터를 이용하여, 자바 클래스를 찾고 메서드를 호출하거나, 자바 타입과 C++ 타입 간 데이터를 변환할 수 있다.

### 안드로이드 라이브러리 빌드

C++ 코드베이스에서 JNI를 이용해서 Android 플랫폼 코드를 호출 하기 위해서 먼저 안드로이드 라이브러리를 빌드 해야한다. aar은 Android Archive package의 준말이다. 자바 코드만 포함할 수 있는jar과 다르게 aar은 안드로이드 리소스 파일(레이아웃 등)과 매니페스트 파일과 C++ Library를 포함하수 있다.

1. 안드로이드 스튜디오를 이용해서 Java / Kotlin으로 로직을 작성한다.

2. AGP 버전,  gradle 버전, 안드로이드 필수 라이브러리 호환성 체크

3. gradle을  이용해서 안드로이드 라이브러리(aar) 빌드

```bash
$ ./gradlew --version # gradle 버전
$ ./gradlew :AndroidUtil:assembleRelease # build/output 폴더로 라이브러리 빌드
```

### 언리얼엔진과 통합

언리얼엔진과 안드로이드 라이브러리 파일(aar)을 통합하기 위해서는 JNI를 이용 해야 한다. 프로젝트에 바로 추가하는 것보다 플러그인에 추가하는 것이 낫다.

* 플러그인 모듈을 만들고 `$(PluginDir)/../..Resource/Library/Android`에  빌드한 aar을 넣는다.
*  PluginDir에 `{Module_Name}_UPL.xml`을 생성한다.
*  `{Module_Name}_UPL.xml` 작성
	* `PrivateDependencyModuleNames.Add("Launch");` 필수 
		* Launch 모듈에 언리얼 JNI 헬퍼 함수들이 구현되어 있음.

```csharp
// MyModule.Build.cs
using UnrealBuildTool;
using System.IO;
public class MyModule : ModuleRules
{
	public MyModule(ReadOnlyTargetRules Target) : base(Target)
	{
		// ... //
		string PluginPath = Utils.MakePathRelativeTo(ModuleDirectory, Target.RelativeEnginePath)
		if (Target.Platform == UnrealTargetPlatform.Android) 
		{
			// ... // 

			PrivateDependencyModuleNames.Add("Launch");
			AdditionalPropertiesForReceipt.Add("AndroidPlugin", Path.Combine(PluginPath), "MyModule_UPL.xml")
		}
	}
}


```

* UPL을 작성한다.
	* 안드로이드 패키징을 할 때 aar 파일을 안드로이드 앱에 복사함
	* 복사한 aar을 dependency로 설정함
	* GameActivity에 aar에 있는 package를 import 함
	* 안드로이드 매니페스트 업데이트 (권한)

```xml
<?xml version="1.0" encoding="utf-8"?>  
<!--  
  Copyright (c) 2024 Hose  
-->  
<root xmlns:android="[http://schemas.android.com/apk/res/android">
	<init>  
		<log text="Android UPL initialization" />  
	</init>
	  
	<prebuildCopies>  
	<copyFile src="$S(PluginDir)/../../Resources/Library/Android/AndroidUtil-release.aar" dst="$S(BuildDir)/gradle/app/libs/AndroidUtil-release.aar" />  
	<log text="Copy Success" />  
	</prebuildCopies>

<!--
	프로젝트의 빌드 스크립트 자체를 구성하는 데 필요한 종속성을 정의 
	gradle version 지정
-->
  <buildscriptGradleAdditions>
		<insert>  
	      dependencies {  
		      classpath 'com.android.tools.build:gradle:7.4.2'  
	      }  
		</insert>  
	</buildscriptGradleAdditions>
	
<!--
	앱의 종속성을 정의
-->
	<buildGradleAdditions>  
		<insert>  
	      dependencies {  
		      implementation files('libs/AndroidUtil-release.aar')  
	      }  
		</insert>  
	</buildGradleAdditions>

  <!-- GameActivity에서 네이티브 메서드를 인식하도록 설정 -->  
	<gameActivityImportAdditions>  
		<insert>
		    import com.psychehose.androidutil.network.NativeNetworkManager;  
		</insert>  
	</gameActivityImportAdditions>

	<gradleProperties>  
		<insert>  
			android.useAndroidX=true  
		    android.enableJetifier=true
		</insert>  
	</gradleProperties>

	<proguardAdditions>  
		<insert>  
 
		</insert>  
	</proguardAdditions>

	<androidManifestUpdate>  
		<addPermission android:name="android.permission.ACCESS_WIFI_STATE"/>  
		<addPermission android:name="android.permission.ACCESS_NETWORK_STATE"/>  
		<addPermission android:name="android.permission.CHANGE_WIFI_STATE"/>  
		<addPermission android:name="android.permission.CHANGE_NETWORK_STATE"/>  
		<addPermission android:name="android.permission.ACCESS_COARSE_LOCATION"/>  
		<addPermission android:name="android.permission.ACCESS_FINE_LOCATION"/>  
		<addPermission android:name="android.permission.NEARBY_WIFI_DEVICES"  
		             android:usesPermissionFlags="neverForLocation"  
		             android:minSdkVersion="33"/>  
	</androidManifestUpdate>  
</root>

```


라이브러리 내에 있는 AndroidManifest.xml에서 권한 처리를 한다면 `<androidManifestUpdate>`를 생략하면 된다.


#### C++ 코드베이스에서 Android 함수 호출

안드로이드는 자바 또는 코틀린으로 구현되어 있어서 JVM 위에서 구동된다. 그래서 C++ 코드베이스에서 Java 환경을 먼저 가져오고 함수를 호출 해야한다.

1. JNIEnv를 가져온다.
2. 실행하고자 하는 Java Class를 찾는다.
3. Java Class에서 Java method를 찾는다.(jmethod에 캐싱)
4. Java Class를 통해서 instance를 생성한다. (jobject에 캐싱)
5. jobject와 jmethod를 이용해서 호출


아래 static 함수 signature

```java
public static NativeNetworkManager getInstance(Context context) {
    if (instance == null) {
        synchronized (NativeNetworkManager.class) {
            if (instance == null) {
                instance = new NativeNetworkManager(context);
            }
        }
    }
	return instance;
}
private static native void nativeCallbackMessage(); // callback을 위한
  
public void helloMessage() {  
    nativeCallbackMessage();  
}
```


```cpp

UWifiManager::UWifiManager()
{
}
 
UWifiManager::~UWifiManager()
{
	JNIEnv* Env = FAndroidApplication::GetJavaEnv();
	Env->DeleteGlobalRef(NativeNetworkManagerObject);
}
 
void UWifiManager::Init()
{
	JNIEnv* Env = FAndroidApplication::GetJavaEnv();
	if (Env == nullptr)
	{
		UE_LOG(LogTemp, Log, TEXT("Env is nullptr"));
		return;
	}

	// class를 가져옴 (instance가 아님을 주의)
	NativeNetworkManagerClass = FAndroidApplication::FindJavaClass("com/psychehose/androidutil/network/NativeNetworkManager");
	if (!NativeNetworkManagerClass)
	{
		UE_LOG(LogTemp, Error, TEXT("Failed to find the NativeNetworkManager class."));
		return;
	}
 
	if (Env->ExceptionCheck())
	{
		Env->ExceptionDescribe();
		Env->ExceptionClear();
		UE_LOG(LogTemp, Error, TEXT("Exception occurred after finding class"));
		return;
	}

	// 안드로이드의 화면 - Activity
	jobject Context = FAndroidApplication::GetGameActivityThis();

	// Static Method를 찾고 변수에 캐싱함
	jmethodID alternativeMethod = Env->GetStaticMethodID(NativeNetworkManagerClass, "getInstance",
		"(Landroid/content/Context;)Lcom/psychehose/androidutil/network/NativeNetworkManager;");
 
	if (!alternativeMethod)
	{
		UE_LOG(LogTemp, Error, TEXT("Failed to find the alternativeMethod getInstance method."));
	}
	if (Env->ExceptionCheck())
	{
		Env->ExceptionDescribe();
		Env->ExceptionClear();
		UE_LOG(LogTemp, Error, TEXT("Exception occurred after finding method"));
	}
 
	jmethodID NativeNetworkManagerGetInstanceMethod = FJavaWrapper::FindStaticMethod(Env, NativeNetworkManagerClass, "getInstance", "(Landroid/content/Context;)Lcom/psychehose/androidutil/network/NativeNetworkManager;", true);
	if (Env->ExceptionCheck())
	{
		Env->ExceptionDescribe();
		Env->ExceptionClear();
		UE_LOG(LogTemp, Error, TEXT("Exception occurred after finding method2"));
	}
 
	if (!NativeNetworkManagerGetInstanceMethod)
	{
		UE_LOG(LogTemp, Error, TEXT("Failed to find the getInstance method."));
	}

	// Static Method Call (인스턴스 필요 없음)
	// static 함수의 리턴값은 NativeNetworkManager 인스턴스 이건 jobject에 캐싱함
	NativeNetworkManagerObject = Env->CallStaticObjectMethod(NativeNetworkManagerClass, alternativeMethod, Context);
	if (!NativeNetworkManagerObject)
	{
		UE_LOG(LogTemp, Error, TEXT("Failed to get the NativeNetworkManager object."));
		return;
	}

	// 클래스에서 찾은 함수들을 캐싱함
	IsLocationPermissionGrantedMethod = FJavaWrapper::FindMethod(
		Env,
		NativeNetworkManagerClass,
		"isLocationPermissionGranted", "()Z",
		true
	);
	if (!IsLocationPermissionGrantedMethod)
	{
		UE_LOG(LogTemp, Error, TEXT("Failed to find the isLocationPermissionGranted method."));
		return;
	}
 
	GrantLocationPermissionMethod = FJavaWrapper::FindMethod(
		Env,
		NativeNetworkManagerClass,
		"grantLocationPermission", "()Z",
		true
	);
	if (!GrantLocationPermissionMethod)
	{
		UE_LOG(LogTemp, Error, TEXT("Failed to find the grantLocationPermission method."));
		return;
	}
 
	FetchCurrentNetworkInformationMethod = FJavaWrapper::FindMethod(
		Env,
		NativeNetworkManagerClass,
		"fetchCurrentNetworkInformation", "()V",
		true
	);
	if (!FetchCurrentNetworkInformationMethod)
	{
		UE_LOG(LogTemp, Error, TEXT("Failed to find the fetchCurrentNetworkInformation method."));
		return;
	}
 
	ConnectWifiMethod = FJavaWrapper::FindMethod(
		Env,
		NativeNetworkManagerClass,
		"connectWifi", "(Ljava/lang/String;Ljava/lang/String;Z)V",
		true
	);
	if (!ConnectWifiMethod)
	{
		UE_LOG(LogTemp, Error, TEXT("Failed to find the connectWifi method."));
		return;
	}
 
	HelloMessageMethod = FJavaWrapper::FindMethod(
		Env,
		NativeNetworkManagerClass,
		"helloMessage", "()V",
		true
	);
}

```



```cpp
void UWifiManager::HelloMessage()
{
	JNIEnv* Env = FAndroidApplication::GetJavaEnv();
	if (Env == nullptr)
	{
		UE_LOG(LogTemp, Log, TEXT("Env is nullptr"));
		return;
	}
	// 캐싱한 object와 method로 자바 함수 호출
	Env->CallVoidMethod(NativeNetworkManagerObject, HelloMessageMethod);
}
 
JNI_METHOD void Java_com_psychehose_androidutil_network_NativeNetworkManager_nativeCallbackMessage(JNIEnv* jenv, jobject thiz)
{
	// Callback은 여기를 통해서 받을 수 있다.
	UE_LOG(LogTemp, Log, TEXT("Native Callback Message"));
}
```


### JNI를 사용하기 위한 선행 지식

#### AGP 변경

* `gradle/libs.versions.toml` 파일 열어서 AGP 버전 변경

#### gradle 버전 변경

6. `gradle/wrapper/gradle-wrapper.properties` 파일을 연다.
7. `distributionUrl=https\://services.gradle.org/distributions/gradle-X.X-all.zip` X.X에 버전을 지정 한다.
8. `./gradlew --version`  버전 확인.


#### Activity란?
- **Activity**는 안드로이드 앱에서 하나의 화면을 나타내는 구성 요소
- 사용자가 앱을 실행하면 화면에 보이는 인터페이스를 담당하며, 화면 전환, 사용자 입력 처리 등을 관리

게임은 모든 화면을 다 지우니까 하나의 Activity만 유지함. 그게 바로 GameActivity

#### Context란?
- **Context**는 현재 앱의 상태나 환경 정보를 제공하는 객체
- 자원(리소스), 시스템 서비스, 파일 입출력, 테마, 디바이스 관련 정보 등에 접근할 때 필요
- Activity는 Context의 하위 개념이며, Activity 자신도 Context 역할
- UI를 생성하거나 변경할 때, UI 위젯이나 Dialog, Toast 등을 생성하는 데 Context가 필요

언리얼엔진에서는 Context를 매개변수로 받는 것이 있으면 GameActivity를 넣으면 됨.
