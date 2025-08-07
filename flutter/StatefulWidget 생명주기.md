```dart
class MainPage extends StatefulWidget {
  const MainPage({super.key, required this.title});
  final String title;

  @override
  State<MainPage> createState() => _MainPageState();
}
```



1. 위젯 생성: 파라미터로 title을 받아서 위젯 생성
2. State 생성: 프레임워크가 override 된 **createState**() 메서드로 `State' 객체를 생성함
3. **initState**(): 
	* 최초의 초기화 장소, 위젯 생성 후 build() 메서드가 호출 되기전에 실행됨. 
	* 위젯이 화면에 보이기 전에 필요한 모든 사전 작업 처리
		* 컨트롤러 초기화
		* 리스너 등록
		* 데이터 로딩

4. UI 구축: State의 **build**() 가 호출되어 UI 그림
5. 상태변화: **setState**()가 호출되면 build() 호출되어서 UI 업데이트 일어남
6. dispose():
	* 리소스 정리, 메모리 누수 방지
	*  State 객체가 위젯 트리에서 영구적으로 제거될 때 한번 호출됨. initState랑 쌍이라고 할 수 있음
	* initState에서 설정한 리소스 정리하는 곳




