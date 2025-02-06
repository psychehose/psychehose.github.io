
* jni와 jvm, jnienv, thread
* on the c++ side, how to call android code(written by kt)
* on the c++ side, how to bind c++ function to callback(from android code)




### gradle

#### aar build

```bash
$ ./gradlew :AndroidUtil:assembleRelease # root에서 
$ ./gradlew --version #gradle 버전 확인
```

#### gradle 버전 변경

1. `gradle/wrapper/gradle-wrapper.properties` 파일을 연다.
2. `distributionUrl=https\://services.gradle.org/distributions/gradle-X.X-all.zip` X.X에 버전을 지정 한다.
3. `./gradlew --version`  버전 확인.


C++ 라이브러리를 빌드할 떄, aar dep



C++ 함수 -> aar -> 코틀린 바인딩
C++ 함수 -> framework -> swift, objc 바인딩


공통 로직 C++


OCR


```c++
extern "C"  
JNIEXPORT jstring JNICALL  
Java_com_golfzon_android_util_network_NativeNetworkManager_nativeTest(JNIEnv *env, jobject thiz) {  
    std::string hello = "Hello from C++";  
  
    __android_log_print(ANDROID_LOG_INFO, TAG, "Checking location permission");  
    return env->NewStringUTF(hello.c_str());  
}
```

