# OpenGL 골프공(구) 지오메트리 생성 코드 분석

이 문서는 OpenGL에서 구(sphere) 지오메트리를 생성하는 코드를 분석합니다. 특히 3D 공간에서 골프공을 표현하기 위한 구를 생성하는 과정을 자세히 살펴봅니다.

## 1. 전체 기능 개요

`createBall()` 함수는 3D 공간에서 골프공을 표현하기 위한 구(sphere) 지오메트리를 생성합니다. 코드는 다음과 같은 단계로 진행됩니다:

1. 구의 파라미터 설정 (반지름, 분할 수)
2. 정점(vertex) 데이터 생성
3. 인덱스(index) 데이터 생성
4. OpenGL 버퍼 객체 설정 (VAO, VBO, EBO)

## 2. 구 모델링의 수학적 이론

### 2.1 구의 수학적 표현

3D 공간에서 구는 다음과 같은 매개변수 방정식으로 표현됩니다:
- x = r × cos(φ) × cos(θ)
- y = r × cos(φ) × sin(θ)
- z = r × sin(φ)

여기서:
- r은 반지름
- φ(phi)는 수직각(latitude): -π/2(하단) ~ π/2(상단)
- θ(theta)는 수평각(longitude): 0 ~ 2π

코드에서는 이를 다음처럼 구현했습니다:
```cpp
float stackAngle = PI / 2 - i * stackStep; // φ 계산
float xy = radius * cosf(stackAngle);      // r × cos(φ) 계산
float z = radius * sinf(stackAngle);       // r × sin(φ) 계산

float sectorAngle = j * sectorStep;        // θ 계산
float x = xy * cosf(sectorAngle);          // r × cos(φ) × cos(θ)
float y = xy * sinf(sectorAngle);          // r × cos(φ) × sin(θ)
```

### 2.2 구의 분할 방법

구는 두 가지 방향으로 분할됩니다:
- stackCount: 위에서 아래로 분할 (위도 분할)
- sectorCount: 수평 방향 분할 (경도 분할)

이 분할은 지구본을 상상하면 이해하기 쉽습니다. stackCount는 적도에서 북극/남극까지 몇 단계로 나눌지를, sectorCount는 경도선을 몇 개 사용할지 결정합니다.

## 3. 코드 분석

### 3.1 구 파라미터 설정
```cpp
const float radius = 0.3f;      // 골프공 반지름
const int sectorCount = 36;     // 수평 분할 수 (경도)
const int stackCount = 18;      // 수직 분할 수 (위도)
const QVector3D ballColor(1.0f, 1.0f, 1.0f); // 흰색 설정
```

### 3.2 정점 데이터 생성
```cpp
const float PI = M_PI;
const float sectorStep = 2.0f * PI / sectorCount; // 각 섹터(경도) 사이의 각도
const float stackStep = PI / stackCount;          // 각 스택(위도) 사이의 각도

for (int i = 0; i <= stackCount; ++i) {
  float stackAngle = PI / 2 - i * stackStep; // 시작: π/2(상단), 끝: -π/2(하단)
  // ...

  for (int j = 0; j <= sectorCount; ++j) {
    // ...
  }
}
```

이 중첩 루프는 구의 모든 정점을 생성합니다:
- 외부 루프(i)는 위에서 아래로 각 스택(위도)을 순회
- 내부 루프(j)는 각 스택에서 수평 방향(경도)으로 순회

각 정점에 대해:
1. 위치(x, y, z) 계산
2. 색상(r, g, b) 설정 (모든 정점은 흰색)

### 3.3 인덱스 생성
```cpp
for (int i = 0; i < stackCount; ++i) {
  int k1 = i * (sectorCount + 1);     // 현재 스택의 시작 인덱스
  int k2 = k1 + sectorCount + 1;      // 다음 스택의 시작 인덱스
  
  for (int j = 0; j < sectorCount; ++j, ++k1, ++k2) {
    // 각 섹터에 2개의 삼각형 추가
    if (i != 0) { // 첫 번째 스택이 아닐 경우
      indices.append(k1);
      indices.append(k2);
      indices.append(k1 + 1);
    }
    
    if (i != (stackCount - 1)) { // 마지막 스택이 아닐 경우
      indices.append(k1 + 1);
      indices.append(k2);
      indices.append(k2 + 1);
    }
  }
}
```

