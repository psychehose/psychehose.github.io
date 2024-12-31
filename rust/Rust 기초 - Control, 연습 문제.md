### 제어문

 조건의 상태가 참인지에 따라 어떤 코드의 실행 여부를 결정하거나 조건이 만족되는 동안 반복 수행을 하는 것은 대부분의 프로그래밍 언어의 기초 문법입니다. 대다수의 언어처럼 if, for, while을 러스트에서도 사용합니다.

```
fn main() {
    let number = 6;

    if number % 4 == 0 {
        println!("number is divisible by 4");
    } else if number % 3 == 0 {
        println!("number is divisible by 3");
    } else if number % 2 == 0 {
        println!("number is divisible by 2");
    } else {
        println!("number is not divisible by 4, 3, or 2");
    }
}
```

if 뒤에 오는 코드의 조건은 반드시 boolean 타입이어야 합니다. 러스트에서 많은 else if가 많으면 ‘match’를 이용하길 권합니다. match 키워드는 나중에! 포스팅하겠습니다.

또 let 구문을 if, else를 이용해서 사용할 수 있습니다. 만약 다음과 같이 let 구문에서 if 블록을 사용하려면, if 블록과 else 블록은 같은 타입을 반환해야 합니다.

```
fn main() {
    let condition = true;
    let number = if condition {
        5
    } else {
        6
    };

    println!("The value of number is: {}", number);
}
```

아래의 코드는 에러를 발생시킵니다. 이유는 변수가 가질 수 있는 타입이 오직 하나이기 때문이에요. 러스트는 컴파일 시에 변수의 타입이 뭔지 확실하게 정의해야 합니다. 그래야 number가 사용되는 모든 곳에서 유효한지 검증할 수 있으니까요. 즉, number의 타입을 런타임에 정의할 수 없습니다. 이는 컴파일러가 보증할 수 있는 것을 많이 가져가고 복잡해지는 것을 방지하기 위함입니다.

```
fn main() {
    let condition = true;
    let number = if condition {
        5
    } else {
        "six"
    };

    println!("The value of number is: {}", number);
}
```

### 반복문과 반복

 러스트가 제공하는 반복문은 loop, while, for가 있습니다.

**loop**

 loop는 무한루프를 생성합니다. loop를 탈출하기 위해서는 if break와 같이 탈출 코드를 작성하면 됩니다.

```
fn main() {
    loop {
        println!("again!");
    }
}
```

**while**

 while은 대다수의 언어에 존재하는 while과 같습니다. 조건이 true인 동안 코드가 실행되고, 그렇지 않으면 루프에서 탈출합니다. while을 사용하게 되면 loop, if, else와 break를 사용할 때 발생하는 {  }를 줄일 수 있기 때문에, 더 깔끔해요. 

**for**

 역시 while과 마찬가지로 많이 익숙한 for입니다. for문은 while과 다르게, 조건 검사를 수행하는 코드가 없기 때문에 while보다 더 빠르고 안전합니다. for in 문법을 사용합니다.

```
fn main() {
    let a = [10, 20, 30, 40, 50];
	//	그냥 for element in a { } 도 가능
    for element in a.iter() {
        println!("the value is: {}", element);
    }
}
```

for 반복문이 안전하고 간결하기 때문에 러스트에서 가장 보편적으로 사용한다고 하네요. 정확히 몇 번 반복을 해야 하는 경우에도, 기본 라이브러리로 제공하는 Range를 사용합니다. Range는 한 숫자에서 다른 숫자 전까지 모든 숫자를 차례로 생성합니다.

```
fn main() {
    for number in (1..4).rev() {
        println!("{}!", number);
    }
    println!("LIFTOFF!!!");
}
```

연습 문제를 한번 풀어보도록 하겠습니다.

- 화씨와 섭씨를 상호 변환.
- n번째 피보나치수열 생성.
- 크리스마스 캐롤 “The Twelve Days of Christmas”의 가사를 반복문을 활용해 출력.

```
fn main() {
    println!("섭씨: {} -> 화씨: {}", 0, c_to_f(0.0));
    println!("화씨: {} -> 섭씨: {}", 32.0, f_to_c(32.0));

    let n = 7;
    println!("{}번째 피보나칠 수열의 값 = {}", n, fibo(n));

    let string = "The Twelve Days of Christmas";

    for c in string.chars() {
        print!("{}", c);
    }

    println!();

    for i in string.char_indices() {
        print!("{}", string.chars().nth(i.0).unwrap());

    }
    println!();

    for i in string.char_indices() {
        print!("{}", i.1);

    }



}


fn c_to_f(c: f64) -> f64 {
    (9.0/5.0) * c + 32.0
}

fn f_to_c(f: f64) -> f64 {
    (f - 32.0) * (5.0/9.0)
}

fn fibo(n: i32) -> i32 {
    if n <= 2 {
        1
    } else {
        fibo(n-1) + fibo(n-2)
    }
}


/*

1. 화씨와 섭씨를 상호 변환. -> // 배운것: f64로 타입 맞춰줄 것
    f to c
        -> (x°F − 32) × 5/9
        c = (f - 32) (5/9)
    c to f
        -> f = (9/5)c + 32

2. n번째 피보나치 수열 생성.

3. 크리스마스 캐롤 “The Twelve Days of Christmas”의 가사를 반복문을 활용해 출력. // 배운것: nth(x), .chars()

string 인스턴스 함수
.chars() -> 요소 값
.char_indices() -> index랑 요소
.nth() -> n번째 요소 출력(에러 가능성이 있으므로, unwrap()을 사용하거나, expect를 이용해서 에러 핸들 해야함)

*/
```

#### Github

[https://github.com/psychehose/rust_basic_problem](https://github.com/psychehose/rust_basic_problem)