
C++, SFML을 이용해서 간단한 2D 골프 시뮬레이터를 만든 과정을 공유할 것입니다.

이 프로젝트는 C++ 코드로 골프공 움직임을 시뮬레이션하고, SFML 라이브러리를 이용해서 렌더링 하는 것이 목표입니다.


### 구조

```
.
└── src
    ├── core
    │   ├── GolfBall.cpp
    │   ├── GolfBall.h
    │   ├── Simulator.cpp
    │   └── Simulator.h
    └── main.cpp
```

### 물리 모델

```cpp
class GolfBall {
private:
    const double gravity = 9.81;      // 중력가속도
    const double airDensity = 1.225;  // 공기밀도
    const double mass = 0.0459;       // 골프공 질량
    const double radius = 0.0213;     // 골프공 반지름
    const double dragCoefficient; // 항력계수: 0.47
	const double liftCoefficient; // 양력계수: 0.1
	
	double x, y; // 골프공 위치
	double vx, vy; // x축, y축 방향의 속도
	double spin; // 회전속도 (rad/s)

public:
	GolfBall(double initialVelocity, double launchAngle, double initialSpin);
	void update(double dt);
	bool isFlying() const;
	std::pair<double, double> getPosition() const;
}
```


골프공에 작용하는 주요 힘은 간단하게 아래 3가지만 고려했습니다.
1. 중력
2. 항력
3. 양력

```cpp
void GolfBall::update(double dt) {
	double velocity = sqrt(vx * vx + vy * vy); // 스칼라
	double area = M_PI * radius * radius; // 골프공 단면적 
	
	// 공기저항력 계산
	double dragForce =
		0.5 * airDensity * dragCoefficient * area * velocity * velocity;

	// 양력 계산
	double liftForce =
		0.5 * airDensity * liftCoefficient * area * velocity * velocity;

	// F = ma -> a = F/m을 이용.
	double ax = -(dragForce * vx / velocity) / mass;
	double ay =
		-gravity - (dragForce * vy / velocity) / mass + (liftForce/mass);
		
	x += vx * dt;
	y += vy * dt;
	vx += ax * dt;
	vy += ay * dt;

}
```

update 함수에서 **위치**(x,y)를 업데이트 하고 **속도** (vx, vy)를 업데이트 하기 위해
항력(dragForce), 양력(liftForce)를 계산한 다음 이를 이용해 가속도를 구했습니다.

### 렌더링

#### SFML 설치
[[SFML]]

#### 핵심 구성 요소

```cpp
class Simulator {
private:
    sf::RenderWindow window;     // SFML 윈도우
    GolfBall golfBall;          // 골프공 객체
    sf::CircleShape ballShape;   // 골프공 그래픽
    sf::View camera;            // 카메라 뷰
    const float SCALE = 10.0f;   // 물리적 거리를 픽셀로 변환하는 비율 (1m = 10px)
}
```

#### 화면 설정

```cpp
Simulator::Simulator(double initialVelocity, double launchAngle, double initialSpin)
    : window(sf::VideoMode(sf::Vector2u(800, 600)), "Golf Simulator") {
    camera.setSize(sf::Vector2f(1600.f, 1200.f));
    // ...
}
```

window 사이즈를 800, 600으로 설정했고 camera size를 2배로 설정했다. 따라서 2배 넓은 시야를 가진다. (축소 모드)
#### 메인 게임 루프

```cpp
void Simulator::run() {
    while (window.isOpen()) {
        handleEvents();    // 이벤트 처리
        update();          // 물리 시뮬레이션 업데이트
        render();          // 화면 렌더링
    }
}
```

#### render() 함수

```cpp
void Simulator::render() {
	window.clear(sf::Color(50, 50, 50));
	updateCamera();
	window.draw(groundShape);

	auto position = golfBall.getPosition();
	sf::Vector2f ballPosition(position.first * SCALE,
		500.f - position.second * SCALE);

	ballShape.setPosition(ballPosition);
	window.draw(ballShape);  
	window.display();

}
```

핵심이 되는 함수다. 이 함수는 매 프레임마다 호출된다. 그래서 clear가 반드시 필요하다. clear를 하지 않으면 이전 프레임의 그림이 남아있어 새로운 프레임이 계속 중첩되어서 그려질 것이다.

 `updateCamera()`를 호출해서 카메라 위치를 업데이트하고 window에 groundShape를 그린다. (ground는 위치가 고정이라 그려주기만 하면 됨.)

그 다음으로는 ballShape를 그려야하므로 ball의 위치를 가져온다. 현재 ball의 위치를 `ballPosition` 을 생성할 때 y축의 값이 `500.f - position.second * SCALE`  인 이유는 SFML의 좌표계와 ball의 좌표계가 다르기 때문이다.

1. **물리 시뮬레이션 좌표계**
    - Y축이 위로 갈수록 양수 (+)
    - 지면이 0
    - 공이 위로 올라갈수록 y값이 증가
      
2. **SFML 화면 좌표계**
    - Y축이 아래로 갈수록 양수 (+)
    - 화면 최상단이 0
    - 아래로 갈수록 y값이 증가

`500.0f`는 지면의 y좌표고 `- position.second * SCALE` 는 물리 좌표계의 y값을 화면 좌표계로 변환한 값이다.

```
물리 좌표계    화면 좌표계
    ↑ +y         0 ─────
    │            │    ↓ +y
    │            │
  0 ─────      500 ─────
```

마지막으로 ball의 위치를 설정하고 window에 그린 다음에 display 하면 끝이다.

#### 골프공을 자연스럽게 따라가는 카메라 구현

```cpp
void Simulator::updateCamera() {
    auto ballPos = golfBall.getPosition();
    sf::Vector2f targetCenter(ballPos.first * SCALE, 300.f);
    
    // 부드러운 카메라 이동
    cameraCenter.x = cameraCenter.x + (targetCenter.x - cameraCenter.x) * CAMERA_SPEED;
}
```

####  repos

https://github.com/psychehose/golf_simulator