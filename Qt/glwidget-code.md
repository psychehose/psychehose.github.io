# Qt OpenGL GLWidget 코드 분석

이 문서는 Qt 프레임워크를 이용한 OpenGL 위젯(GLWidget) 구현 코드에 대한 상세 분석입니다.

## 1. 헤더 파일 및 기본 설정

```cpp
#include "glwidget.h"
#include <QMouseEvent>
#include <QKeyEvent>
#include <QWheelEvent>
#include <QtMath>
#include <QOpenGLFunctions>
#include <QMatrix4x4>
#include <QVector3D>
#include <QDebug>
#include <QVector>
```

이 부분은 필요한 Qt 헤더 파일들을 포함합니다:
- 이벤트 처리(마우스, 키보드, 휠)
- OpenGL 함수 인터페이스
- 3D 수학 라이브러리(행렬, 벡터)
- 디버깅 기능

## 2. 쉐이더 소스 코드

```cpp
static const char *vertexShaderSource =
        "attribute vec3 aPos;\n"
        "attribute vec3 aColor;\n"
        "uniform mat4 model;\n"
        "uniform mat4 view;\n"
        "uniform mat4 projection;\n"
        "varying vec3 ourColor;\n"

        "void main() {\n"
        "    gl_Position = projection * view * model * vec4(aPos, 1.0);\n"
        "    ourColor = aColor;\n"
        "}\n";


static const char *fragmentShaderSource =
        "varying vec3 ourColor;\n"
        "void main() {\n"
        "    gl_FragColor = vec4(ourColor, 1.0);\n"
        "}\n";
```

### 정점 쉐이더(Vertex Shader)
- `attribute vec3 aPos`: 정점 위치 데이터
- `attribute vec3 aColor`: 정점 색상 데이터
- `uniform mat4 model/view/projection`: 변환 행렬들
- `varying vec3 ourColor`: 프래그먼트 쉐이더로 전달될 색상 데이터
- 변환 행렬 순서: projection * view * model * position (행렬 곱셈은 오른쪽에서 왼쪽으로)

### 프래그먼트 쉐이더(Fragment Shader)
- `varying vec3 ourColor`: 정점 쉐이더에서 전달받은 색상 값
- `gl_FragColor`: 최종 픽셀 색상 (RGB + 알파)

## 3. GLWidget 클래스 구현

### 생성자 및 소멸자

```cpp
GLWidget::GLWidget(QWidget *parent)
    : QOpenGLWidget(parent),
    m_program(nullptr),
    m_groundVBO(QOpenGLBuffer::VertexBuffer),
    m_groundEBO(QOpenGLBuffer::IndexBuffer),
    m_groundIndices(0)
{
    // 포커스 정책 설정 (키보드 입력 받기 위함)
    setFocusPolicy(Qt::StrongFocus);
}

GLWidget::~GLWidget()
{
    // 현재 스레드에서 OpenGL 컨텍스트를 활성화합니다.
    makeCurrent();

    m_groundVBO.destroy();
    m_groundEBO.destroy();
    m_groundVAO.destroy();

    // doneCurrent();  // QOpenGLWidget에서 자동 호출되므로 제거됨
}
```

- **생성자**: OpenGL 버퍼 객체 초기화, 키보드 포커스 설정
- **소멸자**: OpenGL 리소스 정리
  - `makeCurrent()`: GL 컨텍스트 활성화 (리소스 해제를 위해 필수)
  - 버퍼 객체들 파괴

### 초기화 함수

```cpp
void GLWidget::initializeGL()
{
    // opengl 함수 사용 준비 함수
    initializeOpenGLFunctions();

    // 배경색 설정 (하늘색)
    glClearColor(0.5f, 0.7f, 1.0f, 1.0f);

    // 깊이 테스트 활성화 - 3D 공간에서 물체의 앞뒤 관계 처리
    glEnable(GL_DEPTH_TEST);

    // 면 컬링 활성화 - 보이지 않는 뒷면을 렌더링하지 않아 성능 향상
    glEnable(GL_CULL_FACE);

    m_program = new QOpenGLShaderProgram(this);
    m_program->addShaderFromSourceCode(QOpenGLShader::Vertex, vertexShaderSource);
    m_program->addShaderFromSourceCode(QOpenGLShader::Fragment, fragmentShaderSource);
    m_program->link();

    // 그라운드 지오메트리 생성
    createGeometry();
}
```

