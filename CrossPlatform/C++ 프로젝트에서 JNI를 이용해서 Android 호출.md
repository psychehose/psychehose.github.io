
C++ 기반의 프로젝트에서 안드로이드 함수를 사용 해야하는 경우가 있다. 안드로이드 게임을 타겟으로 하는 언리얼 엔진 프로젝트가 예시이다.

어떻게 하면 C++ 프로젝트에서 안드로이드 개발자가 정의한 함수를 호출할 수 있을까? JNI를 이용하면 된다. JNI는 Java Native Interface의 준말로 안드로이드 앱(자바/코틀린)과 네이티브 코드(C/C++) 사이를 연결해주는 인터페이스다. `JNIEnv*`라는 환경 포인터를 이용하여, 자바 클래스를 찾고 메서드를 호출하거나, 자바 타입과 C++ 타입 간 데이터를 변환할 수 있다.


C++를 이용해서 JNI를 이용해서 Android API를 호출 하는 방법은 크게 세 단계로 나눌 수 있다. 순서는 다음과 같다.

1. 안드로이드 스튜디오를 이용해서 Java / Kotlin으로 로직 작성하기
2.  gradle을  이용해서 안드로이드 라이브러리(AAR) 빌드
	* 이 과정에서 AGP 버전과, gradle 버전을 신경 써야함
	* gradle이 지원하는 필수 라이브러리들의 버전도 신경써야함
3.  C++ 프로젝트에서 JNI를 이용해서 Java 호출


```java

package com.example.myproject;

public class MyJavaClass {
    public static void sayHello(String msg) {
        // 간단히 로그 출력
        Log.d("MyJavaClass", "Hello from C++: " + msg);
    }
}
```

```cpp
#if PLATFORM_ANDROID
#include "Android/AndroidApplication.h"
#include "Android/AndroidJNI.h"
#endif

void MyJNIHelperFunction()
{
#if PLATFORM_ANDROID
    // 1) JNI 환경 포인터 가져오기
    JNIEnv* Env = FAndroidApplication::GetJavaEnv();
    if (!Env)
    {
        UE_LOG(LogTemp, Error, TEXT("Failed to get JavaEnv!"));
        return;
    }

    // 2) 자바 클래스 찾기
    //    FindClass에 넘길 때는 경로를 '/'로 구분해야 하며, 확장자는 쓰지 않습니다.
    //    com/example/myproject/MyJavaClass
    jclass JavaClass = Env->FindClass("com/example/myproject/MyJavaClass");
    if (!JavaClass)
    {
        UE_LOG(LogTemp, Error, TEXT("Failed to find MyJavaClass"));
        return;
    }

    // 3) sayHello(String) 메서드의 ID 가져오기
    //    함수 시그니처: (Ljava/lang/String;)V  →  인자는 String 하나, 반환 타입은 void
    jmethodID MethodID = Env->GetStaticMethodID(JavaClass, "sayHello", "(Ljava/lang/String;)V");
    if (!MethodID)
    {
        UE_LOG(LogTemp, Error, TEXT("Failed to get sayHello method ID"));
        return;
    }

    // 4) C++에서 전달할 문자열을 JNI jstring으로 변환
    const char* MessageCStr = "This message is from C++ via JNI!";
    jstring JMsg = Env->NewStringUTF(MessageCStr);

    // 5) 정적 메서드 호출(CallStaticVoidMethod)
    Env->CallStaticVoidMethod(JavaClass, MethodID, JMsg);

    // 6) 참조 해제(로컬 레퍼런스 등)
    Env->DeleteLocalRef(JMsg);
    Env->DeleteLocalRef(JavaClass);
#endif
}

```



**JNI Exception / Crash 처리** 

- JNI 사용 중 자바 쪽에서 예외가 발생해도 C++ 단에서 처리가 어려울 수 있습니다.
- 보통은 `Env->ExceptionCheck()` 나 UE의 헬퍼 함수를 통해 예외가 있었는지 확인해주고, 필요 시 예외를 Clear 한 뒤 적절히 처리해야 합니다.

