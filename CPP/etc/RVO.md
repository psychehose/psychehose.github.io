
`Return Value Optimization`

C++에서 함수가 객체를 반환할 때 불필요한 복사를 피하기 위한 최적화 기법이다. 일반적으로 함수가 객체를 반환하면 임시 객체가 생성되고 복사하는데 **컴파일러가** 이를 최적화 하는 것이다.


RVO에는 두가지 종류가 있다.

1. NRVO (Named Return Value Optimization)

```cpp
MyClass func() {
	MyClass obj;  // 지역 객체
    // obj 초기화 작업
    return obj;   // NRVO 적용 가능
}
```


2. RVO (Return Value Optimization)

```cpp
MyClass createObject() {
    return MyClass();  // 임시 객체 직접 반환, RVO 적용
}
```


핵심은 RVO 최적화는 개발자가 아닌 컴파일러가 하는 것이다. 개발자가 해야하는 것은 컴파일러가 RVO를 적용할 수 있도록 구조적으로 코드를 작성 해야한다.

RVO가 효과적으로 작동하려면 아래 조건들이 충족 되어야 한다.
- 단순하게 반환 - 복잡한 로직 피하기
- 단일 경로 - 조건부 반환 최소화
- 직접 반환 - 불필요한 중간 변수 제거
- 참조 활용: 매개변수는 const 참조로
- move 남용 금지: RVO를 방해할 수 있음


#### NRVO vs RVO

성능 관점에서는 RVO가 더 확실함. 최적화는 컴파일러 재량이기 때문에 RVO가 더 확실함. RVO는 거의 모든 현대 컴파일러에서 적극 적용이 되었고, NRVO는 잘 되긴 하지만 누락 가능성도 존재함.

즉!
- RVO 가능하면 RVO (더 확실한 최적화)
- 복잡하면 NRVO (가독성과 유지보수성)