이 함수는 OpenGL 초기화를 담당합니다:
1. OpenGL 함수 초기화
2. 렌더링 설정:
   - 배경색: 하늘색(0.5, 0.7, 1.0)
   - 깊이 테스트 켜기: 3D 물체의 앞뒤 관계 처리
   - 면 컬링 켜기: 보이지 않는 뒷면을 그리지 않아 성능 향상
3. 쉐이더 프로그램 생성:
   - 정점 및 프래그먼트 쉐이더 추가
   - 프로그램 링크
4. 기하 데이터(그라운드) 생성

### 크기 조정 함수

```cpp
void GLWidget::resizeGL(int w, int h)
{
    glViewport(0,0,w,h);
}
```

- 위젯 크기가 변경될 때 호출됨
- 뷰포트를 새 크기로 설정 (화면 표시 영역)

### 지오메트리 생성 함수

```cpp
void GLWidget::createGeometry()
{
    // 그라운드 크기 설정
    constexpr float size = 10.0f;
    
    // 기본 색상 설정 (그린)
    const QVector3D green(0.0f, 0.5f, 0.1f);
    
    // 정점 데이터 (위치와 색상 정보)
    const float vertices[] = {
        // 위치(x, y, z)        // 색상(r, g, b)
        -size, 0.0f, -size,   green.x(), green.y(), green.z(),  // 좌상단
         size, 0.0f, -size,   green.x(), green.y(), green.z(),  // 우상단 
         size, 0.0f,  size,   green.x(), green.y(), green.z(),  // 우하단
        -size, 0.0f,  size,   green.x(), green.y(), green.z()   // 좌하단
    };
    
    // 인덱스 데이터 (삼각형 정의)
    const unsigned int indices[] = {
        0, 1, 2,  // 첫 번째 삼각형
        0, 2, 3   // 두 번째 삼각형
    };
    
    // 인덱스 개수 저장
    m_groundIndices = 6;  // 2개의 삼각형 = 6개의 인덱스
    
    // VAO 생성 및 바인딩
    m_groundVAO.create();
    m_groundVAO.bind();
    
    // VBO(정점 버퍼) 생성 및 데이터 할당
    m_groundVBO.create();
    m_groundVBO.bind();
    m_groundVBO.allocate(vertices, sizeof(vertices));
    
    // EBO(인덱스 버퍼) 생성 및 데이터 할당
    m_groundEBO.create();
    m_groundEBO.bind();
    m_groundEBO.allocate(indices, sizeof(indices));
    
    // 위치 속성 설정 (attribute 0)
    m_program->enableAttributeArray(0);  // aPos 속성 활성화
    m_program->setAttributeBuffer(0, GL_FLOAT, 0, 3, 6 * sizeof(float));
    
    // 색상 속성 설정 (attribute 1)
    m_program->enableAttributeArray(1);  // aColor 속성 활성화
    m_program->setAttributeBuffer(1, GL_FLOAT, 3 * sizeof(float), 3, 6 * sizeof(float));
    
    // 버퍼 및 VAO 해제
    m_groundEBO.release();
    m_groundVBO.release();
    m_groundVAO.release();
}
```

이 함수는 그라운드(바닥) 데이터를 설정합니다:

1. **정점 및 인덱스 데이터 정의**:
   - 4개의 정점으로 사각형 평면 정의
   - 모든 정점이 같은 녹색 색상을 가짐
   - 2개의 삼각형(6개 인덱스)으로 사각형 구성

2. **OpenGL 버퍼 객체 생성 및 데이터 할당**:
   - VAO(Vertex Array Object): 정점 데이터 레이아웃 정의 
   - VBO(Vertex Buffer Object): 정점 데이터 저장
   - EBO(Element Buffer Object): 인덱스 데이터 저장

3. **속성 포인터 설정**:
   - 속성 0(위치): 각 정점의 처음 3개 float (x,y,z)
   - 속성 1(색상): 각 정점의 다음 3개 float (r,g,b)
   - 각 정점 데이터의 전체 크기는 6개 float (위치 3개 + 색상 3개)

4. **버퍼 해제**: 버퍼 설정 완료 후 바인딩 해제

### 그라운드 그리기 함수

```cpp
void GLWidget::drawGround()
{
    // 그라운드 VAO 바인딩
    m_groundVAO.bind();
    
    // 모델 행렬 설정 (단위 행렬 - 원점에 위치)
    QMatrix4x4 model;
    m_program->setUniformValue("model", model);
    
    // 그라운드 그리기 (EBO 사용)
    glDrawElements(GL_TRIANGLES, m_groundIndices, GL_UNSIGNED_INT, nullptr);
    
    // VAO 해제
    m_groundVAO.release();
}
```