```cpp
#if PLATFORM_ANDROID
#include "Android/AndroidApplication.h"
#include "Android/AndroidJNI.h"
#endif

void SomeJNIFunction()
{
#if PLATFORM_ANDROID
    JNIEnv* Env = FAndroidApplication::GetJavaEnv();
    if (!Env)
    {
        UE_LOG(LogTemp, Error, TEXT("JavaEnv를 가져오지 못했습니다."));
        return;
    }
    
    // 예를 들어, 자바 클래스 및 메서드 호출
    jclass JavaClass = Env->FindClass("com/example/myproject/MyJavaClass");
    if (!JavaClass)
    {
        UE_LOG(LogTemp, Error, TEXT("MyJavaClass 클래스를 찾을 수 없습니다."));
        return;
    }
    
    jmethodID MethodID = Env->GetStaticMethodID(JavaClass, "someStaticMethod", "()V");
    if (!MethodID)
    {
        UE_LOG(LogTemp, Error, TEXT("메서드 ID를 가져오지 못했습니다."));
        Env->DeleteLocalRef(JavaClass);
        return;
    }
    
    // 메서드 호출
    Env->CallStaticVoidMethod(JavaClass, MethodID);
    
    // JNI 예외 체크
    if (Env->ExceptionCheck())
    {
        // 예외 발생 시 로그 출력
        Env->ExceptionDescribe();
        // 예외 상태 클리어
        Env->ExceptionClear();
        UE_LOG(LogTemp, Error, TEXT("JNI 호출 중 예외가 발생했습니다."));
    }
    
    Env->DeleteLocalRef(JavaClass);
#endif
}
```

`AsyncTask` 등을 통해 UI 스레드를 거치는 방법을 고려 - 언리얼엔진



`com.example.myproject.MyJavaClass`라는 자바 클래스가 있고, 기본 생성자와 인스턴스 메서드 `doSomething(String message)`를 가진 경우의 예시

```cpp
#if PLATFORM_ANDROID
#include "Android/AndroidApplication.h"
#include "Android/AndroidJNI.h"
#endif

// 전역 변수 또는 클래스 멤버 변수로 캐싱
static jobject CachedMyJavaInstance = nullptr;

// 자바 인스턴스 생성 및 캐싱
void CreateJavaInstance()
{
#if PLATFORM_ANDROID
    JNIEnv* Env = FAndroidApplication::GetJavaEnv();
    if (!Env)
    {
        UE_LOG(LogTemp, Error, TEXT("JavaEnv를 가져오지 못했습니다."));
        return;
    }

    // 자바 클래스 경로: 슬래시('/')로 구분, 확장자 없이 작성
    jclass JavaClass = Env->FindClass("com/example/myproject/MyJavaClass");
    if (!JavaClass)
    {
        UE_LOG(LogTemp, Error, TEXT("MyJavaClass 클래스를 찾을 수 없습니다."));
        return;
    }

    // 기본 생성자: 인자가 없으므로 시그니처는 "()V"
    jmethodID ConstructorID = Env->GetMethodID(JavaClass, "<init>", "()V");
    if (!ConstructorID)
    {
        UE_LOG(LogTemp, Error, TEXT("MyJavaClass 생성자 ID를 가져오지 못했습니다."));
        Env->DeleteLocalRef(JavaClass);
        return;
    }

    // 객체 생성 (로컬 참조)
    jobject LocalInstance = Env->NewObject(JavaClass, ConstructorID);
    if (!LocalInstance)
    {
        UE_LOG(LogTemp, Error, TEXT("MyJavaClass 인스턴스 생성 실패"));
        Env->DeleteLocalRef(JavaClass);
        return;
    }

    // 전역 참조로 변환하여 캐싱 (JNI 로컬 참조는 호출 종료 후 소멸됨)
    CachedMyJavaInstance = Env->NewGlobalRef(LocalInstance);

    // 더 이상 필요 없는 로컬 참조 해제
    Env->DeleteLocalRef(LocalInstance);
    Env->DeleteLocalRef(JavaClass);
#endif
}

// 캐싱된 인스턴스로 인스턴스 메서드 호출
void CallInstanceMethodExample()
{
#if PLATFORM_ANDROID
    if (!CachedMyJavaInstance)
    {
        UE_LOG(LogTemp, Error, TEXT("캐싱된 자바 인스턴스가 없습니다."));
        return;
    }

    JNIEnv* Env = FAndroidApplication::GetJavaEnv();
    if (!Env)
    {
        UE_LOG(LogTemp, Error, TEXT("JavaEnv를 가져오지 못했습니다."));
        return;
    }

    // 캐싱된 객체로부터 클래스 정보를 가져옴
    jclass JavaClass = Env->GetObjectClass(CachedMyJavaInstance);
    if (!JavaClass)
    {
        UE_LOG(LogTemp, Error, TEXT("캐싱된 인스턴스로부터 클래스를 가져오지 못했습니다."));
        return;
    }

    // 예시: 인스턴스 메서드 "doSomething" 호출, 시그니처: (Ljava/lang/String;)V
    jmethodID MethodID = Env->GetMethodID(JavaClass, "doSomething", "(Ljava/lang/String;)V");
    if (!MethodID)
    {
        UE_LOG(LogTemp, Error, TEXT("doSomething 메서드 ID를 가져오지 못했습니다."));
        Env->DeleteLocalRef(JavaClass);
        return;
    }

    // 전달할 문자열 생성
    const char* Msg = "Unreal에서 호출한 메시지입니다!";
    jstring jMsg = Env->NewStringUTF(Msg);

    // 인스턴스 메서드 호출
    Env->CallVoidMethod(CachedMyJavaInstance, MethodID, jMsg);

    // 사용한 로컬 참조 해제
    Env->DeleteLocalRef(jMsg);
    Env->DeleteLocalRef(JavaClass);
#endif
}

// 사용이 끝난 후, 전역 참조 해제
void ReleaseJavaInstance()
{
#if PLATFORM_ANDROID
    JNIEnv* Env = FAndroidApplication::GetJavaEnv();
    if (Env && CachedMyJavaInstance)
    {
        Env->DeleteGlobalRef(CachedMyJavaInstance);
        CachedMyJavaInstance = nullptr;
    }
#endif
}

```


