TickerProviderStateMixin는 애니메이션을 만들기 위해 필요한 기능

Mixin이란?
- Dart 언어의 특징 중 하나로, 클래스에 특정 기능을 섞어 넣는(mix-in) 방법
- 상속과 비슷하지만 여러 Mixin을 하나의 클래스에 적용할 수 있어 유연

TickerProvider의 역할은?
* AnimationController은 Tick을 받아서 애니메이션의 각 프레임을 업데이트함.
* TickerProvider는 Ticker를 제공하는 역할

TickerProviderStateMixin는 무엇?
* StatefulWidget의 State 클래스에서 AnimationController를 쉽게 사용할 수 있도록 제공되는 Mixin
* Mixin을 with 키워드로 추가하면 State가 Ticker를 제공할 수 있음
* 위젯이 화면에서 사라질 때 Ticker를 정리해서 메모리 누수 방지



SingleTickerProviderStateMixin

* TickerProviderStateMixin 경량화된 버전
* State에서 단 하나의 Animation Controller를  사용할 수 있음.
* 가볍고 리소스 소모 적음