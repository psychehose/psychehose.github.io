iOS 개발에서  UI 업데이트는 Run Loop를 기반으로 동작한다.

각 Run Loop 사이클마다 순서대로 UI를 업데이트 한다.

1. 이벤트 처리(터치, 타이머 ...)
2. 레이아웃 업데이트
3. 디스플레이 업데이트


### Event Handling

```swift
// 터치 이벤트
override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?)

// 제스처 이벤트  
@objc func buttonTapped(_ sender: UIButton)

// 타이머 이벤트
Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { _ in }

// 네트워크 콜백
URLSession.shared.dataTask(with: url) { data, response, error in }

// 노티피케이션
NotificationCenter.default.post(name: .dataUpdated, object: nil)
```

이벤트 핸들링 흐름

1. 이벤트 큐에서 이벤트 수신
2. Hit Testing
3. Responder Chain을 통한 이벤트 전달
4. 이벤트 핸들러 실행
5. 상태 변경

### Layout cycle

레이아웃 메서드 함수들.

1. setNeedsLayout()

```swift
view.setNeedsLayout() // 다음 업데이트 사이클에서 레이아웃 갱신!
```

* 즉시 실행 되지 않음. 다음 업데이트 사이클에서 레이아웃 갱신을 예약
* 여러번 호출해도 한번만 실행됨


2. layoutIfNeeded()

```swift
view.layoutIfNeeded()
```

* 예약된 레이아웃 작업을 즉시 실행
* 보통 애니메이션과 함께 사용할 때 유용함

3. sizeThatFits(_:)

```swift
let size = view.sizeThatFits(CGSize(width: , height:))
```

* 주어진 크기에 맞는 최적의 크기를 계산하여 변환
* 실제 크기 변경 X
* 직접 호출 or 시스템이 자동으로 호출

4. layoutSubviews()

```swift
override func layoutSubviews() {
    super.layoutSubviews()
    // 하위 뷰들의 프레임 설정
}
```

* 실제 하위 뷰들의 위치와 크기를 설정
* 직접 호출 X, setNeedsLayout()으로 예약 해야함
* 시스템이 필요할 때 자동으로 호출


5. viewDidLayoutSubviews()

```swift
override func viewDidLayoutSubviews() {
    super.viewDidLayoutSubviews()
    // 레이아웃 완료 후 추가 작업
}
```

* 뷰 컨트롤러의 뷰와 하위 뷰들의 레이아웃이 완료된 후 호출
* 레이아웃 기반으로 추가 계신이나 설정을 할 때 사용


```
1. setNeedsLayout() 호출 (레이아웃 갱신 예약)
   ↓
2. 다음 Run Loop 사이클에서...
   ↓
3. sizeThatFits() (필요한 경우)
   ↓
4. layoutSubviews() 
   ↓
5. viewDidLayoutSubviews() (뷰 컨트롤러에서)
```


#### 구체화

오토레이아웃을 사용할 때 순서

1. Layout 예약 확인 (setNeedsLayout()으로 예약된 뷰들을 수집)
2. AutoLayout 제약 조건 확인 및 해결, 우선순위에 따른 제약 조건 적용
3. sizeThatFits가 필요한 경우에 호출 - AutoLayout이 아닌 경우 주로 사용함
4. layoutSubviews() 호출 - 현재 뷰의 하위 뷰들 프레임 설정됨, 상위 뷰 -> 하위 뷰 재귀적으로 호출됨.
5. viewDidLayoutSubviews - 모든 하위 뷰의 레이아웃이 완료된 후 호출

```
1. setNeedsLayout() 예약 처리
2. Auto Layout Engine 동작
   ├── 제약 조건 수집
   ├── 제약 조건 해결 (Linear Programming)
   └── 프레임 계산
3. sizeThatFits() 호출 (필요시)
4. 뷰 계층 구조를 따라 layoutSubviews() 재귀 호출
   ├── 상위 뷰 → 하위 뷰 순서
   └── 각 뷰의 프레임 최종 결정
5. viewDidLayoutSubviews() 호출

```


### 주의사항
- `layoutSubviews()`를 직접 호출하지 말고 `setNeedsLayout()`이나 `layoutIfNeeded()` 사용
- 무한 루프 방지: `layoutSubviews()` 내에서 `setNeedsLayout()` 호출 주의
- 성능을 위해 불필요한 레이아웃 갱신은 피하기
- Auto Layout 사용 시에는 대부분 자동으로 처리됨
