#### Immutable, Mutable

 러스트는 다른 언어와 다르게 기본적으로 Immutable(불변)입니다. 이것은 안전성과 동시성이라는 장점을 얻을 수 있도록 코드를 강제하는 요소 중 하나라고 합니다. 

```
fn main() {
    let x = 5;
    println!("The value of x is: {}", x);
    x = 6;
    println!("The value of x is: {}", x);
}
```

위처럼 작성하게 되면 immutable variable에 값을 재할당할 수 없다는 에러메시지를 출력합니다.

> error[E0384]: re-assignment of immutable variable `x`

만약 mutable 하게 사용하고 싶으면 ‘mut’ keyword를 사용하면 됩니다.

```
fn main() {
    let mut x = 5;
    println!("The value of x is: {}", x);
    x = 6;
    println!("The value of x is: {}", x);
}
```

 Rust에서 이런 정책을 채택한 이유는 컴파일러가 변경되지 않은 값에 대해서 보증을 하기 때문에, 버그 예방에 효율적입니다.

하지만 mut 키워드를 사용하는 것이 효율적인 경우가 있습니다. 예를 들면 대규모 데이터 구조체를 다루는 경우입니다. 기존의 인스턴스를 변경하는 것이 새로 인스턴스를 만들고 값을 사용하는 것보다 더 효율적이기 때문입니다. trade - off를 잘 고려해서 코드를 작성하면 될 것 같습니다.

#### Constant (상수)

상수는 mut을 사용하는 것이 허용되지 않고 불변 그 자체입니다. 상수 선언할 때는 타입을 무조건 지정해줘야 합니다.

```
const MAX_POINTS: u32 = 100_000;
```

#### Shadowing

 러스트는 shadowing을 허용합니다. 같은 이름의 변수를 선언하면 기존의 값은 가려지게 됩니다.

```
fn main() {
    let x = 5;

    let x = x + 1;

    let x = x * 2;

    println!("The value of x is: {}", x);
}
```

mut과 차이점은 새 변수를 선언하고, 값의 유형을 변경하면서도 같은 이름을 사용할 수 있다는 것이 장점입니다.

```
let spaces = "   ";
let spaces = spaces.len();

// mut 사용시에 이것은 타입을 변경했기 때문에
// 에러를 발생시킵니다.

let mut spaces = "   ";
spaces = spaces.len();
```

### Ref.

[https://rinthel.github.io/rust-lang-book-ko/ch03-01-variables-and-mutability.html](https://rinthel.github.io/rust-lang-book-ko/ch03-01-variables-and-mutability.html)