인덱스 생성은 구를 삼각형 메쉬로 변환하는 중요한 과정입니다:

1. 각 스택과 섹터의 교차점마다 사각형 영역이 생성됨
2. 각 사각형은 두 개의 삼각형으로 분할됨
3. 특별 케이스 처리:
   - 첫 번째 스택(i=0): 상단 극점만 처리
   - 마지막 스택(i=stackCount-1): 하단 극점만 처리

각 삼각형은 세 개의 인덱스로 구성되며, 이 인덱스는 먼저 정의된 정점 배열을 참조합니다.

### 3.4 OpenGL 버퍼 설정

```cpp
// VAO(Vertex Array Object) 생성 및 바인딩
m_ballVAO.create();
m_ballVAO.bind();

// VBO(Vertex Buffer Object) 생성 및 데이터 할당
m_ballVBO.create();
m_ballVBO.bind();
m_ballVBO.allocate(vertices.constData(), vertices.size() * sizeof(float));

// EBO(Element Buffer Object) 생성 및 데이터 할당
m_ballEBO.create();
m_ballEBO.bind();
m_ballEBO.allocate(indices.constData(), indices.size() * sizeof(GLuint));
```

이 부분은 OpenGL의 핵심 개념인 버퍼 객체를 설정합니다:

1. **VAO(Vertex Array Object)**: 정점 속성 포인터의 상태를 저장하는 컨테이너
2. **VBO(Vertex Buffer Object)**: 정점 데이터(위치, 색상 등)를 저장하는 버퍼
3. **EBO(Element Buffer Object)**: 인덱스 데이터를 저장하는 버퍼

### 3.5 정점 속성 설정

```cpp
// 속성 설정 - 셰이더의 aPos 및 aColor 속성과 연결
int posAttr = m_program->attributeLocation("aPos");
int colorAttr = m_program->attributeLocation("aColor");

// 위치 속성 설정
m_program->enableAttributeArray(posAttr);
m_program->setAttributeBuffer(posAttr, GL_FLOAT, 0, 3, 6 * sizeof(float));

// 색상 속성 설정
m_program->enableAttributeArray(colorAttr);
m_program->setAttributeBuffer(colorAttr, GL_FLOAT, 3 * sizeof(float), 3, 6 * sizeof(float));
```

이 부분은 셰이더 프로그램에 전달할 정점 속성을 설정합니다:

1. **위치 속성(aPos)**: 
   - 시작 오프셋: 0 (정점 데이터의 시작)
   - 요소 개수: 3 (x, y, z)
   - 스트라이드: 6 * sizeof(float) (한 정점의 전체 크기)

2. **색상 속성(aColor)**:
   - 시작 오프셋: 3 * sizeof(float) (위치 다음)
   - 요소 개수: 3 (r, g, b)
   - 스트라이드: 6 * sizeof(float) (한 정점의 전체 크기)

## 4. 주요 개념 설명

### 4.1 정점 데이터 구조

각 정점은 다음과 같은 형식으로 저장됩니다:
```
[x, y, z, r, g, b]
```
- 처음 3개 값(x, y, z)은 정점의 위치
- 다음 3개 값(r, g, b)은 정점의 색상

### 4.2 인덱스 기반 렌더링의 이점

인덱스를 사용하는 이유:
1. **메모리 효율성**: 같은 정점을 여러 삼각형에서 재사용할 수 있어 메모리 사용량 감소
2. **성능 향상**: 중복된 정점 처리를 줄여 GPU 효율 증가

## 5. 인덱스 생성 시각화

