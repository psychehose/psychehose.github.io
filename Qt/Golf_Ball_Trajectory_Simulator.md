# Qt Quick 3D 골프공 궤적 시뮬레이터

> 최종 업데이트: 2025-03-26

## 프로젝트 개요

이 프로젝트는 LGPL 라이센스를 준수하는 Qt 모듈을 사용하여 골프공의 궤적을 시뮬레이션하는 앱을 개발하는 과정을 기록합니다. 이 앱은 MacOS, Windows, Linux, iOS, Android를 타겟으로 하며, 현재 개발 환경은 MacOS입니다.

### 목적
- Qt Quick 3D를 활용한 골프공 궤적 시뮬레이션 프로토타입 개발
- 향후 자체 제작 골프공 궤적 계산 라이브러리 연동을 위한 테스트 앱 구현

## 기술 스택

- Qt 6.8
- Qt Quick 3D (3D 렌더링)
- CMake (빌드 시스템)
- C++ (백엔드)
- QML (프론트엔드)

## 구현 단계

### 1. 프로젝트 설정

프로젝트 구조를 설정하고 Qt Quick 3D 종속성을 추가했습니다.

```cmake
# CMakeLists.txt
cmake_minimum_required(VERSION 3.16)
project(Simulator VERSION 0.1 LANGUAGES CXX)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Qt Quick 3D 종속성 추가
find_package(Qt6 REQUIRED COMPONENTS Quick Quick3D)

qt_standard_project_setup(REQUIRES 6.8)

qt_add_executable(appSimulator
    main.cpp
)

qt_add_qml_module(appSimulator
    URI Simulator
    VERSION 1.0
    QML_FILES
        Main.qml
)

# 여러 플랫폼 설정
set_target_properties(appSimulator PROPERTIES
    MACOSX_BUNDLE_BUNDLE_VERSION ${PROJECT_VERSION}
    MACOSX_BUNDLE_SHORT_VERSION_STRING ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}
    MACOSX_BUNDLE TRUE
    WIN32_EXECUTABLE TRUE
)

# Qt Quick 3D 링크
target_link_libraries(appSimulator
    PRIVATE Qt6::Quick Qt6::Quick3D
)
```

### 2. 메인 C++ 파일

기본적인 Qt 애플리케이션을 설정합니다.

```cpp
// main.cpp
#include <QGuiApplication>
#include <QQmlApplicationEngine>

int main(int argc, char *argv[])
{
    QGuiApplication app(argc, argv);

    QQmlApplicationEngine engine;
    QObject::connect(
        &engine,
        &QQmlApplicationEngine::objectCreationFailed,
        &app,
        []() { QCoreApplication::exit(-1); },
        Qt::QueuedConnection);
    engine.loadFromModule("Simulator", "Main");

    return app.exec();
}
```

### 3. Qt Quick 3D 골프공 궤적 시뮬레이터 구현

QML을 사용하여 골프공 궤적 시뮬레이션을 위한 3D 환경을 구현했습니다.

#### 주요 기능:
- **3D 시각화**: Qt Quick 3D 기반의 골프 시뮬레이션 환경
- **물리 계산**: 골프공의 발사 각도와 속도에 기반한 궤적 계산
- **사용자 상호작용**: 발사 각도와 속도를 슬라이더로 조절 가능
- **궤적 시각화**: 골프공의 이동 경로 시각화

### 4. 사용자 인터페이스 구성

1. **3D 뷰**: 골프공과 궤적이 표시되는 메인 화면
2. **컨트롤 패널**: 
   - 발사 각도 슬라이더 (0-90°)
   - 볼 속도 슬라이더 (10-150 m/s)
   - 발사 버튼 (Launch Ball)
   - 리셋 버튼 (Reset) - 2025-03-26 추가

### 5. 물리 모델 구현

간단한 물리 모델을 구현하여 골프공의 궤적을 계산합니다:

```javascript
// 골프공 위치 계산 코드
var angle = ballLaunchAngle * (Math.PI / 180)
var vx = ballSpeed * Math.cos(angle)
var vy = ballSpeed * Math.sin(angle)
var x = vx * simulationTime
var y = vy * simulationTime - 0.5 * gravity * simulationTime * simulationTime
```

