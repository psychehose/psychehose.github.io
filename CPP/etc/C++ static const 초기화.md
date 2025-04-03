
## 헤더 파일에서의 static const 초기화

C++17에서는 다음과 같은 방법으로 헤더 파일에서 static const 변수를 초기화할 수 있다.

### 1. inline 키워드 사용 (C++17 이상)

```cpp
// header.h
#ifndef HEADER_H
#define HEADER_H

// C++17에서는 inline 키워드를 사용하여 헤더에서 정의 가능
inline static const int MAX_SIZE = 100;
inline static const double PI = 3.14159;

class MyClass {
public:
    // 클래스 내부의 static const
    static const int CLASS_CONSTANT = 200;

    // C++17에서는 inline을 사용하여 복잡한 타입도 가능
    inline static const std::string NAME = "MyClass";
};
#endif
```

### 2. constexpr 사용 (권장)

```cpp
// header.h
#ifndef HEADER_H
#define HEADER_H

// constexpr을 사용하면 컴파일 타임에 값이 결정됨
constexpr int MAX_SIZE = 100;
constexpr double PI = 3.14159;

class MyClass {
public:
    // 클래스 내부에서도 constexpr 사용 가능
    static constexpr int CLASS_CONSTANT = 200;
    // C++17에서는 문자열 리터럴도 가능
    static constexpr const char* NAME = "MyClass";
};
#endif
```
### 3. 복잡한 객체 초기화

```cpp
// header.h
#ifndef HEADER_H
#define HEADER_H

#include <vector>
#include <string>

// 복잡한 객체도 inline을 사용하면 헤더에서 초기화 가능
inline static const std::vector<int> DEFAULT_VALUES = {1, 2, 3, 4, 5};
inline static const std::string APP_NAME = "My Application";

#endif
```

## 클래스 내 static const 초기화 규칙

### 정수형과 부동소수점형의 차이

C++에서 클래스 내에서 static const 변수의 초기화는 타입에 따라 규칙이 다름

1. **정수형 상수** (int, char, bool, enum 등)
   - 클래스 선언 내에서 직접 초기화 가능
    
```cpp
class MyClass {
    static const int a = 3; // 가능
};
```

2. **부동 소수점형 상수** (float, double)
   - 클래스 선언 내에서 직접 초기화 불가능
     
```cpp
class MyClass {
    static const float a = 3.0f; // 컴파일 에러
};
```

### 부동소수점형 초기화 해결 방법

1. **C++17 이상에서는 `inline` 키워드 사용**:

```cpp
class MyClass {
    inline static const float a = 3.0f; // C++17 이상에서 가능
};
```

2. **별도의 소스 파일(.cpp)에서 정의**:

```cpp
// MyClass.h
class MyClass {
    static const float a; // 선언만
};
   
// MyClass.cpp
const float MyClass::a = 3.0f; // 정의와 초기화
```

3. **constexpr 사용** (권장):
```cpp
class MyClass {
    static constexpr float a = 3.0f; // 모든 타입에 가능
};
```

## 요약 및 권장사항

- static const는 정수형이 아닌 타입은 별도의 소스 파일에서 정의하는 것이 필요
- 단 C++17 이상에서는 `inline static const` 또는 `static constexpr`을 사용하면 헤더에서 선언 정의 동시에 가능
