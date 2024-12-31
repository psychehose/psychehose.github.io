
WWDC 2023 Generalize APIs with parameter pack을 보고 알게된 것을 정리하기 위해 포스팅 하게 되었습니다. 모바일 애플리케이션을 만들 때 Swift로 Variadic Function을 사용하는 경우가 많이는 있을 것 같지 않지만, Swift 언어 자체 관점에서 보면 참 매력적인 피쳐라고 생각해요. 얼른 실험이 끝나고 swift 메인에 병합되었으면 좋겠네요.

Generalize APIs with parameter pack을 보기 위해서는 제네릭에 대한 이해를 어느 정도 요구하고 이 포스팅도 마찬가지로 영상을 따라가기 때문에 제네릭에 대한 선수적인 지식이 조금 필요할 것 같아요. 하지만 쉽게 풀어쓰도록 최선을 다하겠습니다.




---

#### Variadic

파라미터 팩에 대한 영상을 볼 때 계속 나오는 단어가 있습니다. **Variadic argument** 한글로 뜻 풀이하면 가변 전달인자입니다. 우리가 흔히 함수를 작성할 때 파라미터 영역과 리턴타입을 선언부에 작성하고 body에 함수 내용을 작성합니다.

```
// 선언부
func sum(_ operand1: Int, _ operand2: Int) -> Int {
	// Body
	return operand1 + operand2
}

// main.swift

sum(3, 4)
```

위 코드에서 3, 4가 바로 argument(전달인자)입니다. 지금처럼 함수 파라미터 개수가 2개인 경우도 있지만, 상황이 바뀌어 세 수를 더하고 싶습니다. 그러면 어떻게 하면 될까요? 배열을 만들어 루프를 돌아 처리 하는 방법이 있겠지만 저장할 필요가 없기 때문에 배열을 사용하고 싶지 않습니다. 그렇다면 함수 오버로딩을 통해서 구현하면 됩니다.

```
// 함수 오버로딩

func sum(_ operand1: Int, _ operand2: Int, _ operand3: Int) -> Int {
	// Body
	return operand1 + operand2 + operand3
}

// main.swift

sum(3, 4, 5)
```

또 상황이 바뀌어 6개의 수를 더하고 싶습니다… 함수 오버로딩을 이용하게 되면 반복적인 패턴이 나타나게 되고 로직이 바뀌면 유지보수가 쉽지 않습니다. 그럴 때 가변 인수개수를 허용하는 것입니다. 이것이 바로 Variadic function 입니다.

```
func sum(_ operands: Int...) -> Int {
  var sum = 0
  for op in operands {
    sum += op
  }

  return sum
}


print(sum(1,2,3,4,5,6,7,8)) // 36
```

이런 Variadic function은 확실히 유용합니다. 하지만 한계점이 명확합니다.

1. 여러개의 인자를 받고 이걸 tuple로 만들어서 return 하는 함수를 만든다고 가정합시다. 그렇다면 어떤 타입을 리턴 해야할까요?

```
func tuplify(_ numbers: Int...) -> ??? {
	
}
```

2. Variadic parameter는 다양한 타입을 사용할 수 없습니다. 여러 타입을 사용하려면 type erasure를 고려해야합니다. 하지만 이 방법을 사용하게 된다면 전달인자의 specific한 타입에 대한 정보를 사용할 수 없습니다. 이 말은 캐스팅을 해서 사용해야 하기 때문에 런타임에러가 발생할 확률이 높습니다. (디버깅 비용을 낮추기 위해서는 컴파일러를 최대한 이용해야 합니다.) 

```
func testFunc(objects: Any...) {
	// 컴파일러는 일을 안한다.
	// object들은 type information을 이용할 수 없음 -> 캐스팅 해야함 -> 런타임에러
}
```

 즉 현재까지의 Swift로는 제네릭과 Variadic parameter로 type information을 보존할 수가 없고 여러개의 전달인자를 사용할 수 없었습니다. 이것을 하는 방법은 overloading 밖에 없었습니다. 하지만 Swift 5.9에서 Parameter Pack이 도입되었고 제네릭과 함께 전달인자 길이에 대해 추상화가 가능해졌습니다.

#### Parameter Pack

파라미터 팩은 영단어 그대로 해석하면 됩니다. 파라미터를 담는 팩입니다. 타입 또는 값을 가지고 있고 (holds) 그것을 싸서 (packs) 전달인자로 넘겨줄 수 있습니다.

> Bool, Int, String 이 세가지 individual type을 한꺼번에 묶어서 처리함 → type pack  
> true, 10, “” 위의 type pack에 대응하는 value pack

