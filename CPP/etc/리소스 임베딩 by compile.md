
크로스플랫폼을 타겟으로 하는 C++ 라이브러리를 개발하면서 리소스를 포함해야 하는 경우가 있다.

PC 기반의 환경 (macOS, Windows, Linux 등등)에서 파일 경로를 이용해서 `fopen()` 으로 리소스에 접근할 수 있다. 그러나 샌드박스 환경 (Android, iOS 등)에서는 파일 시스템 접근이 제한된다. 그래서 각 플랫폼 API를 이용하는 방법으로 리소스에 접근하거나 **이미지, 오디오, 텍스트 파일등의 바이너리 리소스를 C / C++ 소스코드 배열로 변환하는 리소스 임베딩** 방식을 사용한다.


#### 동작 방식

1. 리소스 파일을 바이트 배열로 변환한다.
2. 바이트 배열을 C++ 소스 코드 파일에 포함시킨다.
3. 컴파일을 할 때 이 배열이 프로그램의 바이너리에 포함된다.
4. 런타임에 이 배열에 접근해서 리소스를 사용한다.


리소스 파일을 바이트 배열로 변환할 때 Python 스크립트나 `bin2c`, CMake의 `configure_file`을 사용한다.

