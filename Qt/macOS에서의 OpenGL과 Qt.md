# macOS에서의 OpenGL과 Qt 프레임워크

## OpenGL 버전 설정

Qt에서 OpenGL을 사용할 때 QSurfaceFormat을 통해 버전을 설정할 수 있습니다:

```cpp
QSurfaceFormat format;
format.setVersion(3, 3);             // OpenGL 3.3 사용
format.setProfile(QSurfaceFormat::CoreProfile); // Core 프로파일 사용
format.setDepthBufferSize(24);
format.setStencilBufferSize(8);
format.setSamples(4);                // 멀티샘플링 설정
QSurfaceFormat::setDefaultFormat(format); // 중요: 전역 설정으로 지정
```

## macOS의 OpenGL 지원 특성

1. **기본 지원**: macOS는 기본적으로 OpenGL 2.1(레거시 프로파일)을 지원
2. **확장 지원**: macOS 10.9 (Mavericks) 이상에서는 OpenGL 3.3 Core 프로파일 및 최대 4.1까지 지원
3. **Apple 정책**: Apple은 2018년부터 OpenGL을 더 이상 발전시키지 않기로 결정하고 Metal API로의 전환을 권장

## 버전 설정과 실제 동작 차이

macOS에서 OpenGL 3.3으로 설정했는데 실제로는 4.1이 표시되는 현상:

```cpp
// 설정
format.setVersion(3, 3);
// 확인 결과
qDebug() << "OpenGL 버전:" << format.majorVersion() << "." << format.minorVersion();
// 출력: OpenGL 버전: 4 . 1
```

이런 현상이 발생하는 이유:
1. **드라이버의 버전 선택**: OpenGL은 하위 호환성이 있는 API로, 요청한 버전보다 높은 버전이 지원된다면 드라이버는 보통 지원되는 가장 높은 버전을 선택
2. **macOS의 OpenGL 구현 방식**: macOS는 최대 OpenGL 4.1까지 지원하며, 3.3을 요청해도 실제로는 4.1 컨텍스트를 제공
3. **호환성 보장**: 더 높은 버전의 OpenGL은 이전 버전의 기능을 모두 포함하므로 3.3 코드는 4.1에서도 문제없이 작동

## 실제 OpenGL 버전 확인 방법

```cpp
QOpenGLContext context;
if (context.create()) {
    qDebug() << "OpenGL Version:" << context.format().majorVersion() << "." << context.format().minorVersion();
    qDebug() << "Profile:" << (context.format().profile() == QSurfaceFormat::CoreProfile ? "Core" : "Compatibility");
}
```

## 크로스 플랫폼 개발 고려사항

### OpenGL에서 OpenGL ES로의 전환

**소요 시간 요소**:
- 단순한 렌더링: 몇 시간~며칠
- 복잡한 셰이더와 고급 기능: 몇 주~몇 달
- Qt 프레임워크 사용 시 변환 작업이 간소화됨

**주요 변경 사항**:
- 고정 파이프라인 제거 (OpenGL ES 2.0+)
- 정밀도 한정자 추가 (highp, mediump, lowp)
- GLSL 문법 차이 대응

**추천 접근법**:
- 처음부터 OpenGL ES 3.0 호환 코드 작성
- Qt 6의 RHI(Rendering Hardware Interface) 고려
- Qt의 추상화 클래스(QOpenGLFunctions) 활용

## 골프공 궤적 시뮬레이션을 위한 권장사항

1. 크로스 플랫폼 지원을 위해 OpenGL ES 호환 코드 작성
2. Qt 6 이상 버전 사용 시 RHI 활용 고려
3. macOS 타겟의 경우 장기적으로 Metal API 지원 검토
4. 모바일 플랫폼(iOS, Android) 지원을 위해 OpenGL ES 기반으로 개발

## 참고 사항

- LGPL 라이센스 모듈 사용 시 라이센스 요구사항 준수
- 여러 플랫폼(MacOS, Windows, Linux, iOS, Android) 지원을 위한 조건부 컴파일 고려
- 개발환경은 MacOS이나 타겟 플랫폼별 테스트 필수