파라미터 팩은 생각보다 친숙한 개념입니다. 데이터를 담는 바구니 같은 것이기 때문에요. 마치 컬렉션처럼요. 다만 컬렉션은 동일한 타입만을 다룰 수 있는데 반해 파라미터 팩은 각각의 아이템들이 different 타입일 수 있다는 점입니다. 즉 파라미터 팩을 이용하면 type - level에서 작동할 수 있으며 이는 각각의 유형을 쉽게 처리할 수 있다는 뜻입니다.

좋아요. 파라미터 팩이 뭔지 어느정도 알 거 같죠? 그러면 코드를 통해서 어떻게 작동하는 지 살펴보도록 할게요.

오버로딩 패턴으로 이뤄진 기존의 코드를 파라미터 팩을 이용해서 리팩토링 하겠습니다. item들을 받아서 이것을 튜플로 리턴하는 함수입니다.

```
// Query 
// 아래 코드를 리팩토링 할 것입니다
func query<Payload>(
    _ item: Request<Payload>
) -> Payload

func query<Payload1, Payload2>(
    _ item1: Request<Payload1>,
    _ item2: Request<Payload2>
) -> (Payload1, Payload2)

func query<Payload1, Payload2, Payload3>(
    _ item1: Request<Payload1>,
    _ item2: Request<Payload2>,
    _ item3: Request<Payload3>
) -> (Payload1, Payload2, Payload3)

func query<Payload1, Payload2, Payload3, Payload4>(
    _ item1: Request<Payload1>,
    _ item2: Request<Payload2>,
    _ item3: Request<Payload3>,
    _ item4: Request<Payload4>
) -> (Payload1, Payload2, Payload3, Payload4)
```

실제 코드로 살펴봐야하니 약간 구현을 하도록 하겠습니다.

```
struct Request<Payload> {
  var payload: Payload

  init(payload: Payload) {
    self.payload = payload
  }
}

func query<Payload1, Payload2, Payload3>(
    _ item1: Request<Payload1>,
    _ item2: Request<Payload2>,
    _ item3: Request<Payload3>
) -> (Payload1, Payload2, Payload3) {
  return (item1.payload, item2.payload, item3.payload)
}

print(query(Request(payload: true), Request(payload: 10), Request(payload: "")))
// Print: (true, 10, "")
```

파라미터 팩을 이용한 코드는 each 키워드를 사용해 작성되고 repeat 키워드를 사용해 repetition 패턴을 이용합니다.

```
repeat (Requset<each Payload>)
```


위 코드의 뜻은 패턴이 반복되고, each Payload 자리가 각 반복동안 concrete 타입으로 대체된다는 것을 의미 합니다. 우리가 만든 예시는 `Request<Bool>`, `Request<Int>`, `Request<String>`, `Request<Bool>`, `Request<Int>`가 될 거에요.

```
(Request<Bool>, Request<Int>, Request<String>) == (repeat Reqeust<each Payload>)
```
그러면 아래처럼 함수를 바꿀 수 있게 됩니다.
``` swift
func query<each Payload>(
  _ item: repeat (Request<each Payload>)
) -> (repeat (each Payload)) {
  return (repeat (each item).payload)
}

let result = query(
  Request(payload: true),
  Request(payload: 10),
  Request(payload: "")
)

print(result) // Print: (true, 10, "")
```

#### 파라미터 팩에 프로토콜 제약조건 추가하기

제네릭 Payload에 Equatable 프로토콜을 컨펌하는 타입만 허용하게 하고 싶어요. 그러면 평소 제네릭을 쓰는 것처럼 제네릭 <>에 제약조건을 추가하거나, where 을 이용하면 됩니다.

```
// MARK: - 제네릭에 제약 조건 추가

// 1. 제네릭 선언부에 제약조건

func query<each Payload: Equatable>(
  _ item: repeat (Request<each Payload>)
) -> (repeat (each Payload)) {
  return (repeat (each item).payload)
}

// 2. where Clause 제약조건

func query<each Payload>(
  _ item: repeat (Request<each Payload>)
) -> (repeat (each Payload)) where repeat (each Payload): Equatable {
  return (repeat (each item).payload)
}


let result = query(
  Request(payload: true),
  Request(payload: 10),
  Request(payload: "")
)

print(result)


class TestClass { 
}

// Complie Error: 클래스는 Equatable 만족안함. Equatable 직접 구현하면 컴파일 성공!!
let result2 = query(
  Request(payload: true),
  Request(payload: 10),
  Request(payload: ""),
  Request(payload: TestClass())
)
```

#### 파라미터 팩에 최소한 한개 전달인자를 보장하는 방법

```
func query<each Payload>(
  _ item: repeat (Request<each Payload>)
) -> (repeat (each Payload)) where repeat (each Payload): Equatable {
  return (repeat (each item).payload)
}

var test: () = query() // 컴파일 성공!
```

