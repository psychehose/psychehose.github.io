## OpenGL 버전 설정

Qt에서 OpenGL을 사용할 때 QSurfaceFormat을 통해 버전을 설정 해야 한다.


```cpp
// main.cpp
QSurfaceFormat format;
format.setVersion(3, 3);             // OpenGL 3.3 사용
format.setProfile(QSurfaceFormat::CoreProfile); // Core 프로파일 사용
format.setDepthBufferSize(24);
format.setStencilBufferSize(8);
format.setSamples(4);                // 멀티샘플링 설정
QSurfaceFormat::setDefaultFormat(format); // 중요: 전역 설정으로 지정
```

## macOS의 OpenGL 지원

1. **기본 지원**: macOS는 기본적으로 OpenGL 2.1(레거시 프로파일)을 지원함.
2. 확장 지원: macOS 10.9 (Mavericks) 이상에서는 OpenGL 3.3 Core 프로파일 및 최대 4.1까지 지원

#### OpenGL 버전 확인

```cpp
QOpenGLContext context;
if (context.create()) {
    qDebug() << "OpenGL Version:" << context.format().majorVersion() << "." << context.format().minorVersion();
    qDebug() << "Profile:" << (context.format().profile() == QSurfaceFormat::CoreProfile ? "Core" : "Compatibility");
}
```

macOS에서 OpenGL 3.3으로 설정했는데 실제로는 4.1이 표시되는 현상이 있을 수 있다. 왜냐하면 OpenGL은 하위 호환성이 있는 API로, 요청한 버전보다 높은 버전이 지원된다면 드라이버는 보통 지원되는 가장 높은 버전을 선택한다. 하위 호환성을 거의 보장하므로 3.3 코드는 4.1에서도 문제없이 작동한다.