이 구현은 다음 물리 공식에 기반합니다:
- x(t) = v₀ cos(θ) × t
- y(t) = v₀ sin(θ) × t - ½gt²

여기서:
- v₀: 초기 속도
- θ: 발사 각도
- g: 중력 가속도 (9.81 m/s²)
- t: 시간

### 6. 발생한 문제점과 해결

Qt Quick 3D 모듈 사용 시 발생한 문제:
- `PrincipalMaterial` 타입이 존재하지 않는 오류 → `DefaultMaterial`로 대체
- `roughness` 속성이 존재하지 않는 오류 → `specularAmount` 속성으로 대체

이러한 문제들은 Qt 버전이나 모듈에 따른 API 차이로 인해 발생할 수 있으며, 공식 문서를 참조하여 적절한 속성과 타입을 사용하는 것이 중요합니다.

### 7. Reset 버튼 정보 (2025-03-26 추가)

이 버튼은 시뮬레이션을 초기 상태로 돌리는 기능을 합니다.

```qml
Button {
    text: "Reset"
    onClicked: {
        // Reset everything to initial state
        simulationTime = 0.0
        simulationRunning = false
        golfBall.position = Qt.vector3d(0, 0, 0)
        trajectoryModel.clear()
        // 슬라이더도 초기값으로 리셋
        root.ballLaunchAngle = 45.0
        root.ballSpeed = 70.0
        angleSlider.value = 45.0
        speedSlider.value = 70.0
    }
}
```

Reset 버튼을 클릭하면 다음 항목들이 모두 초기 상태로 돌아갑니다:

- 시뮬레이션 시간 (0으로 초기화)
- 시뮬레이션 상태 (정지 상태로 변경)
- 골프공 위치 (원점으로 이동)
- 궤적 데이터 (모두 삭제)
- 발사 각도 슬라이더 (45도로 초기화)
- 볼 속도 슬라이더 (70 m/s로 초기화)

### 8. 전체 Main.qml 코드 설명

#### 개요

Main.qml 파일은 이 프로젝트의 핵심 파일로, Qt Quick 3D를 사용하여 골프공 궤적 시뮬레이터의 전체 사용자 인터페이스와 기능을 구현합니다. 이 파일을 구성하는 주요 요소와 기능에 대해 자세히 살펴보겠습니다.

#### 임포트 및 기본 구조

```qml
import QtQuick
import QtQuick.Controls
import QtQuick3D
import QtQuick.Layouts

Window {
    id: root
    width: 1024
    height: 768
    visible: true
    title: qsTr("Golf Ball Trajectory Simulator")
    color: "#f0f0f0"
    
    // 속성 정의, 타이머, UI 요소 등
}
```

기본 창 설정과 필요한 Qt 모듈을 임포트합니다. 이 프로젝트에서는 다음 모듈이 사용됩니다:
- `QtQuick`: 기본 UI 요소 제공
- `QtQuick.Controls`: 버튼, 슬라이더 등의 컨트롤 제공
- `QtQuick3D`: 3D 렌더링 기능 제공
- `QtQuick.Layouts`: 레이아웃 관리 기능 제공

#### 속성 정의

```qml
property real ballLaunchAngle: 45.0
property real ballSpeed: 70.0
property real gravity: 9.81
property real simulationTime: 0.0
property real maxSimulationTime: 10.0
property bool simulationRunning: false
```

시뮬레이션에 필요한 다양한 물리적 변수와 상태를 정의합니다:
- `ballLaunchAngle`: 골프공 발사 각도 (도)
- `ballSpeed`: 골프공 발사 속도 (m/s)
- `gravity`: 중력 가속도 (m/s²)
- `simulationTime`: 현재 시뮬레이션 시간
- `maxSimulationTime`: 최대 시뮬레이션 시간
- `simulationRunning`: 시뮬레이션 실행 상태

#### 시뮬레이션 타이머

```qml
Timer {
    id: simulationTimer
    interval: 16 // ~60fps
    running: simulationRunning
    repeat: true
    onTriggered: {
        // 시뮬레이션 로직
    }
}
```