생성자에 변수가 있는 경우

```cpp
#if PLATFORM_ANDROID
#include "Android/AndroidApplication.h"
#include "Android/AndroidJNI.h"
#endif

// 전역 또는 클래스 멤버 변수로 자바 인스턴스를 캐싱하는 예시
static jobject CachedMyJavaInstance = nullptr;

void CreateJavaInstanceWithString()
{
#if PLATFORM_ANDROID
    JNIEnv* Env = FAndroidApplication::GetJavaEnv();
    if (!Env)
    {
        UE_LOG(LogTemp, Error, TEXT("JavaEnv를 가져오지 못했습니다."));
        return;
    }

    // 자바 클래스 찾기 (패키지 경로는 '/'로 구분)
    jclass JavaClass = Env->FindClass("com/example/myproject/MyJavaClass");
    if (!JavaClass)
    {
        UE_LOG(LogTemp, Error, TEXT("MyJavaClass 클래스를 찾을 수 없습니다."));
        return;
    }

    // 매개변수가 String인 생성자 시그니처: (Ljava/lang/String;)V
    jmethodID ConstructorID = Env->GetMethodID(JavaClass, "<init>", "(Ljava/lang/String;)V");
    if (!ConstructorID)
    {
        UE_LOG(LogTemp, Error, TEXT("매개변수가 있는 생성자 ID를 가져오지 못했습니다."));
        Env->DeleteLocalRef(JavaClass);
        return;
    }

    // 생성자에 전달할 문자열을 jstring으로 변환
    const char* myParam = "생성자 파라미터 값";
    jstring jParam = Env->NewStringUTF(myParam);

    // 생성자 호출로 객체 생성 (생성자 매개변수로 jParam 전달)
    jobject LocalInstance = Env->NewObject(JavaClass, ConstructorID, jParam);
    if (!LocalInstance)
    {
        UE_LOG(LogTemp, Error, TEXT("MyJavaClass 인스턴스 생성 실패"));
        Env->DeleteLocalRef(JavaClass);
        Env->DeleteLocalRef(jParam);
        return;
    }

    // 장기간 사용하기 위해 전역 참조로 캐싱
    CachedMyJavaInstance = Env->NewGlobalRef(LocalInstance);

    // 사용한 로컬 참조 해제
    Env->DeleteLocalRef(LocalInstance);
    Env->DeleteLocalRef(jParam);
    Env->DeleteLocalRef(JavaClass);
#endif
}

```


싱글톤인 경우?

