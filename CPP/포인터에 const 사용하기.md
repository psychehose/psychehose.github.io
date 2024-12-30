
```cpp
int age = 29;
const int* pt = &age;

*pt += 1; // 불가능

age += 1; // age는 const가 아니므로 가능

cout << age << endl;

```



```cpp
// 가능
const float g_earth = 9.80;
const float* pe = &g_earth;


// 불가능 - pm을 이용해 g_moon의 값을 변경한다면 const라는 것이 무의미해지기 때문에
// c++은 const 변수의 주소를 const가 아닌 일반 포인터에 대입하는 것을 금지함.
const float g_moon = 1.63;
float* pm = &g_moon;

```


포인터를 지시하는 포인터를 사용할 때 복잡해짐

const가 아닌 포인터를 const 포인터에 대입하는 것은 간접 지시인 경우에만 가능(?)

```cpp
int age = 29;
int* pd = &age;
// 간접지시? 왜 간접지시냐면 pt는 결국 age를 가리키는 거랑 마찬가지이기 뗴문ㅇㅇ
const int* pt = pd; //( 이거 가능?)

// age++ 가능
// *pd +=1 가능
// *pt += 1는 불가능!
```

위의 예에서 보다시피 간접지시에서 const와 const가 아닌 것을 섞어쓰는 것은 매우 non - safe임.

```cpp
const int **pt2;
int* p1;
const int n = 13;

pp2 = &p1; // 사실 안됨, 근데 된다고 가정
*pp2 = &n; // const끼리니까 간접지시로 p1이 n을 가리키게 한다.

*p1 = 10; // const n을 변경하게 만든다.? const 무효

```

const가 아닌 포인터를 const 포인터에 대입하는 것은 한다리만 건너는 간접지시인 경우에만 가능. 

노트는 무슨 말이야 젠장..

Note. 데이터형 자체가 포인터가 아니라면 const 데이터의 주소이든, const가 아닌 데이터의 주소이든 const를 지시하는 포인터(한다리 건너는 간접지시?) 에 모두 대입할 수 있다.


```cpp

int n = 1;

const int* p1 = &n;
int* p2 = &n; 
const int* p3 = p2;

// 이게 안됨.
int* n2 = 2;
const int* p4 = &n2;
int* p5 = p4; // 여기서 안됨

// 이것도 안됨
const int x = 5;
int* ptr = &x;

```