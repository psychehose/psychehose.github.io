

Qt 프레임워크와 OpenGL을 사용하여 기본 그라운드를 렌더링하는 코드에 대한 설명이다.

## 1. 그래픽 파이프라인의 기본 개념

OpenGL 그래픽 파이프라인은 3D 객체를 2D 화면에 표시하기 위한 일련의 단계를 말함

1. **정점 데이터 준비**: 3D 공간에서의 점(정점)들을 정의
2. **정점 쉐이더(Vertex Shader)**: 각 정점의 위치 변환 처리
3. **프리미티브 조립(Primitive Assembly)**: 정점들을 삼각형 등의 기본 도형으로 조립
4. **래스터화(Rasterization)**: 3D 기본 도형을 2D 픽셀로 변환
5. **프래그먼트 쉐이더(Fragment Shader)**: 각 픽셀의 최종 색상 계산
6. **프레임 버퍼에 출력**: 계산된 픽셀을 화면에 표시

## 2. 코드 분석

### createGeometry() 함수

바닥(그라운드)을 표현하기 위한 데이터를 준비한다.

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
```

1. **정점 데이터 정의**
   - 4개의 정점으로 이루어진 평면(사각형)을 생성한다.
   - 각 정점은 위치(x,y,z)와 색상(r,g,b) 정보를 가진다.

```cpp
    // 인덱스 데이터 (삼각형 정의)
    const unsigned int indices[] = {
        0, 1, 2,  // 첫 번째 삼각형
        0, 3, 2   // 두 번째 삼각형 - 면 컬링 고려
    };
```

2. **인덱스 데이터 정의**
   - 사각형을 2개의 삼각형으로 나누어 표현한다.
   - 첫 번째 삼각형: 정점 0, 1, 2를 연결
   - 두 번째 삼각형: 정점 0, 3, 2을 연결

```cpp
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
```

3. **OpenGL 버퍼 생성**
   - **VAO(Vertex Array Object)**: 정점 데이터의 구성 방식을 저장하는 객체
   - **VBO(Vertex Buffer Object)**: 실제 정점 데이터를 GPU 메모리에 저장
   - **EBO(Element Buffer Object)**: 인덱스 데이터를 GPU 메모리에 저장

```cpp
    // 위치 속성 설정 (attribute 0)
    m_program->enableAttributeArray(0);  // aPos 속성 활성화
    m_program->setAttributeBuffer(0, GL_FLOAT, 0, 3, 6 * sizeof(float));
    
    // 색상 속성 설정 (attribute 1)
    m_program->enableAttributeArray(1);  // aColor 속성 활성화
    m_program->setAttributeBuffer(1, GL_FLOAT, 3 * sizeof(float), 3, 6 * sizeof(float));
```

4. **속성 포인터 설정**
   - 쉐이더 프로그램에 정점 데이터의 구조를 알려줍니다
   - 첫 번째 속성(0): 위치 데이터 (x,y,z - 3개 float)
   - 두 번째 속성(1): 색상 데이터 (r,g,b - 3개 float)
   - 매개변수 설명:
     - 첫 번째: 속성 인덱스
     - 두 번째: 데이터 타입 (GL_FLOAT)
     - 세 번째: 데이터 시작 오프셋 (바이트 단위)
     - 네 번째: 구성요소 개수 (위치: 3, 색상: 3)
     - 다섯 번째: 다음 정점까지의 간격 (stride) (6개 float)

### drawGround() 함수

이 함수는 실제로 그라운드를 화면에 그려주는 역할을 합니다:

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

1. **VAO 바인딩**: 이전에 설정한 정점 데이터 구성을 활성
2. **모델 행렬 설정**: 객체의 위치, 회전, 크기를 정의
3. **그리기 명령 실행**: 인덱스를 이용해 삼각형을 그림
   - GL_TRIANGLES: 삼각형 모드로 그리기
   - m_groundIndices: 인덱스 개수 (6개)
   - GL_UNSIGNED_INT: 인덱스 데이터 타입
   - nullptr: 인덱스 데이터가 이미 바인딩된 EBO에 있음

### paintGL() 함수

이 함수는 Qt의 QOpenGLWidget 클래스의 가상 함수로, 화면을 그리는 메인 렌더링 함수입니다:

```cpp
void GLWidget::paintGL()
{
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
    // 쉐이더 프로그램 활성화
    m_program->bind();
    
    // 카메라 뷰 설정
    QMatrix4x4 view;
    view.lookAt(
        QVector3D(0.0f, 5.0f, 5.0f),  // 카메라 위치
        QVector3D(0.0f, 0.0f, 0.0f),  // 바라보는 지점
        QVector3D(0.0f, 1.0f, 0.0f)   // 상향 벡터
    );
```

1. **화면 초기화**: 색상 버퍼와 깊이 버퍼를 지움
2. **쉐이더 프로그램 활성화**: 그리기에 사용할 쉐이더를 활성화
3. **카메라 뷰 설정**

```cpp
    // 투영 행렬 설정
    QMatrix4x4 projection;
    float aspectRatio = width() / static_cast<float>(height());
    projection.perspective(45.0f, aspectRatio, 0.1f, 100.0f);
    
    // 쉐이더에 행렬 전달
    m_program->setUniformValue("view", view);
    m_program->setUniformValue("projection", projection);
    
    // 그라운드 그리기
    drawGround();
    
    // 쉐이더 프로그램 비활성화
    m_program->release();
}
```

4. **투영 행렬 설정**: 3D 장면을 2D 화면에 투영하는 방법을 정의
   - 45도 시야각(FOV)
   - 화면 비율(aspect ratio)에 맞게 조정
   - 근거리 절단면(near plane): 0.1
   - 원거리 절단면(far plane): 100.0

4. **행렬 전달**: 뷰 행렬과 투영 행렬을 쉐이더에 전달
5. **그라운드 그리기**: 앞서 정의한 drawGround() 함수 호출
6. **쉐이더 해제**: 사용 완료 후 쉐이더를 비활성화

## 3. 변환 행렬

코드에서 사용된 세 가지 주요 변환 행렬

1. **모델 행렬(Model Matrix)**
   - 객체의 로컬 공간에서 월드 공간으로 변환
   - 객체의 위치, 회전, 크기를 정의
   - 코드에서는 기본 단위 행렬 사용(원점에 위치)

2. **뷰 행렬(View Matrix)**
   - 월드 공간에서 카메라 공간으로 변환
   - 카메라의 위치와 방향을 정의
   - lookAt 함수로 생성

3. **투영 행렬(Projection Matrix)**
   - 카메라 공간에서 클립 공간으로 변환
   - 원근감 적용 (멀리 있는 물체는 작게 보임)
   - perspective 함수로 생성