경우에 따라, 최소한 한개의 길이 보장을 하고 싶어요. 함수 body에 분기코드를 작성해 구현할 수 있으나, 그렇게 되면 런타임 에러에 좀 취약해질 수 있겠죠? 별 차이는 없겠지만 속도가 더 느릴테고요. 이 경우에는 제네릭 선언부에 한개를 더 선언함으로써 쉽게 구현할 수 있습니다. 항상 컴파일러에게 일을 할당해주는 게 좋다고 생각해요.

```
func query<FirstPayload, each Payload>(
  _ first: Request<FirstPayload>, _ item: repeat Request<each Payload>
) -> (FirstPayload, repeat each Payload) where FirstPayload: Equatable, repeat each Payload: Equatable {
  return (first.payload, repeat (each item).payload)
}
```

![](https://blog.kakaocdn.net/dn/S1EJQ/btslw8Cx1gx/cD5Zal2xo39GRSuZpgNau1/img.png)

#### Struct`<Generic>`에 parameter pack 사용하기 (현재기준: 2023년 6월 25일 10:00 PM 빌드 안됨)

 파라미터 팩의 마지막 부분은 Generic에 파라미터 팩을 사용하는 방법입니다. WWDC 영상에서는 이 부분까지 다루긴 합니다. 그런데 아직은 사용하는 것은 시기상조인 것 같습니다. Generic에 파라미터 팩을 사용하는 것은 아직 swift main에 병합되지 않았기 때문입니다. 그래서 코드를 작성하게 되면 아직 실험 단계에 있다고 경고메세지가 뜹니다.

> 현재기준: 2023년 6월 25일 10:00 PM  
> Xcode version: Xcode-15.0.0-Beta.2, Swift Complier version: 5.9

```
$ /Applications/Xcode-15.0.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/swift --version

#swift-driver version: 1.82.2 Apple Swift version 5.9 (swiftlang-5.9.0.114.10 clang-1500.0.29.1) Target: arm64-apple-macosx13.0
```

![](https://blog.kakaocdn.net/dn/bXQUgQ/btsljhIAoYC/m1T6ErQISBbTPPIZJKJBRk/img.png)

실험 Swift를 사용하려면, Swift Setting을 변경해줘야 합니다. 가장 간단하게 할 수 있는 방법은 SPM을 만들어서 package.swift을 수정하는 거에요!

```
// swift-tools-version: 5.9
// The swift-tools-version declares the minimum version of Swift required to build this package.

import PackageDescription

let package = Package(
    name: "VariadicGenerics",
    products: [
        // Products define the executables and libraries a package produces, making them visible to other packages.
        .library(
            name: "VariadicGenerics",
            targets: ["VariadicGenerics"]),
    ],
    targets: [
        // Targets are the basic building blocks of a package, defining a module or a test suite.
        // Targets can depend on other targets in this package and products from dependencies.
        .target(
          name: "VariadicGenerics", swiftSettings:
            [.enableExperimentalFeature("VariadicGenerics")]
        ),
        .testTarget(
            name: "VariadicGenericsTests",
            dependencies: ["VariadicGenerics"]),
    ]
)
```

그러면 Package 내에서 파일을 생성하고 코드를 작성하면 Generic types with parameter packs are experimental 에러는 나오지 않게 됩니다.

추가로 확장하고 싶은 기능은 다음과 같아요.

1. 상태를 저장하는 프로퍼티를 추가하는 것
2. 전달인자로 넘겨주는 타입과 리턴 타입을 다르게 하고 싶음 (differ input type and output type)
3. manage control flow during parameter pack iteration - 파라미터 팩을 사용하는 것은 반복이기 때문에 반복을 조기에 종료했을 때 처리하는 방법

따라서 완성 코드 (애플 제공)는 다음과 같습니다!

```
rotocol RequestProtocol {
// 2번 요구사항 충족
  associatedtype Input
  associatedtype Output
  func evaluate(_ input: Input) -> throws Output
}

struct Evaluator<each Request: RequestProtocol> {
  var item: (repeat each Request) // 1번 요구사항 충족

  func query(_ input: repeat (each Request).Input) -> (repeat (each Request).Output)? {
// 3번 요구사항 충족
    do {
      return (repeat try (each item).evaluate(each input))
    } catch {
      return nil
    }
  }
}
```

그러나, 현재도 Generic Parameter pack에 대한 많은 논의가 이뤄지고 있습니다. 애플이 제공하는 코드도 현재는 에러가 나와서 빌드가 되지 않습니다. 이 부분은 추후 Swift에 이 기능이 정식으로 릴리즈 되면 다시 다뤄봐야 할 것 같습니다.

읽어주셔서 감사합니다!


