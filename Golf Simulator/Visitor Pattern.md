
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


using T부터 다시 공부하고 작성하기
std::visit()을 사용할 때 주의점은 모든 타입 처리가 필요하다는 것이다.


```cpp
class Event {
private:
    // 여러 이벤트 타입을 저장할 수 있는 variant
    std::variant<Closed, KeyPressed, MouseMoved, ...> m_data;

public:
    // visitor 패턴을 이용한 이벤트 처리
    template <typename Visitor>
    auto visit(Visitor&& visitor) const {
        return std::visit(std::forward<Visitor>(visitor), m_data);
    }
};
```


```cpp
namespace sf
{
class Event
{
public:
	struct Closed
	{
	};
	struct KeyPressed
	{
		Keyboard::Key code{}; //!< Code of the key that has been pressed
		Keyboard::Scancode scancode{}; //!< Physical code of the key that has been pressed
		bool alt{}; //!< Is the Alt key pressed?
		bool control{}; //!< Is the Control key pressed?
		bool shift{}; //!< Is the Shift key pressed?
		bool system{}; //!< Is the System key pressed?
	};

private: 
	std::variant<Closed, Resized, FocusLost, /*...*/ > m_data;

// ... 
};
}

```


SFML에서 m_data는 Window::pollEvnet() 호출 시 변경 된다.

```cpp
sf::RenderWindow window;
sf::Event event;

while (window.pollEvent(event)) {
     // 여기서 m_data가 변경됨!
}
```

그런 다음에 visit 함수를 사용해서