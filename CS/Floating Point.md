#### floating point의 메모리 구조

float는 4byte = 32bit다. 32개의 비트는 아래처럼 구성되어 있다.
* 1 비트: 부호비트
* 8 비트: 지수부
* 23 비트: 가수부 (fraction)

실제 값은 (-1)^(부호) * 1.가수부 * 2^(지수부 -127)로 계산할 수 있다.

실제 값을 구할 때는 지수부를 저장할 때 127 편향값을 더하는 과정이 있어서 -127를 빼는 것임

2.5 변환 과정을 보자.

1. 2.5를 이진법으로 나타내면 10.1
2. 정규화하면 1.01 * 2^1 : 여기에서 실제 지수는 1이지만 편향값 127을 더해서 128을 얻음

32bit 표현

* 1비트: 0 (양수)
* 8비트: 지수부 - 128의 이진수(10000000)
* 23비트: 가수부 - 01000000000000000000000 ( 01 + 빈자리 0 21개)
* 결과: 0 10000000 01000000000000000000000

#### int로 캐스팅할 때 발생할 수 있는 문제점

* 소수점 이하 손실
* float이 int의  범위를 초과할 수 있음 (clang 17 기준으로 int max로 초기화 됨)
```cpp
#include <iostream>
#include <limits>
int main() {

  float float_a = std::numeric_limits<float>::max();
  int int_a = 0;

  std::cout << float_a << std::endl; // 3.40282e+38
  std::cout << int_a << std::endl; // 0

  int_a = float_a; // 2147483647
  std::cout << int_a << std::endl;

  return 0;
}
```


#### 값을 비교 및 사칙연산을 할 때 부작용을 줄일 수 있는 방법

아래 코드는 애매하다. c의 값에 따라 true false가 변하니 프로그램을 예측 할 수가 없다. 이런 부작용을 줄이기 위해서는 엡실론을 사용하자.
```cpp
#include <iostream>
#include <limits>

int main() {
  float b = 0.2345;
  float c = 0.2345001;

  if (b == c) {
    std::cout << "b == c" << std::endl; // c가 0.23450001인 경우
  } else {
    std::cout << "b != c" << std::endl; // c가 0.2345001인 경우
  }
  
  return 0;
}
```

엡실론은 개발자가 정의 해도 되고 `limits`에 정의된 엡실론을 사용해도 된다. limits에 정의된 엡실론은 시스템에 맞는 적절한 엡실론 값이다. 일반적으로는 32비트 float의 경우 약 1.19209e-7 값을 가진다고 한다.

```cpp
#include <iostream>
#include <limits>

const float epsilon = 0.0000001;

int main() {
  float b = 0.2345;
  float c = 0.234501;

  if (std::fabs(b - c) < epsilon) {
    std::cout << "b == c" << std::endl;
  } else {
    std::cout << "b != c" << std::endl;
  }

  if (std::fabs(b - c) < std::numeric_limits<float>::epsilon()) {
    std::cout << "b == c" << std::endl;
  } else {
    std::cout << "b != c" << std::endl;
  }

  return 0;
}
```
