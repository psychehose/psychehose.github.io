
### Q.
(a) iOS 앱의 라이프사이클 상태들(Not Running, Inactive, Active, Background, Suspended)을 설명하고, 각 상태에서 주의해야 할 점을 말해주세요.

(b) 앱이 백그라운드로 전환될 때 시스템이 자동으로 정지시키는 것들이 있는데, 어떤 것들이 있고 이를 어떻게 대응해야 하나요?

(c) 앱이 백그라운드/포그라운드 전환될 때 Observable 스트림 관리에서 주의할 점이 있나요?


 (a) iOS 앱 라이프사이클 상태
 
1. Not Running
	- 앱이 아예 실행되지 않은 상태
	- 메모리에 로드되지 않음
 
2. Inactive
	- 앱이 포그라운드에 있지만 이벤트를 받지 않음
	- 전화가 오거나, Control Center 열 때 잠깐 거치는 상태
	- 주의점: 애니메이션이나 타이머 일시정지

3. Active
	- 앱이 포그라운드에서 정상 실행 중
	- 사용자 이벤트를 받을 수 있는 상태

4. Background
	- 앱이 백그라운드에서 실행 중
	- 제한된 시간(보통 30초)만 코드 실행 가능
	- 주의점: UI 업데이트 금지

5. Suspended
	- 백그라운드에 있지만 코드 실행 안 함
	- 메모리는 유지되지만 CPU 사용 안 함
	- 주의점: 언제든 메모리에서 제거될 수 있음

(b) 백그라운드에서 자동 정지되는 것들

1. 타이머
2. 애니메이션
3. 네트워크 요청 (일부)


(c) 스트림 관리하기

1. 백그라운드/포그라운드 전환 UI, 애니메이션, 주기적으로 실행하는 것들 (폴링) 중지, 재시작
2. 포그라운드 전환시 데이터 새로고침하기 (사용자 경험 up)
3. 타이머 pause / resume


---


### Q. sizeThatFits() , layoutSubviews()에서 작성한 flex layout 코드는 각각 어떤 역할을 하나?

sizeThatFits()은 필요한 경우에 셀의 적절한 크기를 리턴하는 함수이다. `flex.layout(mode: .adjustHeight)` 에서 높이만 계산하고 TableView에 적절한 높이를 제공한다.

그런 다음에 layoutSubviews()에서는 실제 서브뷰들을 배치한다.


Q. sizeThatFits()에서 pinLayout을 이용해서 고정하는 이유

너비를 고정해야 FlexLayout이 정확한 높이를 계산하기 때문이다. 

```swift
// 예시: 긴 텍스트가 있는 라벨
nameLabel.text = "아주 긴 이름이 들어가서 여러 줄로 표시될 수 있는 텍스트"
nameLabel.numberOfLines = 0
```

이 경우에 Label이 몇 줄로 표시 될 지 모르기 때문에 정확한 높이 계산이 불가능하기 때문이다.