
간만의 iOS 포스팅입니다.

WWDC 2023에서 ****Generalize APIs with parameter packs**** 를 보고 있습니다.

연관 영상으로 타고 내려가다가 WWDC 2022 Design Protocol Interfaces in Swift를 보게 되었습니다.

영상에서, lazy squence를 이용한 예시가 나왔는데, 평소에 제가 쓰지 않는 Collection이라서 포스팅하게 되었습니다.

#### 정의 (애플 공식 문서)

> A Sequence containing the same elements as a **Base** collection, but on which some operations such as **map** and **filter**  are implemented **lazily**.

```
protocol LazySequenceProtocol : Sequence

@frozen
struct LazySequence<Base> where Base : Sequence
typealias LazyCollection<T> = LazySequence<T> where T : Collection


/**

protocol LazySequenceProtocol : Sequence의 Instance Method

compactMap()
drop()
filter()
flatMap()
joined()
map()
prefix()

**/
```

lazy 키워드는 sequence가 처리되는 방식을 변경합니다. 예를 들면 lazy 키워드를 사용하지 않은 경우, 전체 Sequence를 처리하고 새로운 Sequence가 저장됩니다.

lazy가 사용된 경우에 sequence의 값은 downstream 함수에서 요청 시에 생성됩니다. 값은 저장되지 않고 필요할 때 생성됩니다.

코드로 비교를 하면서 확인해 보겠습니다.

양의 정수를 요소로 가지는 배열이 있을 때 짝수로 필터링하고 2배를 하는 코드를 작성할게요. 

```
// lazy keyword를 사용하지 않은 경우

var numbers: [Int] = [1, 2, 3, 6, 9]

let modifiedNumbers = numbers
    .filter { number in
        print("Even number filter")
        return number % 2 == 0
    }.map { number -> Int in
        print("Doubling the number")
        return number * 2
    }

print(modifiedNumbers)

// 출력결과: 
/**
Even number filter
Even number filter
Even number filter
Even number filter
Even number filter
Doubling the number
Doubling the number
[4, 12]
**/
```

modifiedNumbers에서 filter() 코드 블록이 upstream 함수이고, map() 코드 블록이 downstream 함수입니다. 출력된 결과를 확인하면 upstream 함수인 filter를 다 돌고 os에서 임시로 storage에 [2,6]을 할당하고 map 함수를 통과하고 있습니다.

이제 lazy가 사용된 경우를 볼게요. 

```
// lazy keyword 사용

var numbers: [Int] = [1, 2, 3, 6, 9]

let modifiedLazyNumbers = numbers.lazy
     .filter { number in
         print("Lazy Even number filter")
         return number % 2 == 0
     }.map { number -> Int in
         print("Lazy Doubling the number")
         return number * 2
     }

print(modifiedLazyNumbers)

// 출력결과: 
/**
LazyMapSequence<LazyFilterSequence<Array<Int>>, Int>(_base: Swift.LazyFilterSequence<Swift.Array<Swift.Int>>(_base: [1, 2, 3, 6, 9], _predicate: (Function)), _transform: (Function))
**/
```

출력 결과를 확인했을 때, 어떤 구체적인 값을 나타내지 않습니다. 그렇다면, 다음 코드를 추가하고 다시 실행해 보도록 하겠습니다.

```
print(modifiedLazyNumbers.first!)

// 출력결과: 
/**
LazyMapSequence<LazyFilterSequence<Array<Int>>, Int>(_base: Swift.LazyFilterSequence<Swift.Array<Swift.Int>>(_base: [1, 2, 3, 6, 9], _predicate: (Function)), _transform: (Function))
Lazy Even number filter
Lazy Even number filter
Lazy Doubling the number
4
**/
```

결과가 실제적인 값인 4가 나왔네요. 이렇듯, **lazy는 필요할 때 계산한다.라는** 특징을 확인할 수 있습니다. 좋습니다.

우리는 이러한 맥락에서, lazy 키워드가 붙은 경우에 sequence가 처리되는 방식을 이해할 수 있습니다. lazy를 사용하지 않을 때 모든 filter문을 돌고 map 함수를 도는 반면에, lazy를 사용할 때  각각의 element가 순서대로 함수를 통과하는 모습을 확인할 수 있어요.

그러면 다음 코드의 실행 결과는 무엇일까요?

```
let modifiedLazyNumbers = numbers.lazy
     .filter { number in
         print("Lazy Even number filter")
         return number % 2 == 0
     }.map { number -> Int in
         print("Lazy Doubling the number")
         return number * 2
     }
     .reduce(0) { partialResult, ele in
       return partialResult + 1
     }
```

```
// 출력결과: 
/**
Lazy Even number filter
Lazy Even number filter
Lazy Doubling the number
Lazy Even number filter
Lazy Even number filter
Lazy Doubling the number
Lazy Even number filter
**/
```

.reduce 블록에서 값을 계산해서 처리할 필요가 있기 때문에 위와 같은 결과를 확인할 수 있습니다.

#### lazy sequence를 언제 사용하면 좋을까?

lazy sequence에 대해 알아봤는데요. 어떨 때 사용하면 될까요?

1. intermediate operation이 storage 할당하는 것을 막을 때
2. 불필요한 계산을 피하기 위해서 → 최종 컬렉션의 일부만 필요한 경우
3. downstream process를 더 빨리 시작하고, upstream process가 다 수행될 때까지 기다릴 필요가 없는 경우

예시를 하나 들면, 1부터 10000까지 소수를 찾아서 어떤 작업을 하는 프로그램이 있다고 가정합시다.

(의사코드로 작성)

```
var 양의정수배열 = [1 ... 10000]

양의정수배열
.filter { 
	요소가 소수입니까?
}
.map {
	어떤 작업
}

양의정수배열.lazy
.filter { 
	요소가 소수입니까?
}
.map {
	어떤 작업
}
```

lazy를 사용하지 않은 경우에는 전체 배열을 필터링하고 **결과인 배열의 요소에 대해 어떤 작업**을 하는 반면에

lazy를 사용한 경우에는 배열을 돌 때 요소가 **소수인 경우 바로 어떤 작업**을 하게 됩니다.

끝!

#### 코드

```
import Foundation

var numbers: [Int] = [1, 2, 3, 6, 9]

let modifiedNumbers = numbers
    .filter { number in
        print("Even number filter")
        return number % 2 == 0
    }.map { number -> Int in
        print("Doubling the number")
        return number * 2
    }
//
print(modifiedNumbers)

let modifiedLazyNumbers = numbers.lazy
     .filter { number in
         print("Lazy Even number filter")
         return number % 2 == 0
     }.map { number -> Int in
         print("Lazy Doubling the number")
         return number * 2
     }
//     .reduce(0) { partialResult, ele in
//       return partialResult + 1
//     }


print(modifiedLazyNumbers)
//print(modifiedLazyNumbers.first!)
```

### Ref.

[https://developer.apple.com/documentation/swift/lazysequenceprotocol](https://developer.apple.com/documentation/swift/lazysequenceprotocol)
[https://www.avanderlee.com/swift/lazy-collections-arrays/](https://www.avanderlee.com/swift/lazy-collections-arrays/)
[https://stackoverflow.com/questions/51917054/why-and-when-to-use-lazy-with-array-in-swift](https://stackoverflow.com/questions/51917054/why-and-when-to-use-lazy-with-array-in-swift)