이 타이머는 약 60fps로 실행되며, 골프공의 위치를 계산하고 업데이트하는 핵심 로직을 구현합니다. `simulationRunning` 속성에 따라 타이머 실행 여부가 결정됩니다.

#### 물리 계산

```qml
var angle = ballLaunchAngle * (Math.PI / 180)
var vx = ballSpeed * Math.cos(angle)
var vy = ballSpeed * Math.sin(angle)
var x = vx * simulationTime
var y = vy * simulationTime - 0.5 * gravity * simulationTime * simulationTime
```

포물선 운동 방정식을 사용하여 골프공의 위치를 시간에 따라 계산합니다:
- 각도를 라디안으로 변환
- x 방향 및 y 방향 초기 속도 계산
- 시간에 따른 x, y 위치 계산 (포물선 운동)

#### 사용자 인터페이스 레이아웃

```qml
ColumnLayout {
    anchors.fill: parent
    spacing: 10
    
    View3D {
        // 3D 환경 정의
    }
    
    Rectangle {
        // 컨트롤 패널
    }
}
```

사용자 인터페이스는 두 개의 주요 부분으로 구성됩니다:
1. **3D 뷰**: 골프공과 궤적을 표시하는 메인 화면
2. **컨트롤 패널**: 슬라이더와 버튼이 포함된 하단 패널

#### 3D 환경 구성

```qml
View3D {
    id: view3D
    Layout.fillWidth: true
    Layout.fillHeight: true
    Layout.margins: 10
    environment: SceneEnvironment {
        clearColor: "skyblue"
        backgroundMode: SceneEnvironment.Color
        antialiasingMode: SceneEnvironment.MSAA
        antialiasingQuality: SceneEnvironment.High
    }
    
    // 카메라, 조명, 모델 등
}
```

`View3D`는 Qt Quick 3D의 핵심 컴포넌트로, 3D 렌더링 환경을 설정합니다. 여기에는 다음이 포함됩니다:
- 하늘색 배경
- 안티앨리어싱 처리
- 카메라, 조명, 3D 모델 등

#### 카메라 및 조명

```qml
PerspectiveCamera {
    id: camera
    position: Qt.vector3d(150, 75, 200)
    eulerRotation: Qt.vector3d(-20, 30, 0)
}

DirectionalLight {
    eulerRotation: Qt.vector3d(-30, 30, 0)
    brightness: 1.0
    ambientColor: Qt.rgba(0.3, 0.3, 0.3, 1.0)
}
```

- `PerspectiveCamera`: 3D 환경을 보는 시점 설정
- `DirectionalLight`: 방향성 조명으로 장면에 그림자 효과 추가

#### 3D 모델

```qml
// 지면 모델
Model {
    id: ground
    position: Qt.vector3d(150, -0.5, 0)
    scale: Qt.vector3d(100, 0.1, 20)
    source: "#Cube"
    materials: DefaultMaterial {
        diffuseColor: "green"
        specularAmount: 0.2
    }
}

// 골프공 모델
Model {
    id: golfBall
    source: "#Sphere"
    scale: Qt.vector3d(0.5, 0.5, 0.5) // 2025-03-26 수정: 1.5에서 0.5로 축소
    position: Qt.vector3d(0, 0, 0)
    materials: DefaultMaterial {
        diffuseColor: "white"
        specularAmount: 0.9
    }
}
```

두 가지 주요 3D 모델이 있습니다:
- `ground`: 평평한 녹색 지면을 나타내는 큐브 모델
- `golfBall`: 흰색 골프공을 나타내는 구체 모델

#### 궤적 시각화

```qml
Node {
    id: trajectoryNode
    
    ListModel {
        id: trajectoryModel
    }
    
    Repeater3D {
        model: trajectoryModel
        delegate: Model {
            source: "#Sphere"
            scale: Qt.vector3d(0.3, 0.3, 0.3)
            position: model.position
            materials: DefaultMaterial {
                diffuseColor: "red"
                specularAmount: 0.5
            }
        }
    }
}
```

골프공의 궤적은 다음과 같이 시각화됩니다:
- `ListModel`을 사용하여 궤적 포인트 저장
- `Repeater3D`를 사용하여 각 포인트에 작은 빨간색 구체 생성
- 시뮬레이션이 진행됨에 따라 새 포인트가 모델에 추가됨

