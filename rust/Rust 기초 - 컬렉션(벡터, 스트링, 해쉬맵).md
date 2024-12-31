
## 컬렉션

컬렉션은 내장된 배열과 튜플 타입과는 다르게 힙에 저장됩니다. 이것은 데이터량이 런타임에서 변경될 수 있다는 뜻입니다. 러스트 표준 라이브러리가 제공하는 컬렉션은 다음과 같습니다.

- sequence 타입: Vec, VecDeque, LinkedList
- map 타입: HashMap, BTreeMap
- Sets 타입: HashSet, BTreeSet
- Misc 타입: BinaryHeap

그중에서 자주 사용되는 세 가지 컬렉션을 알아보겠습니다.

- Vec는 여러 개의 값을 서로 붙어 있게 저장합니다.
- String은 문자의 모음입니다.
- HashMap은 키-값으로 저장합니다.

### Vec

```
let v: Vec<i32> = Vec::new();
```

벡터는 제네릭을 이용해서 구현되어있습니다. 그래서 어떠한 종류의 값도 저장할 수 있습니다.

```
let v = vec![1,2,3];
let v = Vec::from([1,2,3,4]);
let v = vec![0; 5]; // [0,0,0,0,0]
```

초기값이 있는 벡터를 생성하는 것은 흔한 일이기 때문에 러스트에서는 vec! 매크로를 제공하고 있습니다. Vec의 static method인 from을 이용해서도 배열을 생성할 수 있습니다. 길이가 n인 배열을 생성하고 0으로 초기화할 수 있습니다.

```
let mut v = Vec:new();
v.push(5);
v.push(6);
v.push(7);
v.pop();
v.pop();

// insert(index, value)
// remove(index)
v.insert(1,4)
v.remove(1)

// 

let mut vec = vec![1, 2, 3];
let mut vec2 = vec![4, 5, 6];
vec.append(&mut vec2);
assert_eq!(vec, [1, 2, 3, 4, 5, 6]);
assert_eq!(vec2, []);
```

컬렉션 변수에 대해서 그 변수가 담고 있는 값이 변경되게 하려면 mut 키워드를 사용합니다. push & pop을 사용해서 벡터 값을 자료구조 스택처럼 사용할 수 있습니다.

특정한 인덱스에 값을 넣고 빼기 위해서는 insert & remove를 사용하면 됩니다. Insert & remove는 인덱스를 직접 사용하기 때문에 인덱스 범위를 넘어가면 패닉을 일으키게 됩니다.

배열과 배열의 연산에는 append를 이용합니다. 특이한 점이 append를 하게 되면 추가 하는 항목들이 있는 배열에서 요소들이 사라집니다.

```
let v = vec![1, 2, 3, 4, 5];

let third: i32 = v[2];
let third: Option<&i32> = v.get(2);
```

벡터 요소를 읽기 위해서는 []과 get 함수를 이용하는 방법입니다. []를 이용하는 경우 존재하지 않는 요소를 참조하게 되면 패닉을 일으키게 되겠죠?

get 함수를 사용하면 벡터 범위를 벗어난 인덱스를 사용하면 퍄닉 없이 None이 반환됩니다.

```
let v = vec![100, 32, 57];
for i in &v {
    println!("{}", i);
}

///

let mut v = vec![100, 32, 57];
for i in &mut v {
    *i += 50;
}
```

인덱스를 명시하지 않아도, Swift나 Python처럼 반복문을 돌면서 요소에 접근할 수 있습니다. 요소들을 변경시키길 원한다면 역참조연산자(*)를 사용하면 됩니다.

### String

String은 문자들의 모음입니다. 러스트에서 스트링은 세 가지 특징적인 개념을 가집니다.

1. 에러를 꼭 명시해줘야 합니다.
2. 복잡한 데이터 구조를 가지고 있습니다. 
3. UTF-8이라는 점입니다.

```
let mut s = String::new();
//

let data = "initial contents";
let s = data.to_string();

// the method also works on a literal directly:
let s = "initial contents".to_string();

let s = String::from("initial contents");
```

스트링 생성은 String 타입의 new() 정적 메소드를 이용해서 할 수 있습니다. 그리고 스트링 리터럴(&str)을 생성하고. to_string() 메서드를 이용해서 생성할 수도 있습니다.

```
let mut s = String::from("foo");
s.push_str("bar");


let mut s1 = String::from("foo");
let s2 = "bar";

// 참조자를 사용하기 때문에 소유권을 넘기지 않음.
s1.push_str(&s2);
println!("s2 is {}", s2);
```

스트링에 스트링을 추가하기 위해서 push_str 메소드를 사용할 수 있습니다.  문자 한 개를 추가하길 원하면 push() 메서드를 사용할 수 있습니다. 

```
let s1 = String::from("Hello, ");
let s2 = String::from("world!");
let s3 = s1 + &s2; // s1은 여기서 이동되어 더이상 쓸 수 없음을 유의하세요
```

+ 연산자를 이용해서 문자열끼리 접합할 수 있습니다. 위의 코드를 보면 let s3 = s1 + &s2; 입니다. s1에는 참조자가 붙지 않고 s2에 참조자가 붙어 있는 것을 볼 수 있는데요. 이것은 스트링에서 + 연산자는 add 메서드를 이용하기 때문입니다.

