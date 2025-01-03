
기존의 SFML에서는 멤버 변수 sf::Event::EventType을 확인해서 각 이벤트를 처리 했었다. 내가 사용하는 3.0 버전에서는 이벤트를 visitor를 이용해서 처리 한다. 


### Visitor 패턴이란?

Visitor 패턴은 객체 구조와 처리를 분리하는 디자인 패턴. 즉 '다른 클래스에 있는 알고리즘을 가져와서 실행할 수 있게 해주는 패턴'임 

```cpp
// 기존 방식 (이제 지원하지 않음)
if (event.type == sf::Event::Closed) {
    window.close();
}

// Visitor 패턴 사용 (새로운 방식)
event.visit([&window](const sf::Event::Closed&) {
    window.close();
});

```


### Visitor 패턴의 장점


1. 컴파일 타임 검사: 잘못된 이벤트 타임에 접근하는 것을 방지하고 타입 불일치 오류를 컴파일 단계에서 발견할 수 있다. 기존의 방식을 사용하면 아래처럼 런타임에서 에러를 잡을 수 있다.
   
```cpp
// 기존
if (event.type == sf::Event::MouseMoved) {
    // 실수로 KeyPressed의 데이터를 사용하려고 하면...
    bool isShiftPressed = event.key.shift;  // 잘못된 메모리 접근
}

// visitor
event.visit([](const sf::Event::MouseMoved& mouse) {
    // mouse.position만 사용할 수 있음
    // 다른 이벤트의 멤버는 접근 불가능
});

```
   
2. 코드 가독성: 중첩 if문, switch문 없음. 각 이벤트 타입별 처리가 원활
3. 유지보수 좋음: 기존 코드 수정하지 않고 새로운 핸들러 추가가 쉬움


### SFML은 어떠한 방식으로 Visitor 패턴을 사용했을까?

`sf:Event` 클래스를 확인하면 클래스 내에 struct으로 각 이벤트가 정의 되어 있다. 그리고 내부 변수에 private로 `m_data`를 가지고 있다.
이 `m_data`의 타입은 `std::variant<Closed, Resized, FocusLost, /*...*/ >` 이다.

`std::variant`는 여러 타입 중 하나를 저장할 수 있는 type-safe union이다.  그렇다면 어떻게 SFML에서 `m_data`와 `visit`을 통해서 이벤트를 처리할까?

이를 이해하기 위해 먼저 `std::variant`와 `std::visit`의 관계와 사용법에 대해 알아야만 한다.

#### std::variant와 std::visit

```cpp
// 기본구조 
// Variant: 여러 타입 중 하나를 저장할 수 있는 컨테이너
std::variant<A, B, C> data;
// Visitor: variant에 저장된 데이터를 처리하는 방법
auto visitor = [](const auto& value) { /* 처리 로직 */ };

// 동작 방식
// 1. variant가 데이터 저장
std::variant<int, std::string> data = 42;
// 2. visitor가 데이터 처리
std::visit([](const auto& value) {
    using T = std::decay_t<decltype(value)>;
    if constexpr (std::is_same_v<T, int>) {
        std::cout << "정수 처리: " << value << std::endl;
    }
    else if constexpr (std::is_same_v<T, std::string>) {
        std::cout << "문자열 처리: " << value << std::endl;
    }
}, data);
```

`using T = std::decay_t<decltype(value)>` 에서 `decltype(value)`는 value의 정확한 타입을 추론하고 `std::decay_t`는 참조와 const를 제거한 순수한 타입을 얻는다. 위 코드에서 `const int&`는 int로 변환된다.

이렇게 얻은 타입 T로 컴파일 타임에 타입 체크를 수행한다.

```cpp
if constexpr (std::is_same_v<T, int>) {
    // int 타입일 때의 처리
}
else if constexpr (std::is_same_v<T, std::string>) {
    // string 타입일 때의 처리
}
```

위의 예시와 같이`std::variant` 과 `std::visit`를 이용하면 컴파일 타임에서 안전하게 타입을 추론할 수 있고 각 타입마다 처리를 하기 용이 해진다는 장점을 알 수 있다.