![구(Sphere)의 인덱스 생성 시각화](sphere-indexing-screenshot.png)

### 5.1 인덱스 생성 알고리즘 분석

인덱스 생성 코드의 핵심 부분을 자세히 살펴보겠습니다:

```cpp
for (int i = 0; i < stackCount; ++i) {
  int k1 = i * (sectorCount + 1);     // 현재 스택의 시작 인덱스
  int k2 = k1 + sectorCount + 1;      // 다음 스택의 시작 인덱스
  
  for (int j = 0; j < sectorCount; ++j, ++k1, ++k2) {
    // 각 섹터에 2개의 삼각형 추가
    if (i != 0) { // 첫 번째 스택이 아닐 경우
      indices.append(k1);
      indices.append(k2);
      indices.append(k1 + 1);
    }
    
    if (i != (stackCount - 1)) { // 마지막 스택이 아닐 경우
      indices.append(k1 + 1);
      indices.append(k2);
      indices.append(k2 + 1);
    }
  }
}
```

### 5.2 인덱스 계산 방법

- `k1`: 현재 스택(i)의 시작 인덱스
- `k2`: 다음 스택(i+1)의 시작 인덱스

이 값들은 다음과 같이 계산됩니다:
```cpp
k1 = i * (sectorCount + 1);
k2 = k1 + sectorCount + 1;
```

예를 들어, sectorCount가 36인 경우:
- i=0일 때: k1=0, k2=37
- i=1일 때: k1=37, k2=74
- i=2일 때: k1=74, k2=111

### 5.3 사각형을 삼각형으로 분할

각 스택과 섹터 사이의 사각형 영역을 두 개의 삼각형으로 분할합니다:

1. **상단 삼각형** (i != 0일 때만):
   ```cpp
   indices.append(k1);     // 현재 스택, 현재 섹터
   indices.append(k2);     // 다음 스택, 현재 섹터
   indices.append(k1 + 1); // 현재 스택, 다음 섹터
   ```

2. **하단 삼각형** (i != stackCount-1일 때만):
   ```cpp
   indices.append(k1 + 1); // 현재 스택, 다음 섹터
   indices.append(k2);     // 다음 스택, 현재 섹터
   indices.append(k2 + 1); // 다음 스택, 다음 섹터
   ```

### 5.4 특별 케이스 처리

1. **첫 번째 스택 (i=0)**: 
   - 상단 극점에 해당
   - 여기서는 하단 삼각형만 생성 (상단 삼각형 생략)
   - `if (i != 0)` 조건으로 처리

2. **마지막 스택 (i=stackCount-1)**:
   - 하단 극점에 해당
   - 여기서는 상단 삼각형만 생성 (하단 삼각형 생략)
   - `if (i != (stackCount - 1))` 조건으로 처리

### 5.5 실제 구의 형태에 적용

이 평면 격자 구조가 실제 3D 구에 어떻게 매핑되는지:

1. 격자의 왼쪽/오른쪽 가장자리를 서로 연결하면 원통이 됩니다.
2. 그 다음 원통의 상단과 하단을 각각 한 점으로 압축하면 구가 됩니다.
3. 첫 번째 행(i=0)의 모든 정점은 실제로 구의 북극점에 해당합니다.
4. 마지막 행(i=stackCount)의 모든 정점은 실제로 구의 남극점에 해당합니다.

이러한 구조는 지구본의 경위도 체계와 유사합니다.

## 6. 요약

이 코드는 OpenGL에서 파라메트릭 방정식을 사용하여 구 지오메트리를 생성하는 방법을 보여줍니다. 핵심 단계는:

1. 구의 수학적 방정식을 사용하여 정점 위치 계산
2. 정점 간 관계를 정의하는 인덱스 데이터 생성
3. OpenGL 버퍼 객체 설정(VAO, VBO, EBO)
4. 셰이더 프로그램에 정점 속성 전달

이 방식으로 그래픽스 파이프라인이 효율적으로 3D 구 객체를 렌더링할 수 있습니다.
