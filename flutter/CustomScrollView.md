CustomScrollView는 플러터에서 고급 스크롤 효과를 구현할 수 있는 위젯이다. Sliver 위젯들을 조합해서 복잡한 스크롤 레이아웃을 생성할 수 있다.

```dart
return Scaffold(
	body: CustomScrollView(
		slivers: [
			
		]
	)
)
```


Sliver란?
Sliver는 얇은 조각을 뜻함. 플러터에서는 스크롤 가능한 영역의 한 조각을 나타낸다. Sliver는 각각 독립적인 스크롤 동작을 가질 수 있다.

CustomScrollView를 구현할 때 SliverAppBar와 SliverToBoxAdapter를 사용했는데 이 클래스들에서 알아본다.

### SliverAppBar

확장 / 축소 가능한 앱바다.

```dart
SliverAppBar(
  expandedHeight: 300,    // 확장된 상태의 높이
  pinned: true,          // 스크롤해도 상단에 고정 - false시 네비게이션 영역 사라짐
  floating: false,       // 스크롤 시 즉시 나타날지 여부
  snap: false,          // floating과 함께 사용, 빠른 애니메이션
  flexibleSpace: FlexibleSpaceBar(...), // 확장 영역 내용
)
```

동작 방식
* 초기에는 설정한 expandedHeight의 사이즈를 가지고 있음
* 스크롤하면 점점 축소되어 일반 앱바 크기가 됨
* pinned가 true라서 완전히 사라지지 않고 상단에 고정됨

### SliverToBoxAdapter

일반 위젯을 Sliver로 변환해주는 어댑터다. Container, Column, Text 등 일반 위젯을 CustomScrollView에서 사용하려면 Sliver로 감싸야한다.이때 SliverToBoxAdapter를 사용한다. 단순한 변환 역할만 수행한다.