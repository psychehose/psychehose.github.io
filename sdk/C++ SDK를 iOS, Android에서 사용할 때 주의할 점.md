C++ SDK를 iOS/Android에서 사용할 때, 메모리 관리에서 주의해야 할 점들이 있다. 특히 C++에서 할당한 메모리를 JNI나 Objective-C에서 해제할 때 문제가 될 수 있다.

힙 관리자의 차이 C++ runtime, JVM, iOS ARC가 각각 다른 메모리 관리자를 사용하기 때문에 한쪽에서 할당한 메모리를 다른 쪽에서 해제하면 크래시가 발생한다.

또 JNI에서는 Global/Local Reference 문제를 주의 해야한다. C++에서 반환한 객체 포인터를 Java에서 오래 보관하려면 NewGlobalRef로 변환해야 하고, 사용 후 반드시 DeleteGlobalRef로 해제해야 한다.

그리고 iOS에서는 ARC와의 충돌할 수 있다. C++에서 할당한 메모리를 Objective-C 객체로 래핑할 때 소유권이 불분명해질 수 있다. 이런 경우 메모리를 할당할 때 unique_ptr로 선언해 메모리 소유권을 보장하고 콜백으로 값을 넘겨줄 때는 작은 객체라면 복사, 아닌 경우라면 shared_ptr을 사용한다.

Android JNI 개발에서 멀티스레딩을 다룰 때 주의할 점이 있다. 특히 C++ 스레드에서 Java 메서드를 호출할 때 어떤 문제가 발생할 수 있고, 해결 방법을 무엇일까?

문제의 핵심 포인트는 JNIEnv 포인터가 스레드별로 고유하다는 점을 명심 해야한다. 한 스레드에서 받은 JNIEnv를 다른 스레드에서 사용하면 크래시가 발생한다. C++ 스레드에서 Java 호출 시 문제 C++에서 생성한 스레드는 기본적으로 JVM에 연결되어 있지 않다따라서 Java 메서드를 호출하려고 하면 JNIEnv를 얻을 수 없다'는 에러가 발생한다.

따라서 Global Reference를 사용해서 Java 객체 참조를 안전하게 공유를 해야한다. 하지만 멀티스레딩에서는 Global Reference만으로는 부족하고, 각 C++ 스레드에서 JNI 함수를 호출하려면 먼저 AttachCurrentThread로 JVM에 연결해서 JNIEnv를 얻어야 합니다. 그래서 보통 멀티 스레딩 환경이면 Global Reference + AttachCurrentThread 조합으로 사용한다고 한다.