#### 컨트롤 패널

```qml
Rectangle {
    Layout.fillWidth: true
    Layout.preferredHeight: 150
    Layout.margins: 10
    color: "#e0e0e0" // 2025-03-26 수정: 가시성 개선을 위해 색상 변경
    radius: 5
    border.color: "#cccccc"
    
    GridLayout {
        // 슬라이더 및 버튼
    }
}
```

컨트롤 패널은 다음과 같은 UI 요소를 포함합니다:
- 각도 슬라이더 (0-90°)
- 속도 슬라이더 (10-150 m/s)
- Launch Ball 버튼 (시뮬레이션 시작)
- Reset 버튼 (2025-03-26 추가)

#### 슬라이더

```qml
Slider {
    id: angleSlider
    from: 0
    to: 90
    value: root.ballLaunchAngle
    stepSize: 1
    Layout.fillWidth: true
    onValueChanged: root.ballLaunchAngle = value
}

Slider {
    id: speedSlider
    from: 10
    to: 150
    value: root.ballSpeed
    stepSize: 1
    Layout.fillWidth: true
    onValueChanged: root.ballSpeed = value
}
```

각 슬라이더는 다음과 같이 작동합니다:
- 사용자가 값을 변경하면 해당 속성(각도 또는 속도)이 업데이트됨
- 범위와 단계 크기 설정

#### 버튼 구현

```qml
Button {
    text: "Launch Ball"
    onClicked: {
        // 시뮬레이션 초기화 및 시작
        simulationTime = 0.0
        golfBall.position = Qt.vector3d(0, 0, 0)
        trajectoryModel.clear()
        simulationRunning = true
    }
}

Button {
    text: "Reset"
    onClicked: {
        // 모든 설정을 초기값으로 리셋
        simulationTime = 0.0
        simulationRunning = false
        golfBall.position = Qt.vector3d(0, 0, 0)
        trajectoryModel.clear()
        root.ballLaunchAngle = 45.0
        root.ballSpeed = 70.0
        angleSlider.value = 45.0
        speedSlider.value = 70.0
    }
}
```

두 버튼은 다음과 같은 기능을 제공합니다:
- **Launch Ball**: 궤적을 초기화하고 시뮬레이션 시작
- **Reset**: 모든 설정과 상태를 초기값으로 복원

## 다음 단계

1. **자체 라이브러리 통합**: 자체 제작 C++ 골프공 궤적 계산 라이브러리 연동
2. **공기 저항 모델 추가**: 더 정확한 궤적 계산을 위한 공기 저항 고려
3. **3D 모델 개선**: 골프공과 골프 코스의 더 상세한 3D 모델 추가
4. **UI 개선**: 사용자 경험 향상을 위한 인터페이스 개선
5. **멀티플랫폼 테스트**: 다양한 플랫폼에서의 동작 테스트

## 결론

이 프로젝트는 Qt Quick 3D를 사용하여 골프공 궤적 시뮬레이션의 기본 프로토타입을 구현했습니다. 현재는 간단한 물리 모델을 사용하고 있지만, 향후 자체 제작 라이브러리를 통합하여 더 정확한 시뮬레이션을 구현할 계획입니다.

## 최근 업데이트 내역

### 2025-03-26 변경사항

| 변경 사항     | 설명                                    |
| --------- | ------------------------------------- |
| 골프공 크기 축소 | 골프공 크기를 1.5에서 0.5로 축소하여 더 현실적인 비율로 조정 |
| UI 가시성 개선 | 컨트롤 패널 배경색을 변경하고 라벨 색상을 진하게 하여 가독성 향상 |
| 리셋 버튼 추가  | 모든 설정과 상태를 초기값으로 되돌리는 버튼 구현           |

### 다음 작업 예정

- 스핀(spin) 변수 및 계산 추가
- 공기 저항 모델 개선
- UI 상단에 현재 설정 요약 표시

## 실행 방법

```bash
cd /path/to/Simulator
mkdir -p build && cd build
cmake ..
make
./appSimulator
```
