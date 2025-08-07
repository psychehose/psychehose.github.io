
`Scaffold` 위젯의 appBar 속성에 설정되어 화면 상단의 앱바를 구성함.

예시
```dart
appBar: AppBar(
  // 1. 타이틀 설정
  title: const Text(
    'Card Collection',
    style: TextStyle(fontWeight: FontWeight.bold),
  ),

  // 2. 배경색
  backgroundColor: Colors.white,
  
  // 3. 전경색 (아이콘, 텍스트 기본 색상)
  foregroundColor: Colors.black87,

  // 4. 그림자 깊이
  elevation: 0,

  // 5. 오른쪽 액션 버튼들
  actions: [
    IconButton(onPressed: () {}, icon: const Icon(Icons.search)),
    IconButton(onPressed: () {}, icon: const Icon(Icons.filter_list)),
  ],
),
```


다양한 속성 제공을 하는데 잘 이용하면 거의 모든 디자인 구현 가능함.

1. 왼쪽 영역 (Leading Area)
	* leading:  제목 영역 왼쪽에 표시되는 위젯 (보통 뒤로가기 버튼 넣음)
	* automaticallyImplyLeading: 
		* leading 위젯을 자동으로 추가할지 결정하는 bool 값
		* 기본 값 true, 다른화면에서 현재 화면으로 이동했을 때 자동으로 뒤로가기 버튼 생성해줌
	

2. 제목 영역 (title area)
	* title: 제목 영역에 표시되는 위젯 (보통 Text 넣음)
	* centerTitle: 제목을 가운데로 정렬할 지 결정하는 bool 값 (안드로이드 기본 값: false, iOS: true)
	* titleSpacing: 제목 주변의 수평 간격(여백)을 조절함.
		* titleSpacing: 0인 경우 제목 왼쪽의 여백을 없애는 거임.


3. 레이아웃 및 크기: 앱 바의 전체적인 구조와 크기 변경
	*  toolbarHeight: 앱 바의 기본 높이 조절 기본값(56.0)
	*  bottom: 앱 바의 메인 영역 바로 아래에 위젯 추가
		* 주로 Tab을 넣어서 탭 레이아웃을 만들 때 사용하는듯
		* bottom에 위젯을 넣는다면 그 만큼 높이가 늘어남

4. 고급 효과 및 스타일
	* flexibleSpace:
		*  앱 바의 배경 영역을 채우는 위젯
		*  주로 스크롤과 함께 동적으로 변하는 배경(예시: 이미지가 점점 작아지는 효과)를 만들 때 FlexibleSpaceBar 위젯과 함께 사용됨
	* shape: 앱 바의 모양을 사각형이 아닌 다른 형태로 바꿀 수 있음
	* systemOverlayStyle: 앱바 위에 있는 디바이스 상태 표시줄 스타일을 제어함
	* 예시
```dart
flexibleSpace: FlexibleSpaceBar(
  title: Text("Parallax Effect"),
  background: Image.network(
    'https://picsum.photos/400/200',
    fit: BoxFit.cover,
  ),
),


// 앱 바 하단 모서리를 둥글게 깎기
shape: RoundedRectangleBorder(
  borderRadius: BorderRadius.vertical(
    bottom: Radius.circular(30),
  ),
),

// 앱 바가 어두워서 상태 표시줄 아이콘을 밝게 만들어야 할 때
systemOverlayStyle: SystemUiOverlayStyle.light,
```
