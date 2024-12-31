
### SFML (Simple and Fast Multimedia Library)

크로스 플랫폼 Golf Simulator를 만들면서 빠른 그래픽 렌더링을 위해 사용해봤다.
SFML은 C++로 작성된 멀티미디어 라이브러리다. 게임 개발이나 그래픽 애플리케이션을 만들 때 사용되는 크로스 플랫폼 프레임워크다. 주로 2D 엔진일 때 사용하는듯?

SFML은 다양한 모듈을 지원하는데 내가 사용한 모듈은 Graphics, System, Window다.

* Window: 어플리케이션의 창과 입력을 관리하는 핵심 모듈
	* sf::RenderWindow - 윈도우
	* sf::VideoMode - 해상도
	* sf::Event - 입력 이벤트 처리
	
* System: 유틸 모듈 (Thread, Clock 등등 지원함)
	* sf::Vector - 벡터 연산

* Graphics: 렌더링
	* sf::CircleShape: 골프공
	* sf::RectangleShape: 그라운드
	* sf::View - 카메라 

### 설치와 적용

cmake를 이용하고 있어서 CMakeLists.txt에서 라이브러리를 링크하는 형태로 적용했다.
SFML 예제를 보면 CMake의 FetchContent 모듈을 통해서 SFML을 다운로드하고 의존성을 관리한다.

```cmake
FetchContent_Declare(SFML
	GIT_REPOSITORY https://github.com/SFML/SFML.git
	GIT_TAG 3.0.0
	GIT_SHALLOW ON
	EXCLUDE_FROM_ALL
	SYSTEM
)
FetchContent_MakeAvailable(SFML)

# ... 

target_link_libraries(${PROJECT_NAME}
	PRIVATE
	SFML::Graphics
	SFML::Window
	SFML::System
)

```
















