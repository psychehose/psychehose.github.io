
1. 성능 같음
   
2. 포인터는 원본 전달을 확실히 알려줌.

참조는 표현적으로 원본을 넘겨주는 지 아닌지 애매함

예시: PrintInfo(&info) - 포인터 vs PrintInfo(info) - 참조

참조에서 마음대로 고치는 부분은 const를 사용하면 해서 읽기용으로 만들 수 있음.

물론 포인터도 const 사용 가능

\*을 기준으로 앞뒤에 따라 의미가 달라짐. 변경
   

앞인 경우

- void (const PrintInfo StatInfo info) {} -> 주소값을 타고 가는 데이터 변경 불가능

```cpp
info->hp = 100; // 에러
```

뒤인 경우:

* void (PrintInfo StatInfo* const info) { } -> 주소값 자체 변경 불가능
```cpp
info = other_info; // 불가능
```

3. 참조 타입은  참조하는 대상이 없으면 안됨 nullptr 개념 없음,  포인터는 nullptr 가능