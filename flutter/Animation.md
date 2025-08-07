
```dart

late AnimationController _animationController;
late Animation<double> _fadeAnimation;

_animationController = AnimationController(
      duration: const Duration(milliseconds: 1000),
      vsync: this,
    );

    _fadeAnimation = Tween<double>(begin: 0.0, end: 1.0).animate(
      CurvedAnimation(parent: _animationController, curve: Curves.easeInOut),
    );

    _animationController.forward();
```


AnimationController는 애니메이션의 전반적인 제어를 담당함.

AnimationController
* duration
* 시작 (forward), 중지 (stop), 반복(repeat), 역재생(reverse) 등
* vsync: 
	* 애니메이션이 화면의 새로고침 주기에 맞춰 실행되도록 동기화하는 역할
	* 없는 경우 버벅임, 부자연스러울 수 있음
	* this를 사용하기 위해서 with TickerProviderStateMixin가 필요


Animation 객체
* 실제로 어떤 값이 변할지를 정의하는 추상화된 객체

CurvedAnimation
* 애니메이션에 속도감을 부여함
* parent를 지정해줘야함. (어떤 제어를 받을 지)
* begin:0.0, end: 1.0 이라는 건 처음 상태를 0으로 설정하고 끝을 1.0으로 설정하겠다는 것
* Curve 종류 (In은 가속, Out은 감속으로 생각하면 됨)
	* easeInOut :애니메이션이 천천히 시작해서 중간에 가속 끝날 때 다시 천천히 멈추는 것
	* easeIn: 점점 빠르게
	* easeOut: 점점 느리게
	* bound: 통통 튀는 효과
	* etc..

