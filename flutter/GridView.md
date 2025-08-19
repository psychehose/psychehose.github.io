
GridView는 화면에 보이는 아이템만 동적으로 생성하여 그리드 형태로 보여주는 위젯임. 


## Grid의 공통 속성

1. scrollDirection
	* Axis.vertical
	* Axis.horizontal
2. padding: 그리드 전체의 바깥 쪽에 여백을 줌
3. physics: 스크롤 동작 방식 설정
	* BoundingScrollPhysics() : 스크롤 끝에서 튕기는 효과 (iOS 기본)
	* ClampingScrollPhysics(): 스크롤 끝에서 멈추는 효과 (안드로이드 기본)
	* NeverScrollablePhysics(): 스크롤 막음
	
4. shrinkWrap: 
	* GridView를 Column이나 ListView 같은 다른 스크롤 위젯 안에 넣을 때 true로 설정해야함.
	* 이 속성은 GridView가 자신의 콘텐츠 크기만큼만 공간을 차지하게 만들어 스크롤 충돌 문제를 해결함. 
	* true로 설정한 경우 성능 저하 가능성이 있어서 필요한 경우에만 사용해야함

## GridView의 주요 생성자

1. GridView.builder
2. GridView.count
3. GridView.extent
4. GridView.custom
### GridView.builder

리스트에 수백, 수천 개의 아이템이 있더라도 현재 화면에 보이는 부분과 곧 보이게 될 일부만 미리 그려서 가장 효율적임.

```cpp
GridView.builder(
    // 1. 그리드 레이아웃을 정의하는 속성
    gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
      crossAxisCount: 2,         // 가로(교차축)에 2개의 아이템을 배치
      crossAxisSpacing: 16,      // 아이템 간의 가로 간격
      mainAxisSpacing: 16,       // 아이템 간의 세로 간격
      childAspectRatio: 0.75,    // 아이템의 가로/세로 비율 (가로 1 : 세로 1.33)
    ),
    // 2. 전체 아이템의 개수
    itemCount: _cards.length,
    // 3. 각 아이템을 어떻게 그릴지 정의하는 함수
    itemBuilder: (context, index) {
      return AnimationCardWidget(cardItem: _cards[index], index: index);
    },
)
```

* gridDelegate: 그리드 내 아이템들의 레이아웃을 결정하는 역할
	* SliverGridDelegateWithFixedCrossAxisCount: 교차 축에 고정된 개수의 아이템을 배치하는 방식

* itemBuilder에서 context가 의미하는건 GridView 위젯 내에서 앞으로 생성될 각 아이템이 위치할 자리의 BuildContext임.

### GridView.count

가장 간단한 방법이다. 표시할 아이템 개수가 적고 정해져있을 때, 간단하게 그리드를 만들고 싶을 때 사용함. childern 속성에 위젯 리스트를 직접 전달하기 때문에 모든 아이템을 미리 생성한다. 그래서 아이템이 너무 많다면 성능 저하가 발생한다.

```dart
GridView.count(
  crossAxisCount: 3, // 한 줄에 3개의 아이템을 배치
  children: <Widget>[
    Icon(Icons.home),
    Icon(Icons.search),
    Icon(Icons.settings),
    Icon(Icons.person),
    Icon(Icons.camera),
    Icon(Icons.mail),
  ],
)
```


### GridView.extent

반응형 레이아웃을 구현할 때 주로 사용됨. maxCrossAxisExtent 속성을 이용해서 각 아이템의 최대 너비 (or 높이)를 설정한다. count와 마찬가지로 childern을 통해서 위젯 리스트를 직접 전달한다.

반응형의 이점과 builder의 아이템 성능적 이점을 모두 얻으려면 builder에서 gridDelegate를 SliverGridDelegateWithMaxCrossAxisExtent로 설정하면 됨.

```dart
GridView.extent(
  maxCrossAxisExtent: 200, // 각 아이템의 최대 너비를 200으로 제한
  children: <Widget>[
    // ... 위젯 리스트
  ],
)
```

### GridView.custom

위의 생성자들로 구현하기 어렵고 복잡한 커스텀된 그리드 레이아웃이 필요할 때 사용된다. 특징은 gridDelegate와 childrenDelegate를 직접 구현해야 한다. childrenDelegate은 아이템을 생성하고 관리하는 방식을 완전히 제어할 수 있게 해준다.







