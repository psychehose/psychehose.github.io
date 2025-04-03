
## 1. View3D의 이해

View3D는 Qt Quick 애플리케이션에서 3D 콘텐츠를 표시하는 핵심 컴포넌트입니다.
### 기본 구조
```qml
View3D {
    id: view3D
    anchors.fill: parent
    
    environment: SceneEnvironment {
        clearColor: "skyblue"
        antialiasingMode: SceneEnvironment.MSAA
        antialiasingQuality: SceneEnvironment.Medium
    }
    
    // 여기에 카메라, 조명, 모델 등이 배치
}
```

### View3D의 주요 속성

- **environment**: 3D 장면의 환경 설정 (배경색, 안티얼라이징 등)
- **renderMode**: 렌더링 방식 설정
- **camera**: 활성 카메라 지정

### SceneEnvironment 설정

- **clearColor**: 배경색 설정
- **antialiasingMode**: 계단 현상 방지 모드 (MSAA, SSAA 등)
- **antialiasingQuality**: 안티얼라이징 품질 (Low, Medium, High, VeryHigh)
- **backgroundMode**: 배경 모드 (단색, 스카이박스 등)

## 2. 카메라 (PerspectiveCamera)

카메라는 3D 공간을 어떤 시점에서 볼지 결정합니다.

### 기본 구조

```qml
PerspectiveCamera {
    id: camera
    position: Qt.vector3d(0, 10, 12)  // (x, y, z) 위치
    eulerRotation: Qt.vector3d(-20, 0, 0)  // (x, y, z) 회전 각도
    
    // 또는 lookAt 함수 사용
    function lookAt(targetPosition, upVector) {
        // 카메라가 특정 지점을 바라보도록 설정
    }
}
```

### 카메라 위치 설정 (position)

- **x**: 좌/우 위치 (양수: 오른쪽, 음수: 왼쪽)
- **y**: 높이 (양수: 위쪽, 음수: 아래쪽)
- **z**: 앞/뒤 위치 (양수: 뒤쪽, 음수: 앞쪽)

### 카메라 회전 (eulerRotation)

- **x축 회전**: 위/아래 시선 조절 (고개 끄덕이기, '-'값은 아래 보기) - pitch
- **y축 회전**: 좌/우 시선 조절 (고개 좌우로 돌리기)- yaw
- **z축 회전**: 카메라 기울기 (머리 기울이기) - roll

### lookAt 함수
- **targetPosition**: 바라볼 대상의 위치
- **upVector**: 카메라의 "위쪽" 방향 (보통 Qt.vector3d(0, 1, 0)), 법선벡터

### 카메라 종류
- **PerspectiveCamera**: 원근감 있는 일반적인 3D 시점
- **OrthographicCamera**: 원근감 없는 도면 같은 시점

## 3. 조명 (Lighting)

조명은 3D 객체를 비추어 보이게 하는 광원입니다.

### DirectionalLight (방향성 광원)

```qml
DirectionalLight {
    eulerRotation: Qt.vector3d(-30, 30, 0)  // 빛의 방향
    brightness: 0.7  // 밝기 (0.0 ~ 1.0)
    ambientColor: Qt.rgba(0.3, 0.3, 0.3, 1.0)  // 주변광
}
```

- **역할**: 태양광처럼 평행한 빛을 제공 (무한히 먼 곳에서 오는 빛)
- **eulerRotation**: 빛이 오는 방향 설정
- **brightness**: 빛의 강도 (0.0 ~ 1.0)
- **ambientColor**: 주변광의 색상과 강도

### PointLight (점광원)

```qml
PointLight {
    position: Qt.vector3d(0, 100, 0)  // 광원 위치
    brightness: 1.0
    color: "white"
    constantFade: 1.0
    linearFade: 0.0
    quadraticFade: 0.0
}
```

- **역할**: 전구처럼 모든 방향으로 빛을 발산
- **position**: 광원의 위치
- **색상 및 감쇠**: 거리에 따른 빛의 감소 설정

### SpotLight (스포트라이트)

```qml
SpotLight {
    position: Qt.vector3d(0, 100, 0)
    eulerRotation: Qt.vector3d(-90, 0, 0)
    brightness: 1.0
    coneAngle: 30.0  // 빛 원뿔의 각도
}
```

- **역할**: 원뿔 형태로 특정 방향을 비추는 빛
- **coneAngle**: 빛 원뿔의 각도

## 4. 3D 좌표계 이해

Qt Quick 3D는 오른손 좌표계를 사용합니다.

```
    y (위)
    |
    |
    +----> x (오른쪽)
   /
  /
 z (화면 안쪽)
```

### 좌표계 이해하기
- **오른손 좌표계**: 오른손 엄지(x), 검지(y), 중지(z)가 서로 수직인 방향
- **원점**: (0, 0, 0) 좌표
- **양수 방향**:
  - x: 오른쪽
  - y: 위쪽
  - z: 화면 안쪽(깊이)

### 회전과 오른손 법칙
- 엄지를 회전축 방향으로 향하면 나머지 손가락이 회전 방향
- x축 회전: 화면을 기준으로 상하 회전
- y축 회전: 화면을 기준으로 좌우 회전
- z축 회전: 화면을 기준으로 시계/반시계 회전

## 5. 실제 적용 예시 (골프공 시뮬레이터)

```qml
View3D {
    // 1. 환경 설정
    environment: SceneEnvironment {
        clearColor: "skyblue"  // 하늘색 배경
        antialiasingQuality: SceneEnvironment.Medium  // 품질 설정
    }
    
    // 2. 카메라 설정
    PerspectiveCamera {
        id: camera
        position: Qt.vector3d(0, 10, 12)  // 지면보다 위쪽, 약간 뒤쪽에서 바라봄
    }
    
    // 3. 조명 설정
    DirectionalLight {
        eulerRotation: Qt.vector3d(0, 0, 0)  // 정면에서 비추는 빛
        brightness: 0.7  // 밝기 70%
        ambientColor: Qt.rgba(0.3, 0.3, 0.3, 1.0)  // 주변광
    }
    
    // 4. 지면 모델
    Model {
        id: ground
        position: Qt.vector3d(0, 0, 0)  // 원점에 위치
        scale: Qt.vector3d(500, 0.1, 500)  // 넓고 얇은 판
        source: "#Cube"  // 내장 큐브 모델 사용
        materials: DefaultMaterial {
            diffuseColor: "green"  // 녹색 지면
        }
    }
    
    // 5. 구
    Model {
        id: ball
        source: "#Sphere"  // 내장 구체 모델
        scale: Qt.vector3d(0.3, 0.3, 0.3)  // 크기 조정
        position: Qt.vector3d(0, 5.0, 0)  // 초기 위치
        materials: DefaultMaterial {
            diffuseColor: "white"  // 흰색
            specularAmount: 0.9  // 반사도
        }
    }
}
```