#### SFML에서의 visit - Event::visit()

`Window::pollEvnet()` 를 호출하면 m_data가 변경된다.

```cpp
//SFML Event 내부 구현
template <typename TEventSubtype>

Event::Event(const TEventSubtype& eventSubtype)
{
	static_assert(isEventSubtype<TEventSubtype>, "TEventSubtype must be a subtype of sf::Event");
	if constexpr (isEventSubtype<TEventSubtype>)
		m_data = eventSubtype;
}
```


```cpp
// SFML Event 내부 구현
class Event {
public:
    // visitor 패턴을 이용한 이벤트 처리
    template <typename T>
    auto visit(T&& visitor) const {
    // visitor는 람다함수 or 함수 객체
        return std::visit(std::forward<T>(visitor), m_data);
    }
};
```

여기에서 Visitor 패턴의 핵심 동작이 일어난다. Event::visit 함수는 std::visit의 단순 래퍼다.
`std::forward<T>`를 통해 visitor **(람다나 함수 객체)** 를 perfect 전달한다. 따라서 m_data와 visitor를 통해서 각 이벤트에 대해서 처리를 할 수 있게 된다.

#### 이벤트 처리하는 방법 - 람다 함수

```cpp
void Simulator::handleEvents() {
if (auto event = window.pollEvent()) {
		event->visit([this](const auto& e) {
		    using T = std::decay_t<decltype(e)>;
		    if constexpr (std::is_same_v<T, sf::Event::Closed>) {
		        window.close();
		    } else if constexpr (std::is_same_v<T, sf::Event::KeyPressed>) {
		        if (e.code == sf::Keyboard::Key::X) {
		            window.close();
		        }
		    }
		});
	}
}
```

이 코드가 실행될 때 내부적으로 아래와 같은 과정이 일어난다.

1. 람다 함수가 `Event::visit`에 전달
2. `Event::visit`은 람다 함수를 `std::visit`으로 전달
3. `std::visit`은 m_data에 저장된 실제 타입을 확인하고 람다를 호출 한다.

이걸 좀 풀어서 설명하면 아래처럼 코드가 실행되는 것이다.

```cpp
std::visit(
    [this](const auto& e) {  // 여기서 e는 variant에 저장된 실제 타입의 참조
        using T = std::decay_t<decltype(e)>;
        if constexpr (std::is_same_v<T, sf::Event::Closed>) {
            window.close();
        } else if constexpr (std::is_same_v<T, sf::Event::KeyPressed>) {
            if (e.code == sf::Keyboard::Key::X) {
                window.close();
            }
        }
    },
    m_data  // variant 객체 전달
);
```

#### 이벤트 처리하는 방법 - 함수 객체

```cpp
struct EventVisitor {
    Simulator& simulator;  // 참조를 저장
    
    
    // 특정 이벤트용 처리기
	void operator()(const sf::Event::Closed& event) {
	    simulator.window.close();
	}
	void operator()(const sf::Event::KeyPressed& event) { ... }

	// 다른 모든 이벤트를 처리하는 템플릿
	template<typename T>
	void operator()(const T& event) {}
};
```

```cpp
void Simulator::handleEvents() {
	if (auto event = window.pollEvent()) {
		event->visit(EventVisitor{*this});
	}
}
```

 객체를 만들고 `operator()`를 구현하면 객체를 함수처럼 사용할 수 있다.
 
```cpp
EventVisitor visitor; 
visitor(something);
```

내부적으로는 위의 람다 함수의 경우와 사실 거의 같다. 람다 함수 대신에 함수 객체를 넘겨주는 것이 차이점이다. 그리고 기본적으로 std::visit은 variant에 저장된 실제 타입을 확인해서 그 타입에 맞는 operator를 호출한다

1. `event->visit(EventVisitor{*this});` 호출 한다.
2. Event::visit은 std::visit에 함수 객체와, m_data를 전달한다.
3. std::visit은 m_data(variant)를 실제 타입을 확인해서 그 타입에 맞는 operator를 호출한다.



