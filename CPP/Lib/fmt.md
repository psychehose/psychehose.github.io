
fmt는 문자열 포매팅 라이브러리다. Python과 유사한 형식의 문법을 가지고 있어서 사용하기 간편하다. 이러한 점은 c++ 20 표준의 `std::format` 의 기반이 되었다. 20 이상에서는 표준을 사용하면 된다.


1. Python과 유사한 형식의 문법

```cpp
string name = "My Name";
int age = 2;
fmt::print("Name: {}, Age: {}", name, age);
```

2. 타입 안정성
	* 컴파일 시점에서 포맷 문자열과 인자의 타입 불일치를 검사 `fmt::format_string`


3. 다양한 포맷팅 옵션
```cpp
// 숫자 포매팅
fmt::print("Integer: {:d}, Float: {:.2f}", 42, 3.14159);

// 정렬과 패딩
fmt::print("Left aligned: {:<10}", "text");
fmt::print("Right aligned: {:>10}", "text");

// 사용자 정의 타입 지원
struct Point { int x, y; };
template <> struct fmt::formatter<Point> {
    // 사용자 정의 타입을 위한 포매팅 구현
};
```


### 라이브러리 링크

주로 CMake를 이용해서 프로젝트에 링크 한다.

```cmake
include(FetchContent)

FetchContent_Declare(
  Fmt
  GIT_REPOSITORY "https://github.com/fmtlib/fmt"
  GIT_TAG "7.1.3"
  )
  
FetchContent_MakeAvailable(Fmt)

target_link_libraries(program PUBLIC fmt)
```


### 사용 예제

* 기본 문자열 포매팅
```cpp
  std::string name = "psychehose";
  int age = 31;

  // basic formatting 순서.
  fmt::print("Name: {}, Age: {} \n", name, age);

  // by indexing
  fmt::print("{1}살을 먹은 {0}\n", name, age);

  // by name
  fmt::print("{name}은 {age}살\n", fmt::arg("name", name),
             fmt::arg("age", age));

  fmt::print("{name}은 {age}살\n", fmt::arg("age", age),
             fmt::arg("name", name));
         
```

이름 기반 포매팅을 할 때는 인자들의 이름을 지정 했기 때문에 순서가 바뀌어도 상관 없다.


* 숫자 포매팅

```cpp
  int num = 42;
  fmt::print("10진수 {}\n", num);             // 42
  fmt::print("16진수 {:x}\n", num);           // 2a
  fmt::print("16진수 (대문자) {:X}\n", num);  // 2A
  fmt::print("8진수 {:o}\n", num);            // 52
  fmt::print("2진수 {:b}\n", num);            // 101010

  // 부동소수점
  fmt::print("소수점 2자리: {:.2f}\n", 3.141592);
  fmt::print("지수 표기: {:e}\n", 1000000.0);
  fmt::print("자동 지수/고정: {:g}\n", 1000000.0);
```

* 정렬과 패딩, 최소 너비 설정

```cpp
// 왼쪽 정렬 (기본값)
fmt::print("왼쪽정렬: {:<10}\n", "left");   // "left      "

// 오른쪽 정렬
fmt::print("오른쪽정렬: {:>10}\n", "right"); // "     right"

// 가운데 정렬
fmt::print("가운데정렬: {:^10}\n", "center"); // "  center  "

// 사용자 지정 패딩 문자
fmt::print("패딩: {:*>10}\n", "pad");        // "******pad"

// 최소 너비 지정
fmt::print("{:10}\n", "text");       // "text      "

```

*  부호
```cpp
fmt::print("{:+}\n", 42);    // "+42"
fmt::print("{:+}\n", -42);   // "-42"
```

* 컨테이너 포맷팅

```cpp

#include "fmt/ranges.h"

std::vector<int> v = {1, 2, 3, 4, 5};
fmt::print("{}\n", v); // [1, 2, 3, 4, 5]

std::map<std::string, int> m = {{"apple", 1}, {"banana", 2}};
fmt::print("{}\n", m); // {"apple": 1, "banana": 2}
```

* 날짜 / 시간 포매팅

```cpp
#include <fmt/chrono.h>

std::time_t t = std::time(nullptr);
fmt::print("현재 시간: {:%Y-%m-%d %H:%M:%S}\n", fmt::gmtime(t));
```

* 문자열 변수로 저장 `fmt::format`

```cpp
// 문자열 반환
std::string result = fmt::format("Hello, {}!", "World");

// 여러 인자 사용
int x = 10, y = 20;
std::string coords = fmt::format("좌표: ({}, {})", x, y);
std::cout << coords << std::endl;

```


#### github
[fmt example github][https://github.com/psychehose/example_fmt]