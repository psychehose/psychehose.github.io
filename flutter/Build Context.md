
Flutter 앱 위젯 트리 구조로 이뤄져있음. BuildContext는 이 트리 안에서 현재 위젯이 어디에 위치하고 있는지에 대한 정보를 담고 있는 객체임. 즉 위젯 트리내에 존재하는 주소 정보라고 생각하면 될 듯

build()에서 BuildContext 파라미터가 필요한 이유는 이 주소에서의 Widget을 그려야하기 때문이라고 생각하면 됨.


### Build Context로 할 수 있는 것

주로 위젯 트리를 거슬러 올라가서 조상을 찾아 필요한 정보를 얻는데 사용된다.

1. Theme(테마) 정보 얻기

```dart
Color primaryColor = Theme.of(context).primaryColor;
```

2.  다른 페이지로 이동하기

```dart
Navigator.of(context).push(MaterialPageRoute(builder: (context) => CountPage()));
```

3. Scaffold의 기능 사용하기 (SnackBar, Drawer 등)

```dart
ScaffoldMessenger.of(context).showSnackBar(
  SnackBar(content: Text('안녕하세요!'))
);
```


### 중요한 규칙

context는 위치에 따라 다르다것을 주의해야함. BuildContext는 특정 위젯의 build 메서드에 의해 생성된 위젯의 것임.

```dart
@override
Widget build(BuildContext context) {
  // 이 context는 Scaffold를 만드는 위젯의 것.
  // 아직 Scaffold가 트리에 존재하지 않음.
  return Scaffold(
    body: ElevatedButton(
      onPressed: () {
        // 이 context를 사용하면 조상 중에 Scaffold를 찾을 수 없어 에러 발생!
        ScaffoldMessenger.of(context).showSnackBar(...);
      },
      child: Text('Show SnackBar'),
    ),
  );
}


// 올바른 코드

@override
Widget build(BuildContext context) {
  return Scaffold(
    body: Builder(
      // 이 builder는 Scaffold의 자식으로 실행되므로, 새로운 context를 가짐.
      builder: (BuildContext innerContext) {
        return ElevatedButton(
          onPressed: () {
            // 이 innerContext는 Scaffold보다 아래에 있으므로,
            // 조상을 거슬러 올라가 Scaffold를 찾을 수 있음!
            ScaffoldMessenger.of(innerContext).showSnackBar(...);
          },
          child: Text('Show SnackBar'),
        );
      },
    ),
  );
}
```