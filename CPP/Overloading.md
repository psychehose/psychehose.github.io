
```cpp
#include <iostream>

class Complex {

private:

 double real, img;
 double get_number(const char* str, int from, int to) const;
 
public:
  Complex(double real, double** img) : real(real), img(img) { }

  Complex(const Complex& c) {
    real = c.real;
    img = c.img ;
  }

  Complex(const char* str);

  Complex operator+(const Complex& c) const;

  Complex operator-(const Complex& c) const;

  Complex operator*(const Complex& c) const;

  Complex operator/(const Complex& c) const;

  Complex& operator=(const Complex& c);

  Complex& operator+=(const Complex& c);

  Complex& operator-=(const Complex& c);

  Complex& operator=(const Complex& c);

  Complex& operator/=(const Complex& c);

  void println() { std::cout << "( " << real << " , " << img << " ) " << std::endl; }

};

Complex Complex::operator+(const Complex& c) const {
  Complex temp(real + c.real, img + c.img);
  return temp;
}

Complex Complex::operator-(const Complex& c) const {
  Complex temp(real - c.real, img - c.img);
  return temp;
}

Complex Complex::operator*(const Complex& c) const {
  Complex temp(real * c.real - img * c.img, real * c.img + img * c.real);
  return temp;
}

Complex Complex::operator/(const Complex& c) const {

  Complex temp(
    (real * c.real + img * c.img) / (c.real * c.real + c.img * c.img),
    (img * c.real - real * c.img) / (c.real * c.real + c.img * c.img));
  return temp;
}

  

Complex& Complex::operator=(const Complex& c) {
  real = c.real;
  img = c.img;

  return *this;

}

  

Complex& Complex::operator+=(const Complex& c) {

  (*this) = (*this) + c;

  return *this;

}

Complex& Complex::operator-=(const Complex& c) {

  (*this) = (*this) - c;

  return *this;

}

Complex& Complex::operator*=(const Complex& c) {

  (*this) = (*this) * c;

  return *this;

}

Complex& Complex::operator/=(const Complex& c) {

  (*this) = (*this) / c;

  return *this;

}

Complex::Complex(const char* str) {
  int begin = 0, end = strlen(str);
  img = 0.0;
  real = 0.0;

  // 먼저 가장 기준이 되는 'i' 의 위치를 찾는다.
  int pos_i = -1;

  for (int i = 0; i != end; i++) {
    if (str[i] == 'i') {
      pos_i = i;
      break;
    }
  }
  // 만일 'i' 가 없다면 이 수는 실수 뿐이다.
  if (pos_i == -1) {
    real = get_number(str, begin, end - 1);
    return;
  }
  // 만일 'i' 가 있다면,  실수부와 허수부를 나누어서 처리하면 된다.
  real = get_number(str, begin, pos_i - 1);
  img = get_number(str, pos_i + 1, end - 1);

  if (pos_i >= 1 && str[pos_i - 1] == '-') img *= -1.0;
}

double Complex::get_number(const char *str, int from, int to) const {
  bool minus = false;
  if (from > to) return 0;
  if (str[from] == '-') minus = true;
  if (str[from] == '-' || str[from] == '+') from++;

  double** num = 0.0;
  double** decimal = 1.0;

  bool integer_part = true;

  for (int i = from; i <= to; i++) {
    if (isdigit(str[i]) && integer_part) {
      num *= 10.0;
      num += (str[i] - '0');
      
    } else if (str[i] == '.')
      integer_part = false;

    else if (isdigit(str[i]) && !integer_part) {
      decimal /= 10.0;
      num += ((str[i] - '0') * decimal);
    } else // 그 이외의 이상한 문자들이 올 경우
      break;  
  }
  if (minus) num *= -1.0;

  return num;
}
int main() {

  Complex a(0, 0);

  a = a + "-1.1 + i3.923";
  a.println();

  a = a - "1.2 -i1.823";
  a.println();
  
  a = a * "2.3+i22";
  a.println();
  
  a = a / "-12+i55";
  a.println();

}
```


#### Q1. 연산자 오버로딩에서 왜 리턴 값이 Complex&가 아닌 Complex일까?
 Complex a = b + c + b;를 고려 했을 때 (b.plus(c)).plus(b) 가 되므로 -> (b + c) + (b + c)가 되어버림
 그래서 사칙연산에는 주소값을 리턴하지 않는다.


#### Q2. Some_Class a = b; 와 Some_Class a, a = b;의 차이점은?

전자는 복사생성자, 후자는 생성 후 대입연산자 여기에서는 얕은 복사가 일어난다.



#### Q3. 아래 함수가 없어도 a = a + "-1.1 + i3.923";  는 컴파일 된다. 이유는?

```cpp
Complex operator+(const char* str) const;
Complex operator-(const char* str) const;
Complex operator*(const char* str) const;
Complex operator/(const char* str) const;

Complex Complex::operator+(const char* str) const {

  Complex temp(str);
  return (*this) + temp;
}

Complex Complex::operator-(const char* str) const {
  Complex temp(str);
  return (*this) - temp;
}

Complex Complex::operator*(const char* str) const {
  Complex temp(str);
  return (*this) * temp;
}

Complex Complex::operator/(const char* str) const {
  Complex temp(str);
  return (*this) - temp;
}
```

컴파일러는 문자열 리터럴로부터 `const Complex` 타입의 객체를 새롭게 생성할 수 있다.

a = a + "-1.1 + i3.923"; 를 a = a.operator+("-1.1 + i3.923")로
그리고 이것을  a = a.operator+(Complex("-1.1 + i3.923")); 변환할 수 있다.

그러나 a = "-1.1 + i3.923" + a;는 컴파일 실패한다. 변환할 수 없는 형태이기 때문이다.