```cpp
#if PLATFORM_ANDROID
#include "Android/AndroidApplication.h"
#include "Android/AndroidJNI.h"
#endif

// 전역 변수 또는 클래스 멤버 변수로 싱글톤 인스턴스를 캐싱
static jobject CachedMySingletonInstance = nullptr;

// 싱글톤 인스턴스를 얻어오는 함수
void GetSingletonInstance()
{
#if PLATFORM_ANDROID
    JNIEnv* Env = FAndroidApplication::GetJavaEnv();
    if (!Env)
    {
        UE_LOG(LogTemp, Error, TEXT("JavaEnv를 가져오지 못했습니다."));
        return;
    }

    // 자바 싱글톤 클래스 찾기
    jclass SingletonClass = Env->FindClass("com/example/myproject/MySingleton");
    if (!SingletonClass)
    {
        UE_LOG(LogTemp, Error, TEXT("MySingleton 클래스를 찾을 수 없습니다."));
        return;
    }

    // 정적 메서드 getInstance()의 ID 가져오기  
    // 시그니처는 "()Lcom/example/myproject/MySingleton;" 형태
    jmethodID GetInstanceMethod = Env->GetStaticMethodID(SingletonClass, "getInstance", "()Lcom/example/myproject/MySingleton;");
    if (!GetInstanceMethod)
    {
        UE_LOG(LogTemp, Error, TEXT("getInstance() 메서드 ID를 가져오지 못했습니다."));
        Env->DeleteLocalRef(SingletonClass);
        return;
    }

    // 정적 메서드 호출로 싱글톤 인스턴스 획득
    jobject LocalInstance = Env->CallStaticObjectMethod(SingletonClass, GetInstanceMethod);
    if (!LocalInstance)
    {
        UE_LOG(LogTemp, Error, TEXT("MySingleton 인스턴스 획득 실패"));
        Env->DeleteLocalRef(SingletonClass);
        return;
    }

    // 장기간 사용하기 위해 전역 참조로 저장
    CachedMySingletonInstance = Env->NewGlobalRef(LocalInstance);

    // 사용한 로컬 참조 해제
    Env->DeleteLocalRef(LocalInstance);
    Env->DeleteLocalRef(SingletonClass);
#endif
}

// 캐싱한 싱글톤 인스턴스의 인스턴스 메서드를 호출하는 예시
void CallSingletonInstanceMethod()
{
#if PLATFORM_ANDROID
    if (!CachedMySingletonInstance)
    {
        UE_LOG(LogTemp, Error, TEXT("캐싱된 싱글톤 인스턴스가 없습니다."));
        return;
    }

    JNIEnv* Env = FAndroidApplication::GetJavaEnv();
    if (!Env)
    {
        UE_LOG(LogTemp, Error, TEXT("JavaEnv를 가져오지 못했습니다."));
        return;
    }

    // 싱글톤 인스턴스의 클래스 정보 가져오기
    jclass SingletonClass = Env->GetObjectClass(CachedMySingletonInstance);
    if (!SingletonClass)
    {
        UE_LOG(LogTemp, Error, TEXT("싱글톤 인스턴스의 클래스를 가져오지 못했습니다."));
        return;
    }

    // 인스턴스 메서드 doSomething(String)의 ID 가져오기  
    // 시그니처: (Ljava/lang/String;)V
    jmethodID MethodID = Env->GetMethodID(SingletonClass, "doSomething", "(Ljava/lang/String;)V");
    if (!MethodID)
    {
        UE_LOG(LogTemp, Error, TEXT("doSomething 메서드 ID를 가져오지 못했습니다."));
        Env->DeleteLocalRef(SingletonClass);
        return;
    }

    // 전달할 문자열 생성
    const char* Msg = "싱글톤 인스턴스에서 호출한 메시지입니다!";
    jstring jMsg = Env->NewStringUTF(Msg);

    // 인스턴스 메서드 호출
    Env->CallVoidMethod(CachedMySingletonInstance, MethodID, jMsg);

    // 사용한 로컬 참조 해제
    Env->DeleteLocalRef(jMsg);
    Env->DeleteLocalRef(SingletonClass);
#endif
}

// 사용이 끝난 후, 전역 참조 해제 함수
void ReleaseSingletonInstance()
{
#if PLATFORM_ANDROID
    JNIEnv* Env = FAndroidApplication::GetJavaEnv();
    if (Env && CachedMySingletonInstance)
    {
        Env->DeleteGlobalRef(CachedMySingletonInstance);
        CachedMySingletonInstance = nullptr;
    }
#endif
}

```


### 안드로이드 이해

### Activity란?

- **Activity**는 안드로이드 앱에서 하나의 화면을 나타내는 구성 요소입니다.
- 사용자가 앱을 실행하면 화면에 보이는 인터페이스를 담당하며, 화면 전환, 사용자 입력 처리 등을 관리합니다.
- 예를 들어, 로그인 화면, 메인 화면, 설정 화면 등이 모두 Activity로 구현됩니다.

