CMake는 빌드파일을 생성하는 프로그램이다. CMake를 통해 빌드 파일을 생성하고 빌드 프로그램 (Make, Visual Studio, Xcode etc..)을 이용해 프로젝트를 빌드할 수 있다.


### 필수 문법

* `add_executable (<실행 파일 이름> <소스1> <소스2> ... <소스들>)`
* `target_compile_options(<실행 파일 이름> PUBLIC <컴파일 옵션1> <컴파일 옵션2> ...)`
	* Ex: `target_compile_options(program PUBLIC -Wall -Werror)
	* PUBLIC: 
	* PRIVATE:
* `target_include_directories(program PUBLIC ${CMAKE_SOURCE_DIR}/includes)`
	* 헤더 파일 탐색 경로 추가
* `add_library (<라이브러리 이름> [STATIC | SHARED | MODULE ] <소스 1> <소스 2> ...)`
	* STATIC: 정적
	* SHARED: 동적
	* MODULE: 동적링크 X, `dlopen` 같은 함수로 런타임에 불러올 수 있는 라이브러리


CMake 명령은 타겟을 정의하는 것과 타겟의 속성을 지정하는 명령어로 구성된다.
#### 빌드 파일 생성하는 방법

```zsh
$ mkdir build && cd build
$ cmake .. # root CMakeList.txt 경로를 지정해야함
```









#### Ninja 설치

```zsh
$ brew install ninja
```

#### Ninja

```
$ cmake .. -G Ninja
$ cmake --build .
```
