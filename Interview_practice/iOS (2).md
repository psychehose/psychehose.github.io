
Q. sizeThatFits() , layoutSubviews()에서 작성한 flex layout 코드는 각각 어떤 역할을 하나?

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