이 함수는 그라운드를 렌더링합니다:
1. VAO 바인딩: 정점 데이터 레이아웃 활성화
2. 모델 행렬 설정: 기본 단위 행렬(원점에 위치)
3. 그리기 명령 실행: 삼각형 모드로 인덱스를 사용하여 그리기
4. VAO 해제

### 페인트 함수

```cpp
void GLWidget::paintGL()
{
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);

    // 쉐이더 프로그램 활성화
    m_program->bind();
    
    // ��메라 뷰 설정
    QMatrix4x4 view;
    view.lookAt(
        QVector3D(0.0f, 5.0f, 5.0f),  // 카메라 위치
        QVector3D(0.0f, 0.0f, 0.0f),  // 바라보는 지점
        QVector3D(0.0f, 1.0f, 0.0f)   // 상향 벡터
    );
    
    // 투영 행렬 설정
    QMatrix4x4 projection;
    float aspectRatio = width() / static_cast<float>(height());
    projection.perspective(45.0f, aspectRatio, 0.1f, 100.0f);
    
    // 쉐이더에 행렬 전달
    m_program->setUniformValue("view", view);
    m_program->setUniformValue("projection", projection);
    
    // 그라운드 그리기 (주석 처리됨)
    // drawGround();
    
    // 쉐이더 프로그램 비활성화
    m_program->release();
}
```

이 함수는 OpenGL 렌더링 루프의 핵심 부분입니다:

1. **화면 초기화**: 색상 버퍼와 깊이 버퍼 지우기
2. **쉐이더 프로그램 활성화**
3. **카메라 설정**:
   - `lookAt` 함수로 뷰 행렬 생성
   - 카메라 위치: (0, 5, 5) - 바닥으로부터 5 높이, 뒤로 5 거리
   - 바라보는 지점: (0, 0, 0) - 원점(그라운드 중앙)
   - 상향 벡터: (0, 1, 0) - Y축 방향이 위쪽

4. **투영 설정**:
   - 원근 투영(perspective) 사용
   - 시야각(FOV): 45도
   - 화면 비율: 위젯 너비/높이
   - 근평면(near): 0.1
   - 원평면(far): 100.0

5. **행렬 전달**: 쉐이더에 view와 projection 행렬 전달
6. **그리기 호출**: 
   - 현재 주석 처리되어 있음 (`// drawGround();`)
   - 코드 주석에 "여기서 사망", "여기 고치자~~"라는 디버깅 메모가 있음
7. **쉐이더 해제**

## 4. 주목할 점

1. **OpenGL 리소스 관리**:
   - 적절한 시점에 리소스 생성 및 해제
   - RAII(Resource Acquisition Is Initialization) 패턴 사용

2. **쉐이더 프로그램**:
   - 간단한 정점/프래그먼트 쉐이더로 3D 변환 및 색상 처리
   - 모델-뷰-투영(MVP) 변환 행렬 사용

3. **그라운드 렌더링 문제**:
   - `drawGround()` 함수 호출이 주석 처리되어 있음
   - "여기서 사망", "여기 고치자~~" 주석으로 문제 지점 표시
   - 이 부분을 주석 해제하면 그라운드가 화면에 표시될 것으로 예상됨

4. **Qt와 OpenGL 통합**:
   - Qt의 QOpenGLWidget, QOpenGLFunctions, QOpenGLBuffer 등을 사용하여 OpenGL 작업을 간소화
   - Qt의 QMatrix4x4, QVector3D 등 수학 클래스를 이용한 3D 변환 처리

## 5. 골프 시뮬레이션 관련 참고사항

이 코드는 골프 시뮬레이션 앱을 위한 기초 구조로, 그라운드(골프장 바닥)만 구현되어 있습니다. 골프공의 궤적을 시뮬레이션하려면:

1. 골프공 3D 모델 추가 필요
2. 골프공 물리 시뮬레이션 연동 필요
3. 궤적 시각화 기능 추가 필요
4. 현재 `drawGround()` 함수가 주석 처리되어 있어 어떤 시각적 출력도 없음 (디버깅 중인 것으로 보임)

이 코드를 확장하여 골프공의 물리 시뮬레이션 라이브러리와 연동하면, 원하는 궤적 시뮬레이션 테스트 앱을 완성할 수 있을 것입니다.
