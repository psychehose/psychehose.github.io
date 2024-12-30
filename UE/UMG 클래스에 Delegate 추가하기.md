## UMG 클래스에 Delegate 추가하기


> 목표: UMultiLineEditableTextBox에 FOnFocusReceviced와 FOnFocusLost를 구현하기


UMultiLineEditableTextBox은 SMultiLineEditableTextBox를 래핑하고 있다. SMultiLineEditableTextBox는 SMultiLineEditableText를 내부적으로 사용하고 있다.



1. 이벤트 전파 체인 이해
   * 따라서 SMultiLineEditableText - > SMultiLineEditableTextBox -> UMultiLineEditableTextBox 체인을 따라 포커스 이벤트가 전파된다. 각 단계에서 처리하고 상위 레벨로 전달함.

2. SMultiLineEditableText 수정
   * 실제 텍스트 편집 기능 담당
   * 포커스 이벤트가 발생하는 가장 하위 레벨
   * Slate Event 추가
   * OnFocusReceived와, OnFocusLost 델리게이트를 추가 (.h에)
   * cpp에서 construct()에서 값 할당
    ```cpp
    OnFocusReceivedDelegate = InArgs._OnFocusReceivedDelegate;
    OnFocusLostDelegate = InArgs._OnFocusLostDelegate;
    ```
    *  SWidget Interface의 OnFocusReceived, OnFocusLost 오버라이딩 후에 그곳에서 이벤트 Execute 하기

  ```cpp
virtual FReply OnFocusReceived(const FGeometry& MyGeometry, const FFocusEvent& InFocusEvent) override;
virtual void OnFocusLost(const FFocusEvent& InFocusEvent) override;
  ```


3. SMultiLineEditableTextBox 수정
   * SMultiLineEditableText를 포함하는 컨테이너 역할
   * SMultiLineEditableText의 포커스 이벤트를 받아 상위로 전달
   * UMultiLineEditableTextBox와 직접 연결되는 Slate 레벨의 위젯

4. UMultiLineEditableTextBox 수정
    * UMG 레벨의 위젯으로 BP에서 사용 가능
    * SMultiLineEditableTextBox의 포커스 이벤트를 받아 BP 이벤트로 변환


### 이러한 방식의 구현이 나쁘지 않은 이유?

1. 캡슐화와 책임 분리: 각 레벨의 위젯이 자신의 역할에 맞는 기능만 담당합니다. SMultiLineEditableText는 실제 편집 기능을, SMultiLineEditableTextBox는 컨테이너 역할을, UMultiLineEditableTextBox는 UMG 연동을 담당

2. 유연성: 각 레벨에서 포커스 이벤트를 처리할 수 있어, 필요에 따라 다양한 방식으로 대응할 수 있음
   
3. 일관성: 언리얼 엔진의 기존 위젯 구조와 일관성을 유지 가능. 다른 위젯들도 비슷한 구조로 이벤트를 처리.

4. 확장성: 나중에 추가적인 기능이나 이벤트가 필요할 때 각 레벨에서 쉽게 확장할 수 있음.

5. 블루프린트 지원: UMultiLineEditableTextBox에서 UPROPERTY와 UFUNCTION을 사용하여 블루프린트에서도 이 이벤트를 쉽게 사용할 수 있음.
