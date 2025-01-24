

## 핵심 개념

1. rvalue란?
	- 메모리상의 임시 값입니다
	- 프로그램에서 곧 사라질 값입니다
	- 주소를 가질 수 없는 값입니다


```cpp
int a = 5 + 3;        // 여기서 5 + 3이 rvalue
std::string str = std::string("hello");  // std::string("hello")가 rvalue
```

2. rvalue 참조(&&)란?
	* rvalue를 참조할 수 있는 특별한 참조 타입
	* C++11


```cpp
// 1. 기본적인 rvalue와 참조
int&& rref = 42;           // 42는 rvalue, rref는 rvalue 참조
int x = 10;
int&& rref2 = x;          // 컴파일 에러! x는 lvalue이므로 rvalue 참조 불가능
int&& rref3 = std::move(x); // OK! std::move는 lvalue를 rvalue로 변환

// 2. 문자열 예제
std::string getName() {
    return "John";         // "John"은 임시 객체(rvalue)
}

std::string&& name = getName();  // 임시 객체를 rvalue 참조로 받음
```


# 왜 필요?

성능 최적화, 불필요한 복사 방지
```cpp
class BigData {
    int* data;
    size_t size;
public:
    // 이동 생성자 (rvalue 참조 사용)
    BigData(BigData&& other) noexcept {
        // 포인터만 복사 (매우 빠름)
        data = other.data;
        size = other.size;
        
        // 원본 무효화
        other.data = nullptr;
        other.size = 0;
    }
};
```



# 자주 하는 실수
* rvalue 참조 변수는 lvalue
```cpp
void process(int&& x) {
    int&& y = x;      // 에러! x는 rvalue 참조지만, 그 자체는 lvalue입니다
    int&& y = std::move(x);  // OK
}
```

* std::move는 실제로 객체 이동X

```cpp
std::string str = "hello";
std::move(str);           // 아무 일도 일어나지 않음
std::string str2 = std::move(str);  // 여기서 실제 이동 발생
```


# 정리
- rvalue: 임시적이고 곧 사라질 값
- rvalue 참조(&&): 이러한 임시 값을 참조할 수 있는 방법
- 주요 용도: 이동 생성자와 이동 대입 연산자 구현
- 성능 최적화: 불필요한 복사를 줄이고 효율적인 리소스 이동 가능

