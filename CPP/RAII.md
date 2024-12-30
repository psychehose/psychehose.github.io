C++ 창시자인 비야네 스트로스트룹은 C++ 에서 자원을 관리하는 방법으로 다음과 같은 디자인 패턴을 제안하였습니다. 바로 흔히 RAII 라 불리는 자원의 획득은 초기화다 `- Resource Acquisition Is Initialization` 입니다. 이는 자원 관리를 스택에 할당한 객체를 통해 수행하는 것


포인터는 객체가 아님. delete를 하지 않고 포인터가 유실되면 힙에 저장되어 있는 데이터는 그대로 남아있음.

객체는 소멸될 떄 소멸자를 호출함.

포인터 '객체' 로 만들어서 자신이 소멸 될 때 자신이 가리키고 있는 데이터도 같이 `delete` 하게 하면 됩니다. 즉, 자원 (이 경우 메모리) 관리를 스택의 객체 (포인터 객체) 를 통해 수행하게 되는 것임


이러한 타입을 스마트 포인터라고 함.


#### unique_ptr
특정 객체에 유일한 소유권을 부여하는 포인터 객체를 `unique_ptr`

여기서 퀴즈! [std::move](https://modoocode.com/301) 가 왜 필요할까요?
```cpp
void do_something() {
  std::unique_ptr<A> pa(new A());
  std::cout << "pa : ";
  pa->some();

  // pb 에 소유권을 이전.
  std::unique_ptr<A> pb = std::move(pa);
  std::cout << "pb : ";
  pb->some();
}
```

1. unique_ptr는 리소스의 독점적 소유권을 보장하기 위해 복사를 허용하지 않습니다. 즉, 복사 생성자와 복사 대입 연산자가 delete되어 있죠.
2. 대신 소유권을 이전하는 방법으로 이동 연산자와 이동 생성자를 사용할 수 있습니다. 이들은 rvalue 참조를 매개변수로 받도록 설계되어 있습니다.
3. 그런데 `pa`와 같은 기존 unique_ptr은 lvalue입니다. 이동 연산자/생성자는 rvalue 참조만 받을 수 있으므로, lvalue인 `pa`를 직접 전달할 수는 없습니다.
4. 여기서 std::move가 필요한 것입니다! move는 lvalue를 rvalue로 캐스팅해주는 역할을 합니다. 이렇게 rvalue가 된 포인터는 이동 연산자/생성자가 받을 수 있는 형태가 되어 소유권 이전이 가능해집니다.

결국 이는 C++의 타입 시스템이 value category(lvalue/rvalue)를 통해 리소스의 소유권 이전을 안전하게 관리하는 방식이라고 볼 수 있습니다.


### unique_ptr 를 함수 인자로 전달하기

```cpp
void do_something(std::unique_ptr<A>& ptr) { ptr->do_sth(3); }

int main() {
  std::unique_ptr<A> pa(new A());
  do_something(pa);
}
```

```cpp
void do_something(A* ptr) { ptr->do_sth(3); }

int main() {
  std::unique_ptr<A> pa(new A());
  do_something(pa.get());
}
```

두 차이점은?

첫 번째 방식 (참조로 전달):
```cpp
void do_something(std::unique_ptr<A>& ptr)
```
이 방식은 unique_ptr 자체를 참조로 받습니다. 이는 다음을 의미합니다:

- 함수가 unique_ptr의 전체 기능에 접근할 수 있습니다. 예를 들어 reset()을 호출하거나, 다른 객체로 재할당할 수 있습니다.
- 소유권 관점에서 보면, 함수가 포인터의 소유권에 영향을 미칠 수 있는 권한을 가집니다.
- 함수가 unique_ptr만 받을 수 있고, 일반 포인터는 받을 수 없습니다.
  
두 번째 방식 (raw 포인터로 전달):

```cpp
void do_something(A* ptr)
```

이 방식은 get()을 통해 내부의 raw 포인터만 전달합니다:

- 함수는 단순히 객체의 멤버에만 접근할 수 있습니다.
- 소유권과 관련된 어떤 작업도 할 수 없습니다.
- 이 함수는 더 유연합니다. unique_ptr 뿐만 아니라 shared_ptr, raw 포인터 등 다양한 소스로부터 포인터를 받을 수 있습니다.

일반적으로 두 번째 방식이 더 선호됩니다. 그 이유는:

1. 함수의 의도가 더 명확합니다 - "나는 객체를 사용만 할 뿐, 소유권은 건드리지 않겠다"
2. 더 유연합니다 - 다양한 포인터 타입과 함께 사용할 수 있습니다
3. SOLID 원칙 중 인터페이스 분리 원칙(Interface Segregation Principle)에 더 부합합니다 - 함수는 실제로 필요한 기능만 받습니다

만약 함수가 정말로 포인터의 소유권을 조작해야 한다면, 그때는 첫 번째 방식을 사용하는 것이 적절할 것입니다.



### unique_ptr 을 쉽게 생성

c++14 `std::make_unique`


### unique_ptr 를 원소로 가지는 컨테이너

```cpp
#include <iostream>
#include <memory>
#include <vector>

class A {
  int *data;

 public:
  A(int i) {
    std::cout << "자원을 획득함!" << std::endl;
    data = new int[100];
    data[0] = i;
  }

  void some() { std::cout << "일반 포인터와 동일하게 사용가능!" << std::endl; }

  ~A() {
    std::cout << "자원을 해제함!" << std::endl;
    delete[] data;
  }
};

int main() {
  std::vector<std::unique_ptr<A>> vec;
  std::unique_ptr<A> pa(new A(1));

  vec.push_back(pa);  // 에러 발생

```


삭제된 `unique_ptr` 의 복사 생성자에 접근하였기 때문이지요. 기본적으로 `vector` 의 [push_back](https://modoocode.com/185) 함수는 전달된 인자를 복사해서 집어 넣기 때문에 위와 같은 문제가 발생하게 되는 것이지요.

이를 방지하기 위해서는 명시적으로 `pa` 를 `vector` 안으로 이동 시켜주어야만 합니다. 즉 [push_back](https://modoocode.com/185) 의 우측값 레퍼런스를 받는 버전이 오버로딩 될 수 있도록 말이지요. -> **move 이용**


```cpp
int main() {
  std::vector<std::unique_ptr<A>> vec;
  std::unique_ptr<A> pa(new A(1));

  vec.push_back(std::move(pa));  // 잘 실행됨
}
```

아니면 emplace_back 이용.

### emplace_back 과 이용할 시에 주의할 점

먼저 perfect forwarding의 핵심을 상기해보면, 이는 함수 템플릿이 인자를 받아서 다른 함수로 전달할 때 인자의 값 카테고리(lvalue/rvalue)와 const/volatile 등의 특성을 그대로 보존하는 것입니다.

emplace_back은 이 perfect forwarding을 활용해서 컨테이너 내부에서 객체를 직접 생성합니다. 기본적인 구현은 대략 이렇게 됩니다:

```cpp
template<typename... Args>
reference emplace_back(Args&&... args) {
    // 메모리 재할당이 필요한지 확인하고 처리
    
    // perfect forwarding을 사용해 객체를 직접 생성
    construct_at(data_ + size_, std::forward<Args>(args)...);
    
    ++size_;
    return back();
}
```

여기서 중요한 점은 `Args&&...`가 forwarding reference(universal reference)로 사용되어, 어떤 타입의 인자든 받을 수 있고, std::forward를 통해 그 특성을 완벽하게 보존

주의할점은 아래 코드를 조심해야함. 어떤 생성자가 호출되는 지.

```cpp
std::vector<std::vector<int>> v;
v.emplace_back(100000); // 100000개의 원소를 가진 벡터를 추가하게 됨.
```

이건 생성자 정의.
```cpp
vector(size_type count);                     // count개의 기본값으로 초기화된 원소
vector(size_type count, const T& value);     // count개의 value로 초기화된 원소
```