```
fn add(self, s: &str) -> String {
```

add()를 자세하게 보고 s3의 코드를 보면 조금 당황할 수도 있습니다. 왜냐하면, add의 파라미터는 &str 타입인데 &s2의 타입은 &String이기 때문입니다. 하지만 컴파일러는 이를 컴파일 할 수 있습니다. 왜 그럴까요? &String인자는 &str로 강제될 수 있기 때문입니다. 이를 러스트에서는 역참조 강제(deref coercion)라고 합니다.

위에서 보듯 +의 동작은 다루기가 불편한 것처럼 보입니다. 어떤 것은 String이고, 붙여지는 건 &str인 것처럼요. 왜냐하면 이는 소유권에 대해서 더 신경을 써야한다는 것을 의미하니까요. 그래서 러스트는 format! 매크로를 제공합니다. 이를 이용하면 어떠한 파라미터의 소유권도 가져오지 않고 보기 쉽게 스트링을 합칠 수 있습니다. 

```
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let s = s1 + "-" + &s2 + "-" + &s3;
//

let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let s = format!("{}-{}-{}", s1, s2, s3);
```

```
let s1 = String::from("hello");
let h = s1[0];
```

다른 언어처럼 스트링에 인덱스로 접근하고 싶습니다. 위의코드는 컴파일이 될까요? 접근하면 어떻게 될까요? 러스트에서 스트링 인덱스 접근은 지원하지 않습니다. 이것을 이해하기 위해서 스트링이 어떻게 메모리에 저장되는지 확인해봐야 합니다.

String은 Vec<u8>을 감싼 것입니다. 

```
let len = String::from("Hola").len(); // 1

let len = String::from("Здравствуйте").len();
```

1. len은 4입니다. 이건 4byte 길이라는 뜻입니다.
2. len은 24입니다. 글자는 12개지만, 이 문자 각각의 유니코드 스칼라 값이 2입니다. 12* 2 = 24

이 말은 스트링의 바이트들 안의 인덱스는 유니코드 스칼라 값과 항상 대응되지 않는다는 것을 의미합니다.

그리고 인덱스 연산은 언제나 상수 시간에 실행될 것으로 기대를 받는데, String으로 그러한 성능을 보장하는 것은 불가능입니다. 왜냐하면 스트링 내에 얼마나 많은 문자가 있는지 알아내기 위해 시작지점부터 인덱스로 지정된 곳까지 살펴봐야 하기 때문입니다.

만약 스트링 슬라이스를 만들기 위하고 인덱스를 사용하길 원한다면 구체적으로 지정해야 합니다.

```
let hello = "Здравствуйте";

let s = &hello[0..4]; // Зд
// 
let s = &hello[0..1]; // 패닉 발생!
```

반복문을 돌면서 바이트에 관계 없이 유니코드 스칼라 값(글자)에 대한 연산을 하길 원하면 chars를 사용하면 됩니다.

```
for c in "नमस्ते".chars() {
    println!("{}", c);
}

for b in "नमस्ते".bytes() {
    println!("{}", b);
}
```

즉 유효한 유니코드 스칼라 값이 하나 이상의 바이트로 구성될지도 모른다는 것을 확실히 기억해야만 합니다.''

#### 해쉬맵

```
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

//

let teams  = vec![String::from("Blue"), String::from("Yellow")];
let initial_scores = vec![10, 50];

let scores: HashMap<_, _> = teams.iter().zip(initial_scores.iter()).collect();
```

추가적으로 튜플의 벡터에 대해 collect 메서드를 사용해서 생성할 수 있습니다

collect 메소드는 데이터를 모아서 컬렉션 타입으로 만들어줍니다.

```
let field_name = String::from("Favorite color");
let field_value = String::from("Blue");

let mut map = HashMap::new();
map.insert(field_name, field_value);
```

insert 할 때, field_name과 field_value의 소유권은 map으로 이동합니다.

```
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

let team_name = String::from("Blue");
let score = scores.get(&team_name);
```

.

.

```
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

for (key, value) in &scores {
    println!("{}: {}", key, value);
}
```

반목문 접근은 다른 컬렉션과 마찬가지로 &를 이용해서 접근할 수 있습니다.

```
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Blue"), 25);

println!("{:?}", scores);
```

위의 코드는 키에 대한 값이 있어도 덮었어 집니다. 

만약 키에 할당된 값이 없을 경우에만 삽입하고 싶으면 다음 entry API를 이용하면 됩니다.

entry의 리턴 값은 Enum 타입 Entry인데 있는지 없는 지를 반환합니다.

Entry의 or_insert 메소드는 해당 키가 존재하는 경우는 Entry 키에 대한 값을 반환하고 아닌 경우에는 새 값을 삽입하고 수정된 Entry에 대한 값을 반환합니다.

```
se std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);

scores.entry(String::from("Yellow")).or_insert(50);
scores.entry(String::from("Blue")).or_insert(50);

println!("{:?}", scores);
```

이걸로 러스트의 콜렉션 타입 중 가장 자주 쓰이는 Vec, String, HashMap에 대해 알아보았습니다.