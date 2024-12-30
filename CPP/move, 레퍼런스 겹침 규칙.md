
#### lvalue, rvalue
lvalue는 "주소가 있는 값"으로 생각할 수 있음
마치 우리가 집 주소로 찾아갈 수 있는 집처럼, 프로그램에서 다시 찾아갈 수 있는 위치를 가진 값

rvalue는 "임시 값"으로 생각할 수 있음.
마치 계산기로 계산한 결과 값처럼, 잠시 존재했다가 사라지는 값

```cpp
int x = 10;        // x는 lvalue
int y = x;         // x는 lvalue
int z = x + y;     // (x + y)는 rvalue
```


#### c++의 move 의미

```cpp
class String {
    char* data;
public:
    // 이동 생성자
    String(String&& other) noexcept {
        data = other.data;        // 데이터 포인터만 가져옴
        other.data = nullptr;     // 원본은 무효화
    }
    
    // 이동 대입 연산자
    String& operator=(String&& other) noexcept {
        if (this != &other) {
            delete[] data;        // 기존 데이터 해제
            data = other.data;    // 새 데이터 이동
            other.data = nullptr; // 원본 무효화
        }
        return *this;
    }
};

// 사용 예시
String str1("Hello");
String str2 = std::move(str1);  // str1의 내용이 str2로 이동됨
```

여기서 `std::move`는 실제로 객체를 이동시키지 않고, **단지 lvalue를 rvalue로 캐스팅**하는 역할. 실제 이동은 이**동 생성자나 이동 대입 연산자**에서 일어남


#### Perfect Fowarding
Perfect forwarding은 함수 템플릿에서 인자의 값 카테고리(lvalue/rvalue)를 그대로 유지하면서 전달하는 기능

```cpp
template<typename T>
void wrapper(T&& param) {                // 보편 참조(universal reference)
    foo(std::forward<T>(param));         // perfect forwarding
}

// 사용 예시
std::string str = "hello";
wrapper(str);              // str은 lvalue로 전달됨
wrapper(std::string("world")); // 임시 객체는 rvalue로 전달됨
```

`std::forward`가 하는 일을 자세히 살펴보면:

1. lvalue가 전달되면 lvalue 참조로 전달
2. rvalue가 전달되면 rvalue 참조로 전달

이를 통해 얻는 이점:

1. 불필요한 복사를 방지할 수 있습니다
2. 인자의 원래 특성(lvalue/rvalue)을 그대로 보존할 수 있음
3. 템플릿 기반의 제네릭 코드를 효율적으로 작성할 수 있음


```cpp
class Widget {
    std::vector<int> data;
public:
    // 생성자에서 perfect forwarding 사용
    template<typename... Args>
    Widget(Args&&... args) 
        : data(std::forward<Args>(args)...) {}
};

// 사용 예시
std::vector<int> vec = {1, 2, 3};
Widget w1(vec);                     // lvalue로 전달 - 복사 발생
Widget w2(std::vector<int>{1,2,3}); // rvalue로 전달 - 이동 발생
```

move 의미론과 perfect forwarding은 현대 C++에서 성능 최적화의 핵심 요소이며, 특히 대용량 데이터를 다루는 프로그램에서 매우 중요한 역할.
이를 통해 불필요한 복사를 줄이고 더 효율적인 코드를 작성할 수 있음