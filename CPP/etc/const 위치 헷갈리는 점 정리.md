C++에서 포인터와 const를 같이 사용할 때 자주 헷갈린다. 그래서 복습을 계속 한다. 목표는 const 위치에 따라 포인터 자체와 가리키는 값 중 무엇이 변경 불가능한지 암기하는 것이다.


'const를 기준으로 왼쪽에 있는 것이 상수고 아무것도 없다면 오른쪽에 있는 것이 상수'



#### 포인터가 가르키는 값이 상수

* `const int* p` 
* `int const* p`

`const int* p`와 `int const* p`는 같다. 원리만 기억하면 같다는 사실을 도출할 수 있다. 
포인터가 가르키는 값이 상수임. 포인터 자체는 변경 가능하니까 다른 주소를 가르키면 됨

```cpp
#include <iostream>
int main() {
  int v = 10;
  int v2 = 30;
  const int *p = &v;
  int const *p2 = &v;

  std::cout << *p << std::endl;
  std::cout << *p2 << std::endl;
  
  *p = 20; // 컴파일 실패
  *p2 = 20; // 컴파일 실패

  p = &v2;
  p2 = &v2;
  std::cout << *p << std::endl;
  std::cout << *p2 << std::endl;


  return 0;
}
```

#### 포인터 자체가 상수

* int* const p

const의 왼쪽은 \*이므로 포인터 자체가 상수라는 뜻이다. 이 경우 p의 가르키는 값은 변경 가능하지만 p 자체를 변경할 수 없음.

```cpp

#include <iostream>
int main() {
  int value1 = 10;
  int value2 = 20;
  int *const p3 = &value1;

  *p3 = 30; // 가능: p가 가리키는 값은 변경 가능
  p3 = &value2; // 컴파일 에러: p 자체를 변경할 수 없음

  return 0;
}
```

#### 둘 다 상수

* const int* const p
```cpp
int main() {
  int value1 = 10;
  int value2 = 20
  const int* const p = &value1;
  *p = 30;  // 컴파일 에러: p가 가리키는 값을 변경할 수 없음
  p = &value2;  // 컴파일 에러: p 자체를 변경할 수 없음
}

```


### 클래스의 멤버 함수에서 const 사용

#### const 멤버 함수

const 멤버 함수는 클래스의 상태를 변경하지 않는다는 뜻이다.

즉 const 멤버 함수 내에서
* 멤버 변수 값 수정 불가
* non-const 멤버 함수 호출 불가
* this 포인터는 const MyClass* 타입이다.

```cpp
class MyClass {
public:
  int getValue() const {
    value = 10; // 컴파일 실패
    return value;
  }

private:
  int value;

}
```


#### const 참조 반환

const 참조 반환은 함수가 객체의 내부 데이터에 대한 읽기 전용 접근을 제공하는 방법이다.
아래 코드에서 const가 있는 경우에는 레퍼런스 타입으로 getText()를 하면 컴파일 에러가 발생해서 수정 할 수 있는 여지가 없다.

const가 없는 경우에는 getText()에서 컴파일 에러 발생 안하고 text를 수정 가능 하므로 Hello world를 출력하게 된다.

```cpp
class StringContainer {
public:
  StringContainer() : text("Hello") {}
  const std::string &getText() { return text; }

  // 수정 가능한 참조 반환
  //   std::string &getText() { return text; }

private:
  std::string text;
};

int main() {
  StringContainer container;
  std::cout << container.getText() << std::endl;

  std::string &text = container.getText(); // 컴파일 에러 발생
  text += " world";
  std::cout << container.getText() << std::endl;

  return 0;
}
```
