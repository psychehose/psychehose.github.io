### 의사 난수

왜 의사 난수라고 하는가? 어떻게 하면 진짜 난수를 추출할 수 있는가?

의사 난수라고 하는 이유는 컴퓨터 알고리즘으로 생성되는 난수는 수학적 공식에 따라 결정적으로 생성되기 때문이다. 이러한 난수 수열은 시드를 기반으로 해서 생성된다. 만약 같은 시드라면 동일한 수열이 생성된다.

의사 난수의 문제점은 시드와 알고리즘을 안다면 난수 시퀀스를 예측할 수 있고 알고리즘이기 때문에 값의 bias (편향성)이 있을 수 있다.

C++에서 난수를 생성하는 방법은 주로 `random의 std::random_device`를 사용하는 것이다. 이것은 수학적 알고리즘을 통해 생성되는 가짜 난수가 아니라 정말로 컴퓨터가 실행 되면서 마주치는 무작위적인 요소들 (예를 들어 장치 드라이버들의 noise) 을 기반으로한 진짜 난수를 제공한다.

다만 이 방식은 의사난수 생성보다 느리다. 그래서 보통 시드만 '진짜 난수'로 생성하고 의사 난수 생성기를 통해 난수를 생성한다. 보통  C++ 표준 라이브러리에 포함된 메르센 트위스터(Mersenne Twister) 의사난수 생성기 `std::mt19937`를 사용함. 

```cpp
#include <iostream>
#include <random>

int main(int argc, char *argv[]) {
  std::random_device rd;
  std::cout << "Entropy: " << rd.entropy() << std::endl;
  std::mt19937 gen(rd());

  std::uniform_int_distribution<> distrib(1, 6);
  for (int i = 0; i < 10; ++i) {
    std::cout << distrib(gen);
  }

  return 0;
}
```
