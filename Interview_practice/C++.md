
Q. VC++에서 Clang++으로 변경한 이유

표준을 준수해서 크로스플랫폼 호환성 확보하기 위해서 Clang++으로 변경했다.

* DWORD, BOOL과 같은 VC++ 타입들을 제거하고 표준으로 변경했다. 
* DWORD -> unsigned long, BOOL -> int
* 파일 IO ReadFile, WriteFile -> filesystem, fstream 사용
* mutex -> recursive_mutex
* VC++ 확장 제거

```
// MSVC 확장
__max(a, b)
__min(a, b)

// 표준 C++ 대안
std::max(a, b)
std::min(a, b)
```
 

Q. Legacy C / C++에서 Modern C++로 리팩토링 할 때 무엇을 했는지

1. new / delete -> shared_ptr

A클래스가 B, C, D, E 클래스를 프로퍼티로 가지고 있어서 의존함. B, C, D,E 들도 서로 각각 참조 관계를 이루고 있다. new - delete를 사용한다면 의존성이 있는 순서대로 해제해야한다. shared_ptr을 이용하면 순서와 관계 없이 메모리가 안전하게 관리되어서 이용하게 되었다.

2. 스마트포인터가 쓰여야 할 때는 언제인가

헤비 리소스라서 복사를 금지하고 싶을 때 unique_ptr을 사용하고, 런타임에 파생 클래스 인스턴스를 가리켜야 할 때 객체 슬라이싱 문제를 해결하기 위해 스마트 포인터를 사용한다. 또 쓰레드 프로그래밍에서 비동기로 큰 데이터를 넘겨줄 때와 같이 여러 객체가 동일 리소스를 생명주기 끝까지 참조해야 할 때 shared_ptr을 사용한다.

3. template 코드

파일 입출력에  LoadType, LoadValue 2개씩 n개의 타입들이 있다. 그러면 코드 확장이 불편하고 수정사항을 각각 반영해야한다. 그래서 템플릿을 사용했다. 템플릿을 이용하면 코드 중복제거, 유지보수성 향상, 타입 안전성, 일관된 인터페이스에서 이점을 가질 수 있다. 즉 코드 감소, 유지보수 부담이 감소한다. 그리고 명시적 인스턴스화를 사용해서 중복 링크 문제를 해결하고, 특수화를 이용해서 특별한 로직 구성한다. 만약 특수화가 아니면 일반적인 템플릿 코드 적용된다.



Q. 현대 C++에서 포인터를 사용 해야하는 경우

C / C++은 스택과 힙 영역 할당에 자유가 있다. 데이터가 큰 객체들을 모두 스택에 올리면 스택오버플로우 위험이 있다. 이때 포인터를 이용해서 동적을 할당을 해서 힙에 올리면 된다. 하지만 현대 C++ STL 도입 후에는 STL 컨테이너를 이용하는 것이 낫다. STL 컨테이너는 값 타입이지만 내부적으로는 메모리를 힙으로 관리하기 때문이다. 또한 함수를 통과 시킬 때 파라미터로 call by reference를 이용해야하는 경우에도 참조 타입으로 사용하면 되어서 포인터를 사용할 여지가 줄어든다. 다만 포인터를 사용해야하는 경우도 있다.

1. 소유권이 명확하고 리소스가 매우 큰 데이터인 경우

데이터가 복사·이동 자체가 비싸거나 금지(non-copyable)된 경우와 복사할 필요 없이 한 번만 넘겨주고 소멸 시 자동 해제하고 싶을 때 `std::unique_ptr<T>`를 사용한다.


2. 다형성 관리 측면 - 런타임에서 파생 클래스를 기반 클래스로 다뤄야할 경우

포인터를 사용하지 않고 값으로 처리하면 객체 슬라이싱 문제가 발생한다. `std::vector<Base>`처럼 값 컨테이너에 넣으면 슬라이싱 되므로 포인터로 보관해야한다.

```cpp
class Scene {     
	std::vector<std::unique_ptr<Shape>> shapes_;
 public:     
	 void addShape(std::unique_ptr<Shape> s) {
		 shapes_.push_back(std::move(s)); 
	} 
};
```

  
3. 공유 소유(Shared Ownership)

여러 객체가 동일 리소스를 생명주기 끝까지 참조해야 할 때 포인터를 사용한다. 다만 소유권을 고려해야 하므로 보통 shared_ptr을 사용한다. 주로 이벤트 디스패쳐, 캐시, 브로드캐스트 구조에서 사용된다. 동시성 프로그래밍에서 에서 콜백 함수로 데이터를 던져줄 때 데이터 크기가 크다면 shared_ptr로 주는 경우가 많다.

4. 비소유 관찰자

소유권이 없고 관찰(view)만 할 수 있을 때 관찰은 할 수 있지만 소멸 책임은 외부에 있는 경우에 원시 포인터나 참조자를 사용한다.


5. 책임 분리 패턴

책임 분리패턴이란 PImpl(Pointer to Implementation) 패턴은 C++에서 인터페이스와 구현 세부사항을 완전히 분리해, 컴파일 의존성과 바이너리 호환성(ABI 안정성)을 확보하기 위해 널리 쓰이는 기법이다. 이때 전방선언을 하고 이 타입을 포인터로 가지고 있다.

책임 분리 패턴을 사용하면 좋은 것이 헤더-소스 의존성 최소화를 할 수 있고, 클래스 내부 구현이 헤더에 드러내지 않아 구현 변경시 헤더가 바뀌지 않는다. 그래서 코드를 재컴파일할 필요가 줄어든다.

코드 예시

```cpp
// Widget.h
#pragma once
#include <memory>

class Widget {
public:
    Widget();
    ~Widget();

    void doSomething();
    // 복사·이동 생성자/대입 연산자도 필요 시 선언

private:
    struct Impl;                 // 전방 선언
    std::unique_ptr<Impl> pImpl; // 구현체 소유 포인터
};

// Widget.cpp
#include "Widget.h"
#include <iostream>

// Impl 정의
struct Widget::Impl {
    int data;
    void helper() { std::cout << "helper: " << data << "\n"; }
};

Widget::Widget() : pImpl(std::make_unique<Impl>()) {
    pImpl->data = 42;
}

Widget::~Widget() = default;  // unique_ptr가 Impl 파괴

void Widget::doSomething() {
    // 실제 로직은 Impl이 담당
    pImpl->helper();
}

// (필요하면) 복사·이동 연산자 구현
Widget::Widget(const Widget& other)
  : pImpl(std::make_unique<Impl>(*other.pImpl)) {}

Widget& Widget::operator=(const Widget& other) {
    if (this != &other) {
        *pImpl = *other.pImpl;
    }
    return *this;
}

Widget::Widget(Widget&&) noexcept = default;
Widget& Widget::operator=(Widget&&) noexcept = default;


```