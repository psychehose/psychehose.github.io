
#### 목표
- `<random>` 라이브러리를 활용한 난수 생성
- `<chrono>` 라이브러리를 활용한 시간 측정

### random

C언어의 `srand()`, `rand()`는 좋은 품질의 난수열을 생성 못함.

- 선형 합동 생성기(LCG) 기반으로 예측 가능한 패턴 발생
*  주기가 RAND_MAX(보통 32767)로 제한되어 짧음
*  통계적 특성이 좋지 않아 시뮬레이션/암호화에 부적합

C언어의 srand(), rand()는 사용하지 말자.
대신에 C++의 `random`을 사용하자


```cpp
#include <iostream>
#include <random>

int main() {
  // 시드값을 얻기 위한 random_device 생성.
  std::random_device rd;

  // random_device 를 통해 난수 생성 엔진을 초기화 한다.
  // 메르센 트위스터 알고리즘 난수 엔진
  std::mt19937 gen(rd());

  // 0 부터 99 까지 균등하게 나타나는 난수열을 생성하기 위해 균등 분포 정의.
  std::uniform_int_distribution<int> dis(0, 99);

  for (int i = 0; i < 5; i++) {
    std::cout << "난수 : " << dis(gen) << std::endl;
  }
}
```


이건 정규 분포

```cpp
#include <iomanip>
#include <iostream>
#include <map>
#include <random>

int main() {
  std::random_device rd;
  std::mt19937 gen(rd());
  std::normal_distribution<double> dist(/* 평균 = */ 0, /* 표준 편차 = */ 1);

  std::map<int, int> hist{};
  for (int n = 0; n < 10000; ++n) {
    ++hist[std::round(dist(gen))];
  }
  for (auto p : hist) {
    std::cout << std::setw(2) << p.first << ' '
              << std::string(p.second / 100, '*') << " " << p.second << '\n';
  }
}
```

```실행결과
-4  1
-3  38
-2 ****** 638
-1 ************************ 2407
 0 ************************************** 3821
 1 ************************ 2429
 2 ***** 595
 3  70
 4  1
```


### chrono

`<chrono>` 는 4가지 주요 구성요소를 가짐

- `std::chrono::duration`: 시간 간격 표현
- `std::chrono::time_point`: 특정 시점 표현
- `std::chrono::system_clock`: 시스템 시계
- `std::chrono::steady_clock`: 단조 증가 시계 (주로 시간 측정에 사용)


time_point들이 연산하면 duration이 됨.

```cpp
#include <chrono>
using namespace std::chrono;

// 현재 시간 얻기
auto now = system_clock::now();

// 시간 간격 정의
hours h(1);              // 1시간
minutes m(30);           // 30분
seconds s(15);           // 15초
milliseconds ms(100);    // 100밀리초

// 시간 계산
auto future = now + h + m;  // 1시간 30분 후

// 시간 간격 측정
auto start = steady_clock::now();
// ... 작업 수행
auto end = steady_clock::now();
auto duration = duration_cast<milliseconds>(end - start);

```




