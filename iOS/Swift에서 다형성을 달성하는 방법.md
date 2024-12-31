
이번 글은 WWDC 2022 Embrace Swift generics와 같은 년도인 Design Protocol interfaces in Swift를 보고 포스팅하게 되었습니다.

이 포스팅에서 다음과 같은 내용을 다룹니다.

1. Ad - Hoc 다형성과 Subtype 다형성을 설명하고 코드 구현, 그리고 이 구조들에서 발생하는 문제점

2. 제네릭을 이용한 파라미터릭 다형성을 달성하는 법. 이 부분에서 some 키워드와 any 키워드를 설명하고 interface를 설계하고 코드를 작성하는 방법

---

#### 다형성

> 다형성(多形性, polymorphism; 폴리모피즘)은 그 프로그래밍 언어의 [자료형 체계](https://ko.wikipedia.org/wiki/%EC%9E%90%EB%A3%8C%ED%98%95_%EC%B2%B4%EA%B3%84)의 성질을 나타내는 것으로, 프로그램 언어의 각 요소들([상수](https://ko.wikipedia.org/wiki/%EC%83%81%EC%88%98), [변수](https://ko.wikipedia.org/wiki/%EB%B3%80%EC%88%98_(%EC%BB%B4%ED%93%A8%ED%84%B0_%EA%B3%BC%ED%95%99)), [식](https://ko.wikipedia.org/wiki/%EC%8B%9D), [오브젝트](https://ko.wikipedia.org/wiki/%EC%98%A4%EB%B8%8C%EC%A0%9D%ED%8A%B8), [함수](https://ko.wikipedia.org/wiki/%ED%95%A8%EC%88%98), [메소드](https://ko.wikipedia.org/wiki/%EB%A9%94%EC%86%8C%EB%93%9C) 등)이 다양한 자료형(type)에 속하는 것이 허가되는 성질을 가리킨다.

즉 하나의 코드로 여러 행동을 하게 해주는 것이 다형성이라고 생각하면 될 것 같아요. 다형성은 다양한 방식으로 달성할 수 있는데요. 

WWDC 2022에서는 3가지로 설명하고 있습니다.

1. ad-hoc 다형성 - 오버로딩

2. Subtype 다형성 - 오버라이딩

3. 파라미터릭 다형성 using Generic

그러면 하나씩 살펴보도록 하겠습니다.

설명하기에 앞서 코딩을 해야 하기 때문에, 요구사항을 먼저 정하겠습니다. 다음과 같은 프로그램을 개발해야 합니다.

> 농장 시뮬레이팅 만들기!  
> Farm (농장)에는 **Cow**(소),**Horse**(말),**Chicken**(닭)이 있습니다.  
> 소는 **Hay**(건초), 말은 **Carrot**(당근), 닭은 Grain(사료)을 먹습니다.  
> 농장은 동물들에게 밥을 줘야 하기 때문에, Hay, Carrot, Grain을 수확해야 함. Alfalfa가 자라서 Hay가 되고, Root는 Carrot, **Wheat**가 자라서 Grain이 됩니다.

#### 1. Adhoc 다형성

```
// Animal.swift

struct Cow {
  func eat(_ food: Hay) {
    print("\(Self.self) eat \(food)")
  }
}

struct Horse {
  func eat(_ food: Carrot) {
    print("\(Self.self) eat \(food)")
  }
}

struct Chicken {
  func eat(_ food: Grain) {
    print("\(Self.self) eat \(food)")
  }
}
```

```
// AnimalFeed.swift

// 재료 -> 음식
// 소는 Alfalfa가 자라서 수확된 Hay를 먹음
// 말은 Root가 자라서 수확된 Carrot을 먹음
// 닭은 Wheat가 자라서 수확된 Grain을 먹음

struct Hay {
  static func grow() -> Alfalfa {
    return Alfalfa()
  }
}

struct Alfalfa {
  func harvest() -> Hay {
    return Hay()
  }
}

struct Carrot {
  static func grow() -> Root {
    return Root()
  }
}

struct Root {
  func harvest() -> Carrot {
    return Carrot()
  }
}

struct Grain {
  static func grow() -> Wheat {
    return Wheat()
  }
}

struct Wheat {
  func harvest() -> Grain {
    return Grain()
  }
}
```

```
// Farm.swift

struct Farm {
  func feed(_ animal: Cow) {
    let alfalfa = Hay.grow()
    let food = alfalfa.harvest()
    animal.eat(food)
  }

  func feed(_ animal: Horse) {
    let root = Carrot.grow()
    let food = root.harvest()
    animal.eat(food)
  }

  func feed(_ animal: Chicken) {
    let wheat = Grain.grow()
    let food = wheat.harvest()
    animal.eat(food)
  }
}
```

```
// FarmSimulator.playgroud

import Foundation

let farm = Farm()

let cow = Cow()
let horse = Horse()
let chicken = Chicken()



farm.feed(cow) // Print: Cow eat Hay()
farm.feed(horse) // Print: Horse eat Carrot()
farm.feed(chicken) // Print: Chicken eat Grain()
```

Ad-Hoc 다형성을 이용해서 요구사항을 만족하는 코드를 작성했습니다. 코드를 보면 유형에 따라 구체적인 feed() 함수가 결정되고 실행되는 것을 볼 수 있습니다. 이것은 함수 오버로딩 (Function overloading)을 통해서 작동하고 있어요(파라미터의 타입을 다르게 동일한 함수를 작성하는 것)

![](https://blog.kakaocdn.net/dn/bad809/btskUMGVFT1/3Wfq3cbT9hTwOGWwsvBIk0/img.png)

이 방식의 문제점은 무엇일까요? 오버로딩은 반복적인 구현, 즉 보일러 플레이트를 유발합니다. 코드를 너무 많이 작성해야 한다는 것이죠. 그리고 요구사항 변경에 매우 취약합니다. 한 가지 예를 들어보도록 할게요.

요구사항 변경

> 농장에 새로운 동물 돼지가 들어왔어요. 돼지는 GrainForPig를 먹습니다. 그리고 돼지가 먹는 GrainForPig는 옥수수 Corn으로 만들어집니다.

```
// Animal.swift 에 추가해야함
struct Pig {
  func eat(_ food: GrainForPig) {
    print("\(Self.self) eat \(food)")
  }
}

// AnimalFeed.swift
struct GrainForPig {
  static func grow() -> GrainForPig {
    return GrainForPig()
  }
}

struct Corn {
  func harvest() -> GrainForPig {
    return GrainForPig()
  }
}

// Farm.swift

func feed(_ animal: Pig) {
   let corn = GrainForPig.grow()
   let food = corn.harvest()
   animal.eat(food)
}

// FarmSimulator.playgroud

let pig = Pig()
farm.feed(pig) // Print: Chicken eat Grain()
```

변경된 요구사항을 만족하기 위해서 위의 코드를 추가로 작성해야 합니다. 새로운 기능이 추가된 것도 아닌데 작성할 코드가 너무 많아 보입니다. 그리고 더 중요한 건 컴파일러는 이러한 요구사항에 일절 관여하지 않기 때문에 코딩할 때 feed 함수를 오버로딩 해야 한다는 것을 깜빡해도 경고를 하지 않는다는 것입니다. 디버깅 비용이 높아지고 있어요. 딱 봐도 안 좋은 방식인 것 같습니다.

#### 2. Subtype 다형성

Subtype 다형성이 작동하는 메커니즘은 **class**를 이용한 상속입니다. 한 Super Class가 있고 이를 **상속해서** 관련 함수를 **오버라이딩(overriding)** 합니다. 여기에서는 Animal이라는 Super Class를 만들고, Cow, Horse, Chicken은 Animal을 상속하게 합니다.

```
import Foundation

// Animal.swift

class Animal {
  func eat(_ food: Any) { fatalError(" Subclass implements 'eat' ") }
}


class Cow: Animal {
  override func eat(_ food: Any) {
    guard let food = food as? Hay else { fatalError("소는 \(food) 를 먹을 수 없습니다.") }
    print("\(Self.self) eat \(food)")
  }
}

class Horse: Animal {
  override func eat(_ food: Any) {
    guard let food = food as? Carrot else { fatalError("말은 \(food) 를 먹을 수 없습니다.") }
    print("\(Self.self) eat \(food)")
  }

}

class Chicken: Animal {
  override func eat(_ food: Any) {
    guard let food = food as? Grain else { fatalError("닭은 \(food) 를 먹을 수 없습니다.") }
    print("\(Self.self) eat \(food)")
  }
}

// Farm.swift
struct Farm {
  func feed(_ animal: Animal) {

    if animal is Cow {
      let alfalfa = Hay.grow()
      let food = alfalfa.harvest()
      animal.eat(food)

      return
    }

    if animal is Horse {
      let root = Carrot.grow()
      let food = root.harvest()
      animal.eat(food)

      return
    }

    if animal is Chicken {
      let wheat = Grain.grow()
      let food = wheat.harvest()
      animal.eat(food)

      return
    }

    fatalError("올바른 동물 타입이 아닙니다.")
  }
}

// AnimalFeed.swift는 변경 안함
// FarmSimulator.playground도 변경 안함
```

소, 닭, 말은 다른 먹이 타입을 먹기 때문에 eat()의 파라미터 food가 구체적일 수가 없습니다. 따라서 Animal 클래스에서의 eat의 파라미터인 food의 타입을 구체적인 타입이 아니라 **유연한 Any 타입**으로 변경해야 합니다.

이제 각각의 구체적인 동물 클래스(소, 닭, 말)에서 eat 메서드를 구현합시다. 이때 주의할 점이 있습니다. eat()을 구현할 때 guard let을 이용해서 타입 검사를 해야 합니다. 왜냐하면 eat의 파라미터가 Any이기 때문에 모든 타입이 전달인자(argument)로 들어올 수 있기 때문이에요.(예를 들면 Cow의 eat에 Hay 말고 다른 것이 들어오면 안 됨!)

다음으로 Farm을 보면 feed() 구현현도 조금 바뀌었죠? 함수 오버로딩을 사용하지 않고 진정한 의미의 함수 하나가 되었습니다.

동물에게 밥을 주기 위해서 인자로 받은 동물의 타입 검사를 하고 동물 타입에 맞게 밥을 주는 코드가 완성되었습니다.

이제 실행을 해볼까요? FarmSimulator.playground 파일을 수정하지 않고도 잘 실행되는 것을 확인할 수 있습니다.

![](https://blog.kakaocdn.net/dn/bTBLKV/btskSn2eIaz/N9VlHTbYzVAiKOu4p4qXVk/img.png)![](https://blog.kakaocdn.net/dn/vdJ0q/btskRekEX7G/RC4a2SzXov76lkcMoEtv9k/img.png)

이제 하나의 코드로 여러 행동을 할 수 있게 되었습니다. 짝짝 😀

좋습니다. 하지만 이 코드도 확실히 문제점이 존재합니다. 위처럼 새로운 동물 Pig가 추가될 때, Pig 내부에 eat을 구현하지 않으면 컴파일 타임에 에러 검출을 하지 못합니다. 앱이 실행되고 나서 런타임에 Pig가 eat을 구현하지 않았다는 에러를 발견할 수 있습니다.

마찬가지로, eat 메서드에서 전달인자로 올바른 타입이 왔는지 검사하는 것도 모두 다 런타임에서 일어납니다.

즉, 이 코드 역시 보일러플레이트를 생성할 뿐만 아니라, 에러 검출이 런타임에서 이뤄지기 때문에 디버깅 난이도가 올라갑니다. 그리고 Any 타입을 사용하기 때문에 코드 파악이 어려운 단점도 존재합니다.

문제점을 정리해 볼게요.

1. 런타임시에 에러 검출
2. 구체적인 동물 타입 구현시에 파라미터 타입이 Any → 코드 파악 어려움

여기서 새로운 개념을 하나만 더 도입하면 문제점 2번은 해결할 수 있을 것 같아요. 조금 더 나아가보도록 할게요.

타입파라미터를 도입할게요!

```
class Animal<Food> {
  func eat(_ food: Food) { fatalError(" Subclass implements 'eat' ") }
}


class Cow: Animal<Hay> {
  override func eat(_ food: Hay) {
    print("\(Self.self) eat \(food)")
  }
}

class Horse: Animal<Carrot> {
  override func eat(_ food: Carrot) {
    print("\(Self.self) eat \(food)")
  }

}

class Chicken: Animal<Grain> {
  override func eat(_ food: Grain) {
    print("\(Self.self) eat \(food)")
  }
}

// Farm.swift
struct Farm {
  func feed<T>(_ animal: Animal<T>) where T: Any {

    if animal is Cow {
      let alfalfa = Hay.grow()
      let food = alfalfa.harvest()
      animal.eat(food as! T)

      return
    }

    if animal is Horse {
      let root = Carrot.grow()
      let food = root.harvest()
      animal.eat(food as! T)

      return
    }

    if animal is Chicken {
      let wheat = Grain.grow()
      let food = wheat.harvest()
      animal.eat(food as! T)

      return
    }

    fatalError("올바른 동물 타입이 아닙니다.")
  }
}
```

이 방법으로 2번의 문제를 조금 해결할 수 있네요. 하지만 근본적인 문제는 해결되지 않았습니다. Farm의 feed()에서, 각 동물들의 eat을 실행하기 위해서 역시 유연한 Any 타입을 사용해야 합니다. 그리고 실제 eat 코드를 실행할 때, T로 캐스팅을 해줘야 한다는 점이 불편해 보여요.

즉, 코드 누락 시에 컴파일러는 알려주지 않고, 런타임 시에 에러가 검출되겠네요. 역시 1번 문제는 해결하지 못했습니다.

문제가 이것뿐만이 아닙니다. 이 밖에도 다음과 같은 문제가 발생합니다!!

동물의 주요 행위가 꼭 먹는 것이라고 할 수 없는데, class에 타입 파라미터를 지정해줘야 한다는 것입니다. 동물의 행위에는 여러 가지가 있고 eat은 그중 하나일 뿐인데요! (예를 들면 동물 메서드에 run(), sleep() 등등이 있는데 이것은 AnimalFeed와 무관합니다.)

그리고 만약 Food 같이 다른 파라미터도 다형성을 구현하기 위해서 아래 코드처럼 보일러 플레이트를 생성하게 될 것입니다.

```
class Animal<Food, Habitat, Commodity> {
  func eat(_ food: Food) { fatalError(" Subclass implements 'eat' ") }
}


class Cow: Animal<Hay, Barn, Milk> {
  override func eat(_ food: Hay) { }
}

// 말 같은 경우에 얻을 Commodity가 없는데, 상위 클래스인 Animal에 타입파라미터로
// Commodity가 있기 때문에 Never 같은 보일러 플레이트가 발생함
class Horse: Animal<Carrot, Stable, Never> {
  override func eat(_ food: Carrot) { }

}

class Chicken: Animal<Grain, Coop, Egg> {
  override func eat(_ food: Grain) { }
}
```

Subtype 다형성에서 가장 큰 문제점은 바로 Super Type이 Data Type이라는 겁니다.

#### 3. 파라미터릭 다형성

지금까지 다형성을 달성하는 방법 2가지를 찾아봤는데 단점이 너무 명백해 보입니다. 이제 우리가 원하는 것을 생각해 보고 새로운 단계로 나아가봅시다!!

우리가 원하는 것은 다음과 같아요.

1. 기능이 작동하는 방식에 대한 세부 정보 없이 유형의 기능을 나타내길 원함
2. 컴파일러가 좀 더 엄격하게 버그를 찾아줬으면 좋겠음(누락된 코드나 잘못 작성하면 컴파일 타임에 알려줘!)

이때 우리에게 한 줄기의 빛이 되는 것이 바로 interface입니다. 현대 언어에서는 위에서 살펴본 문제들을 interface를 통해 해결하고 있어요. swift에서는 protocol로 선언할 수 있습니다.

```
// 구현체 Animal.swift
// Interface
protocol Animal {
   func eat(_ food: AnimalFeed)
}

struct Cow: Animal {
  func eat(_ food: AnimalFeed) {
    print("\(Self.self) eats \(food)")
  }
}

struct Horse: Animal {
  func eat(_ food: AnimalFeed) {
    print("\(Self.self) eats \(food)")
  }
}


struct Chicken: Animal {
  func eat(_ food: AnimalFeed) {
    print("\(Self.self) eats \(food)")
  }
}


// AnimalFeed.swift
// interface
protocol AnimalFeed {

}
struct Hay: AnimalFeed {
  static func grow() -> Alfalfa {
    return Alfalfa()
  }
}

struct Alfalfa {
  func harvest() -> Hay {
    return Hay()
  }
}

struct Carrot: AnimalFeed {
  static func grow() -> Root {
    return Root()
  }
}

struct Root {
  func harvest() -> Carrot {
    return Carrot()
  }
}


struct Grain: AnimalFeed {
  static func grow() -> Wheat {
    return Wheat()
  }
}

struct Wheat {
  func harvest() -> Grain {
    return Grain()
  }
}


// Farm.swift
struct Farm {
  func feed(_ animal: Animal) {

    if animal is Cow {
      let alfalfa = Hay.grow()
      let food = alfalfa.harvest()
      animal.eat(food)

      return
    }

    if animal is Horse {
      let root = Carrot.grow()
      let food = root.harvest()
      animal.eat(food)

      return
    }

    if animal is Chicken {
      let wheat = Grain.grow()
      let food = wheat.harvest()
      animal.eat(food)

      return
    }
  }
}
```

만약 여기에서 Pig가 추가되었다는 기획의 요청을 받으면, Pig 구현체를 만들 때 Animal을 컨펌하면 되겠습니다. 그러면 eat 메서드를 누락하면 컴파일러가 이를 알아채고 고치라고 합니다! ~~아무래도 컴파일러가 알려주니 좀 뇌를 빼고 개발해도 되겠죠?~~

좋습니다. 좋아요. 하지만 위의 코드에도 문제점을 그만 찾고 싶은데 당연히 문제점이 있습니다. Cow, Horse, Chicken과 같은 구현체에서 eat 메서드의 파라미터를 보니 AnimalFeed입니다. 타입이 인터페이스죠? 말은 Hay를 먹고, Horse는 Carrot, Chicken은 Grain을 먹는다는 사실은 자명합니다. 이 구현체에서 매개변수를 **인터페이스로 은닉할 필요가 없다는 뜻**입니다! 이때 **associatedtype** (연관타입)

을 사용하면 됩니다. Animal에  **associatedtype을** 추가합시다.

```
protocol Animal {
   associatedtype Feed: AnimalFeed
   func eat(_ food: Feed)
}
```

![](https://blog.kakaocdn.net/dn/ldB46/btskRomUwNB/4Kays19TdOCR4Qcgd1cgnk/img.png)

그러면 위처럼 각 동물을 구현할 때 Animal을 컨펌하라고 컴파일러가 알려줍니다. 이제 _ food: AnimalFeed를 구체적인 타입으로 변경합시다.

```
struct Cow: Animal {
  func eat(_ food: Hay) {
    print("\(Self.self) eats \(food)")
  }
}

struct Horse: Animal {
  func eat(_ food: Carrot) {
    print("\(Self.self) eats \(food)")
  }
}


struct Chicken: Animal {
  func eat(_ food: Grain) {
    print("\(Self.self) eats \(food)")
  }
}
```

Farm의 feed() 메서드를 봤을 때, 조금 복잡해 보입니다. if 문으로 분기처리를 하지 않고, 조금 명료한 하나의 동작을 코드로 작성하고 싶습니다. 타입 파라미터를 이용해 제네릭 코드를 작성해 봅시다. feed 메서드를 작성할 때, animal의 타입을 지정해줘야 합니다. 매개변수의 타입은 Concrete 타입이어야 합니다. (소, 닭, 말 같은 구현체) 그렇기 때문에 매개변수는 직접적인 protocol이 아니라 아래처럼 제네릭을 이용해서 작성해야 합니다.(두 함수는 표현만 다를 뿐 같은 함수입니다.)

```
struct Farm {
  func feed<A: Animal>(_ animal: A) {  
  }
  func feed<A>(_ animal: A) where A: animal {
  }
}
```

이 제네릭 코드를 이해해 볼까요?

A에는 특정한 Concrete 타입이 들어가고, 이 Concrete 타입은 Animal Protocol을 컨펌한다는 뜻입니다.

위의 두 메서드와 같은 함수를 하나 더 작성해 보겠습니다. 바로 **some** 키워드를 이용하는 것입니다. some 이용하면 좀 더 의미 있게 표현이 되는 것 같습니다. some 키워드에 대해 알아봅시다.

#### some 키워드

```
struct Farm {
  func feed(_ animal: some Animal) { }
}
```

some 키워드를 이해하기 전에 약간의 개념 정립과 용어 정리가 필요한 것 같습니다.

메서드의 매개변수에 specific concrete 타입이 와야 한다고 말했습니다. some 키워드가 붙은 것도 역시 specific concrete 타입일 것입니다. 하지만 정말 명료한 concrete 타입은 아닐 겁니다. 왜냐하면 프로토콜을 컨펌하는 어떤 구현체가 들어가도 되니깐요. 맞나요? 맞습니다. 하나가 딱 정확히 정해져있지 않다는 뜻입니다. 이러한 것을 **opaque** 타입이라고 합니다.(some 단어로 유추할 수 있죠) 즉 특정한 concrete 타입에 대한 추상적인 placeholder를 opaque 타입이라고 부르는 것입니다. 그래서 애플 문서나 WWDC에서 some 키워드가 붙으면 opaque 타입이라고 부릅니다. 좋아요. 이 opaque 타입은 컴파일 타임에 실제 완전 명료한 concrete type으로 대체가 될 것입니다. 이것을 기저에 깔려있는 즉 underlying 타입이라고 합니다. 이제 의미에 대해 생각해 볼게요. 제네릭 코드를 이용하거나 파라미터 변수에 some을 붙인 것은 어떤 의미가 있을까요? 이러한 코드들이 의미하는 바는 컴파일러에게 이 메서드는 **‘specific concrete 타입으로** 작업할 거야’라고 알리는 것과 마찬가지입니다. 즉 컴파일 시점에 이 opaque 타입의 underlying 타입은 고정되어야만 한다는 것을 의미합니다. 그렇기 때문에 재할당도 불가능합니다. 코드를 보면 알 수 있습니다.

```
// 컴파일 성공
let animals: [some Animal] = [
    Cow(),
    Cow(),
    Cow(),
]

// Print: Compile error: Cannot convert value of type 'Horse' to expected element type 'Cow'
let vehicles: [some Animal] = [
    Cow(),
    Cow(),
    Horse(),
]

// 컴파일 성공
func createCow() -> some Animal {
    return Cow()
}

// Print: Compile error: Function declares an opaque return type 'some Animal', but the return statements in its body do not have matching underlying types
func createSomeAnimal(number: Int) -> some Animal {
    if number == 0 {
        return Cow()
    } else if number == 1{
        return Horse()
    } else {
    	return Chicken()
    }
}



var animal: some Animal = Cow()
animal = Horse() // 불가능
animal = Cow() // 이것도 불가능
```

#### some 키워드와 where 절을 통해 feed 메서드 다형성 달성하기

이제 다시 돌아와서 Farm의 feed 메서드를 작성해 볼까요? some 키워드에 대해 알아보느라 우리의 목표를 잠깐 까먹었을 수도 있기 때문에 리마인드 하겠습니다.

**feed 메서드를 여러 유형에게 공통으로 작동하는 코드로 작성하는 것**

어떻게 작성하면 될까요? 설계를 해봅시다.

```
protocol Animal {
  associatedtype Feed: AnimalFeed
  func eat(_ food: Feed)
}
```

Animal은 연관타입으로 Feed (AnimalFeed)을 가지고 있습니다. 자동완성으로 확인해 볼게요.

![](https://blog.kakaocdn.net/dn/m1JBl/btsk4N6CzrV/s9MFR67PU2a2lF9B77xs11/img.png)

AnimalFeed는 프로토콜이고, concrete 타입으로는 동물들이 먹는 Hay, Carrot, Grain입니다.

위에서 이것을 만들기 위해서 어떻게 했나요? Hay 안에 static 메서드 grow()를 사용했습니다.

간단하게 리뷰를 하면 아래와 같습니다. (소를 예로 들게요.)

let crop = Hay.grow() (crop의 타입은 Alfalfa) → let food = crop.harvest() (food의 타입은 Hay)

opaque 타입은 concrete한 타입이기 때문에, type 메서드를 사용할 수 있어요. type을 이용해 코드를 작성해 볼게요. 그러면, 위의 코드를 type(of: animal). Feed.grow()라고 하면 될 것 같아요.

![](https://blog.kakaocdn.net/dn/bbkEzr/btsk1clWC60/jg1GIMpA2zttu9aXmggnh0/img.png)

당연히 뜨지 않겠죠. 왜냐하면 grow() 메서드는 concrete 타입(Hay, Carrot, Grain)에 직접 구현했으니깐요. 따라서 자연스럽게 AnimalFeed 프로토콜에 에 static grow()가 추가되어야 합니다. grow()는 Alfalfa, Root, Wheat 같은 작물을 리턴해야 하기 때문에 이와 관련된 인터페이스(Crop)를 만들고 연관타입으로 가지고 있어야 합니다. 그리고 Crop 프로토콜에는 당연히 harvest()가 있어야 하겠죠? 이 메서드의 리턴은 역시 연관 타입으로 AnimalFeed를 가지고 있어야 할 겁니다.

코드 작성을 해볼게요.

```
protocol AnimalFeed {
  associatedtype CropType: Crop
  static func grow() -> CropType
}

protocol Crop {
  associatedtype Feed: AnimalFeed
  func harvest() -> Feed
}

struct Hay: AnimalFeed {
  static func grow() -> Alfalfa {
    return Alfalfa()
  }
}

struct Alfalfa: Crop {
  func harvest() -> Hay {
    return Hay()
  }
}

struct Carrot: AnimalFeed {
  static func grow() -> Root {
    return Root()
  }
}

struct Root: Crop {
  func harvest() -> Carrot {
    return Carrot()
  }
}


struct Grain: AnimalFeed {
  static func grow() -> Wheat {
    return Wheat()
  }
}

struct Wheat: Crop {
  func harvest() -> Grain {
    return Grain()
  }
}
```

```
struct Farm {
  func feed(_ animal: some Animal) {
    let crop = type(of: animal).Feed.grow()
    let food = crop.harvest()
    animal.eat(food)
  }
}
```

정말 깔끔한 코드가 작성되었습니다. 다형성을 구현했습니다. 하지만 이 코드는 아래처럼 빌드에 실패할 거예요ㅠ

![](https://blog.kakaocdn.net/dn/ds9HYU/btsk1Bsk9Wf/lL1qYwrMh7YhkKAwAETPL1/img.png)

왜냐하면 프로토콜 관계에 **꼬리에 꼬리를 무는 관계**가 있기 때문이에요. (back - and - forth) AnimalFeed 프로토콜을 한번 봐볼까요.

모든 프로토콜은 Self 타입을 가지고 있습니다.

AnimalFeed는 concrete conforming type인 CropType을 가지고 있습니다.

Crop 역시 concrete conforming type인 FeedType을 가지고 있습니다.

**Self: AnimalFeed → Self.CropType: Crop → Self.CropType.FeedType: AnimalFeed → …**

위와 같이 무한반복이 될 거예요.

Crop 프로토콜도 이와 마찬가지고요. 따라서 이 (back - and - forth)를 끊어주면 되겠습니다. AnimalFeed 프로토콜에서 CropType이 가지고 있는 Feed는 바로 자신이라고 알려주고, 마찬가지로 Crop 프로토콜에서 FeedType이 가지고 있는 Crop은 자기 자신이라는 것을 알려주면 됩니다. 어떻게? where을 이용해서!

```
protocol AnimalFeed {
  associatedtype CropType: Crop where CropType.Feed == Self
  static func grow() -> CropType
}

protocol Crop {
  associatedtype Feed: AnimalFeed where Feed.CropType == Self
  func harvest() -> Feed
}
```

여기까지 작성하면, feed에서 나는 에러가 사라지게 됩니다. FarmSimulating.playground로 돌아가서 실행을 하면 잘 작동합니다.

지금까지 .playground을 단 한 번도 수정한 적이 없습니다. 그럼에도 불구하고 같은 결과를 보여주는 게 정말 멋지네요.

#### Type Erasure

FarmSimulating.playground에서 각 동물 인스턴스와 농장 인스턴스를 만들어서 작업을 하고 있어요. 여기에 요구사항을 약간 변경해 보도록 할게요. 그리고 요구사항에 맞게 코드를 변경해 봅시다.

> 농장은 배고픈 동물들에게 밥을 줄 책임이 있습니다. 배고픈 동물들에게 밥을 주는 기능을 추가하세요.

요구사항을 읽었을 때, 바뀌거나 추가되어야 할 부분들은 다음과 같습니다.

1. 동물은 자기가 배고픈지 아닌지를 알 수 있어야 합니다. → 동물은 배고픔을 나타내는 프로퍼티를 가지고 있어야 함
2. 농장은 배고픈 동물들을 판별할 수 있어야 함 → Farm 구현체에 프로퍼티로 동물들을 가지고 있어야 함

```
// 1번 요구사항 충족
// Animal 인터페이스에 isHungry를 프로퍼티를 추가하고
// 동물 인터페이스를 컨펌하는 concrete 객체에 isHungry를 추가함.
protocol Animal {
  associatedtype Feed: AnimalFeed
  func eat(_ food: Feed)
  var isHungry: Bool { get }
}

struct Cow: Animal {
  var isHungry: Bool

  func eat(_ food: Hay) {
    print("\(Self.self) eats \(food)")
  }
}
 // ...
```

1번 요구사항을 충족시키는 것은 매우 쉽습니다! 2번도 만족시켜볼까요?

```
struct Farm {

  var animals: [some Animal]

  func feed(_ animal: some Animal) {
    let crop = type(of: animal).Feed.grow()
    let food = crop.harvest()
    animal.eat(food)
  }
}
```

![](https://blog.kakaocdn.net/dn/kxFNx/btskZsJ6zyL/xKuqodcdpKxuO4oxPn3P60/img.png)

빌드할 수가 없습니다. 왜 그럴까요? some에서 봤듯이, 컴파일 타임에 opaque type의 underlying type을 유추할 수 있어야 하기 때문입니다. 그러면 다음 코드들은 어떨까요?

```
// case 1 (성공) // underlying type이 고정되어 있음
var animals: [some Animal] = [Cow(isHungry: false), Cow(isHungry: false)]

//case 2 (실패) - underlying type이 고정되어 있지 않음
var animals: [some Animal] = [Horse(isHungry: false), Cow(isHungry: false)]
```

농장에는 다른 유형의 동물들이 있어야 합니다. 우리가 원하는 건 case 2입니다. 컴파일러는 고정된 underlying type을 요구합니다. 우리는 이걸 좀 유연하게 가져가고 싶어요. [Cow(), Horse(), Chicken() … ] 이런 식으로요

어떻게 타입을 지우는 법이 없을까요? 이때 필요한 키워드가 바로 **any입니다.**

#### any 키워드

any 키워드는 **existential type를** 생성하기 위해 도입되었어요. 사실 existential type는 새로운 게 아닙니다. 그냥 프로토콜을 타입으로 사용할 때, 존재 유형으로 부릅니다.

```
var cow: Animal = Cow() // 이 코드에서 Animal이 존재 유형임.
// Swift 5.7에서는 위 코드는 
var cow: any Animal = Cow() 가 되어야만 함.
```

any 키워드를 사용한다는 것은 Apple은 concrete 타입을 박스로 싸는 것에 비유합니다. ㅎ

![](https://blog.kakaocdn.net/dn/1tIyb/btsk2fP5Zek/tlhdodbWyyLohnGMOAQkHk/img.png)

즉 박스에 특정한 프로토콜을 준수하는 concrete 타입을 넣는 겁니다. 그렇다면 concrete 타입( Cow, Chicken, Horse)은 숨겨집니다. 그래서 이것을 type erasure라고 합니다! 즉 upper bound로 퉁치는 거입니다. (수학 공부할 때 정말 많이 나오는 그 upper bound 맞아요) 그래서 any를 사용하게 되면 heterogeneous한 컬렉션([Cow(), Horse()] 같은 컬렉션들)을 만들 수 있게 되는 것입니다. 이해하고 나니까 별로 어려운 것이 아니네요.^^

any 키워드에 주의점을 마지막으로 any에 대한 설명을 끝내겠습니다. ‘Donny Wals라는 개발자는 [‘****What is the “any” keyword in Swift?****](https://www.donnywals.com/what-is-the-any-keyword-in-swift/)’라는 아티클에서 any 키워드는 some보다 덜 효율적이고 비용을 더 많이 초래하기 때문에 기본적으로 some을 사용하고 필요할 때, any 키워드를 사용하라고 권고합니다. 그리고 역시 WWDC 2022에서도 some을 디폴트로 사용하라고 언급해요.

이제 any에 대한 이해가 끝났으니 코드 작성을 해봅시다.

```
struct Farm {

  var animals: [any Animal]

  func feedAnimals() {
    animals.filter(\.isHungry).map { animal in
      animal.eat(???)
    }
  }
}

// FarmSimulating.playground

import Foundation

let farm = Farm(animals: [
  Cow(isHungry: true),
  Cow(isHungry: false),
  Chicken(isHungry: true),
  Horse(isHungry: true)
])

farm.feedAnimals()
```

위의 코드는 작동하지 않을 것입니다. 왜일까요? 답은 간단합니다. 닭은 Grain을 먹어야 하고, 소는 Hay를 먹어야 합니다. 이때 animal.eat()에서 어떠한 방식으로 Parameter를 넘겨야 할지 문제가 생깁니다. Animal 프로토콜과 관련된 FeedTye의 upper bound는 'any AnimalFeed'입니다. any AnimalFeed가 닭인 경우에 Grain, 소인 경우에 Hay를 정적으로 보장할 방법이 없습니다. 따라서 위 코드처럼 Type erasure를 허용해서는 안됩니다. 즉 정적으로 보장하기 위해서 existential 타입을 언박싱해야 합니다. 언박싱하는 방법은 opaque 타입인 some 키워드를 매개변수로 하는 함수를 호출하면 됩니다. 바로 아래처럼요!

```
struct Farm {

  var animals: [any Animal]

  private func feed(_ animal: some Animal) {
    let crop = type(of: animal).Feed.grow()
    let food = crop.harvest()
    animal.eat(food)
  }

  func feedAnimals() {
    animals.filter(\.isHungry).map { animal in
      feed(animal)
    }
  }
}

// FarmSimulating.playground

import Foundation

let farm = Farm(animals: [
  Cow(isHungry: true),
  Cow(isHungry: false),
  Chicken(isHungry: true),
  Horse(isHungry: true)
])

farm.feedAnimals()
```

드디어 모든 코드 구현이 완료되었습니다. some과 any 키워드를 사용하고 인터페이스를 잘 설계함으로써 제네릭을 이용한 파라미터릭 다형성을 달성했습니다.

---

 제가 이해한 것을 바탕으로 최대한 이해하기 쉽게 풀어쓰려고 노력했는데 잘 와닿을까 모르겠네요. 제네릭은 참 어려운 것 같습니다.

상당히 긴 호흡의 글을 읽어주셔서 감사합니다.

### 전체 Source Code


https://github.com/psychehose/FarmSimulating

#### Ref.
https://swiftsenpai.com/swift/understanding-some-and-any/
https://www.donnywals.com/what-is-the-any-keyword-in-swift/
https://developer.apple.com/videos/play/wwdc2022/110352/
https://developer.apple.com/videos/play/wwdc2022/110353/