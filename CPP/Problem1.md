
아래 코드에서 "복사 생성"은 몇번 호출 되는가
```cpp

class A {
    int x;

public:
    A(int c) : x(c) {}
    A(const A& a) {
        x = a.x;
        std::cout << "복사 생성" << std::endl;
    }
};

class B {
    A a;

public:
    B(int c) : a(c) {}
    B(const B& b) : a(b.a) {}
    A get_A() {
        A temp(a);
        return temp;
    }
};

int main() {

    B b(10);

    std::cout << "---------" << std::endl;
    A a1 = b.get_A();
}
```




Copy Elision(복사 생략) 중 Return Optimization

왜냐하면 어떤 함수가 함수 내에서 생성한 객체를 리턴 한다면, 굳이 그걸 그냥 사용하면 되지 이를 복사 생성을 또할 필요가 없기 때문