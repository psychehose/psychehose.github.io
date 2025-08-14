
### Q. Qt의 Signal - Slot이란?

Qt의 Signal-Slot은 객체 간 통신을 위한 핵심 메커니즘이다. 시그널은 특정 이벤트가 발생했을 때 방출되는 신호. Slot은 시그널을 받아서 실행되는 함수 일반적인 C++ 멤버 함수와 동일하다. Signal이 발생하면 자동으로 호출됨

1. Signal 방출: 특정 이벤트가 발생하면 객체에서 signal을 방출
2. Connection:  `connect()` 함수로 signal과 slot을 함수로 signal과 slot을 미리 연결
3. Slot 실행: Signal이 방출되면 연결된 모든 slot이 자동 실행

### Q. Q_OBJECT 매크로의 역할

* Q_OBJECT가 없으면 일반적인 C++ 클래스
* Q_OBJECT가 있으면 Qt의 Meta-Object System이 활성화됨

Q_OBJECT 매크로를 선언해줘야 Signal/Slot 메커니즘 사용 가능


### Q. QOpenGL 작동 방식

Qt는 QOpenGLWidget을 통해 OpenGL 컨텍스트를 관리하고 렌더링을 수행한다.

  1. QOpenGLWidget을 상속한다.  QOpenGLWidget은 OpenGL 렌더링을 위한 위젯 클래스다. 일종의 Scene이라고 생각하면 됨.

  2. QOpenGLFunctions를 상속한다. 이건 OpenGL 함수 호출을 위한 크로스 플랫폼 래퍼다.

즉 QOpenGLWidget을, QOpenGLFunctions 2개를 상속하는 건 기본이다. 그리고 아래의 함수를 override해서 구현하면 된다.

```cpp
void initializeGL() override;
void resizeGL(int w, int h) override;
void paintGL() override;
```

세 가지 핵심 가상함수의 역할은 다음과 같다.

1. initializeGL(): OpenGL 컨텍스트 초기화
	* OpenGL 함수 초기화 (initializeOpenGLFunctions())
	* 렌더링 상태 설정 (배경색, 깊이 테스트, 면 컬링)
	* 셰이더 로드 및 셰이더 프로그램 설정 및 
	* VAO/VBO/EBO를 사용한 지오메트리 생성

2. resizeGL(): 뷰포트 크기 조정

3. paintGL(): 실제 렌더링 수행
	* 프레임 버퍼를 초기화
	* 셰이더 프로그램 바인딩
	* 카메라, 프로젝션 설정
	* **렌더링** (drawGround ... etc)
	* 셰이더 프로그램 릴리즈


```
정점 데이터 → VBO (GPU 메모리 저장)
인덱스 데이터 → EBO (GPU 메모리 저장) - 어떤 정점들을 연결해서 삼각형을 만들자 정의
속성 설정 → VAO (상태 관리) - 정점 속성 설정을 저장하는 상태 관리자
```
