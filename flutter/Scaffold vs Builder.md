build 함수에서 어떤 경우에는 Scaffold를 리턴하고 어떤 경우에는 다른 Builder들을 리턴하는 경우가 있다. 

### Scaffold
#### Scaffold는 무엇일까?

build 함수에서 위젯을 리턴할 때 위젯의 역할이 무엇인가에 따라 무엇을 리턴할 지에 대해 결정해야함.

Scaffold는 하나의 완전한 화면, 페이지를 구성할 때 최상위 위젯으로 사용됨. Scaffold를 반환한다는 것은 하나의 페이지를 만들겠다와 똑같은 말로 받아들여도 됨.

주요역할은 

1. 머티리얼 디자인의 기본 레이아웃 구조 제공 (appBar, body, floatingActionButton, drawer 등)
2. 화면 전체의 배경색, 상태표시줄과의 상호작용 등 페이지 수준의 시각적 요소를 관리함.

#### Scaffold에서 body가 의미하는 것
body는 그중 **가장 크고 핵심적인 메인 콘텐츠 영역**을 담당

```dart
@override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.grey[100],
      appBar: AppBar(
      // ... app bar ...
      ),
      body: FadeTransition(
        opacity: _fadeAnimation,
        child: Padding( 
          padding: const EdgeInsets.all(16.0),
          child: GridView.builder(
            gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
              crossAxisCount: 2,
              crossAxisSpacing: 16,
              mainAxisSpacing: 16,
              childAspectRatio: 0.75,
            ),
            itemCount: _cards.length,
            itemBuilder: (context, index) {
              return AnimationCardWidget(cardItem: _cards[index], index: index);
            },
          ),
        ),
      ),
    );
  }
}
```


보면 body에 Fade

### Builder

#### Builder

바로 위 부모 위젯의 context가 필요할 때 사용함

```dart
Scaffold(
  body: Builder(
    // 이 builder는 Scaffold의 자식 위치에 있으므로,
    // 새로운 'context'를 통해 위로 올라가 Scaffold를 찾을 수 있습니다.
    builder: (context) {
      return ElevatedButton(
        onPressed: () => Scaffold.of(context).openDrawer(),
        child: Text('Drawer 열기'),
      );
    },
  ),
)
```

#### Animation Builder

애니메이션은 60fps라서 전체 페이지를 60번씩 다시 그린다는 것은 리소스 낭비임. Animation Builder는  특정한 부분만 효율적으로 애니메이션을 하는 최적화 도구임. 애니메이션 값이 변경될 때마다 전체가 아닌 builder 함수 내부의 위젯들만 다시 그린다. 

```dart
// _scaleAnimation이 변경될 떄 Transform.scale 위젯을 다시 그림
 @override
  Widget build(BuildContext context) {
    return AnimatedBuilder(
      animation: _scaleAnimation,
      builder: (context, child) {
        return Transform.scale(
          scale: _scaleAnimation.value,
          child: GestureDetector(
            onTap: () {}

// ...
```


#### ListView.builder & GridView.builder

```dart
ListView.builder(
  itemCount: 1000, // 아이템이 1000개라도
  itemBuilder: (context, index) {
    // 이 코드는 화면에 보이는 몇 개의 아이템에 대해서만 실행됩니다.
    return ListTile(title: Text('Item $index'));
  },
)
```


#### LayoutBuilder

부모 위젯이 제공하는 공간을 알아내고 그 크기에 따라 다른 UI를 보여주고 싶을 때 사용함. 반응형 UI를 만들 때의 핵심

```dart
LayoutBuilder(
  builder: (context, constraints) {
    // constraints 객체에 최대/최소 너비와 높이 정보가 들어있습니다.
    if (constraints.maxWidth > 600) {
      return WideLayout(); // 넓은 화면용 레이아웃
    } else {
      return NarrowLayout(); // 좁은 화면용 레이아웃
    }
  },
)
```


#### FutureBuilder

네트워크 통신이나 데이터베이스 조회처럼 완료되는 데 시간이 걸리는 비동기 작업의 결과를 UI에 표시할 때 사용

```dart
FutureBuilder(
  future: http.get(url), // 이 Future가 완료되기를 기다립니다.
  builder: (context, snapshot) {
    if (snapshot.hasData) {
      return Text('성공: ${snapshot.data}'); // 성공
    } else if (snapshot.hasError) {
      return Text('에러: ${snapshot.error}'); // 에러
    }
    return CircularProgressIndicator(); // 로딩 중
  },
)
```

#### StreamBuilder

일회성이 아닌 지속적으로 들어오는 데이터를 처리할 때 사용. 주로 채팅 앱, 주식 시세 추적, 위치 추적등 실시간으로 UI가 업데이트 되어야 하는 상황에 적합함

```dart
StreamBuilder(
  stream: myChatRoom.messagesStream, // 메시지 스트림을 구독합니다.
  builder: (context, snapshot) {
    if (snapshot.hasData) {
      // 새 메시지가 올 때마다 이 부분이 다시 빌드됩니다.
      return ListView.builder(...);
    }
    return Text('메시지 기다리는 중...');
  },
)
```