-> 게임은 모든 화면을 다 지우니까 하나의 Activity만 유지하는 거 같아. 그게 바로 GameActivity

### Context란?
- **Context**는 현재 앱의 상태나 환경 정보를 제공하는 객체입니다.
- 자원(리소스), 시스템 서비스, 파일 입출력, 테마, 디바이스 관련 정보 등에 접근할 때 필요합니다.
- Activity는 Context의 하위 개념이며, Activity 자신도 Context 역할을 합니다.
- UI를 생성하거나 변경할 때, UI 위젯이나 Dialog, Toast 등을 생성하는 데 Context가 필요합니다.



라이브러리의 메서드가 요구하는 Context 타입 파악 해야함. 근데? 나는 클래스를 만들 때 context가 있을걸?


####  2. C++ 사이드에서 `getInstance()` 호출하기

1. `JNIEnv*`를 얻는다. (`FAndroidApplication::GetJavaEnv()`)
2. `NativeNetworkManager` 클래스를 찾는다. (`FindClass`)
3. `getInstance(Context)` 메서드를 찾는다. (`GetStaticMethodID`)
4. 현재 `Context` 객체를 가져온다. (걍 엑티비티 넣어도 됨)
5. `CallStaticObjectMethod`로 싱글톤 인스턴스를 호출한다.
6. 반환된 객체를 `NewGlobalRef`로 저장한다.
7. 
```cpp
#if PLATFORM_ANDROID
#include "Android/AndroidApplication.h"
#include "Android/AndroidJNI.h"
#endif

// 전역 변수로 Java 인스턴스 저장
static jobject CachedNativeNetworkManagerInstance = nullptr;

void GetNativeNetworkManagerInstance()
{
#if PLATFORM_ANDROID
    JNIEnv* Env = FAndroidApplication::GetJavaEnv();
    if (!Env)
    {
        UE_LOG(LogTemp, Error, TEXT("JavaEnv를 가져오지 못했습니다."));
        return;
    }

    // 1. NativeNetworkManager 클래스 찾기
    jclass NetworkManagerClass = Env->FindClass("com/psychehose/androidutil/network/NativeNetworkManager");
    if (!NetworkManagerClass)
    {
        UE_LOG(LogTemp, Error, TEXT("NativeNetworkManager 클래스를 찾을 수 없습니다."));
        return;
    }

    // 2. getInstance(Context) 정적 메서드 찾기
    jmethodID GetInstanceMethod = Env->GetStaticMethodID(
        NetworkManagerClass,
        "getInstance",
        "(Landroid/content/Context;)Lcom/psychehose/androidutil/network/NativeNetworkManager;"
    );

    if (!GetInstanceMethod)
    {
        UE_LOG(LogTemp, Error, TEXT("getInstance() 메서드 ID를 찾을 수 없습니다."));
        Env->DeleteLocalRef(NetworkManagerClass);
        return;
    }

    // 3. 현재 Android Context 가져오기
    jobject AndroidContext = FAndroidApplication::GetGameActivity(); // Activity는 Context를 상속받음
    if (!AndroidContext)
    {
        UE_LOG(LogTemp, Error, TEXT("Android Context를 가져오지 못했습니다."));
        Env->DeleteLocalRef(NetworkManagerClass);
        return;
    }

    // 4. getInstance(Context) 호출 -> NativeNetworkManager 객체 반환
    jobject LocalInstance = Env->CallStaticObjectMethod(NetworkManagerClass, GetInstanceMethod, AndroidContext);
    if (!LocalInstance)
    {
        UE_LOG(LogTemp, Error, TEXT("NativeNetworkManager 인스턴스를 가져오는 데 실패했습니다."));
        Env->DeleteLocalRef(NetworkManagerClass);
        return;
    }

    // 5. 장기 사용을 위해 전역 참조로 변환하여 저장
    CachedNativeNetworkManagerInstance = Env->NewGlobalRef(LocalInstance);

    // 6. 로컬 참조 해제
    Env->DeleteLocalRef(LocalInstance);
    Env->DeleteLocalRef(NetworkManagerClass);

    UE_LOG(LogTemp, Log, TEXT("NativeNetworkManager 인스턴스를 성공적으로 가져왔습니다."));
#endif
}

```


