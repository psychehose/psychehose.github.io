
### asdf

Rust를 공부하기 위해서는 Rust를 설치해야 합니다. 저는 개발도구나, 환경은 asdf를 이용합니다. asdf는 도구 버전 관리자입니다. 플러그인을 통해서 정말 많은 것들을 가상환경으로 구성할 수 있게 도와줘요 python, node, jdk 등 우리가 사용하는 거의 대부분을 지원하고 있습니다. Rust 역시 지원하기 때문에 asdf를 사용해서 설치하도록 하겠습니다.

asdf가 더 알고 싶으신 분은 공식 홈페이지를 참고해주세요.

[https://asdf-vm.com/guide/introduction.html](https://asdf-vm.com/guide/introduction.html)

### Rust 설치

```
# 러스트 플러그인을 먼저 설치합니다.
$ asdf plugin add rust

# 설치 가능한 러스트 버전을 확인합니다.
$ asdf list all rust

# 자기에게 알맞은 버전을 설치해주세요.
# 러스트 공홈 release 버전을 확인해서 받으셔도 될 것 같습니다.
# https://www.rust-lang.org/
$ asdf install rust 1.67.1

# 설치된 러스트 버전을 확인합니다.
$ asdf list rust

# global 또는 local을 이용해서 환경을 적용합니다.
$ asdf global rust 1.67.1
```

그러고 나서 버전을 확인합니다.

```
$ rustc --version
$ cargo --version
```

cargo는 러스트 프로젝트를 관리하는 빌드시스템 및 패키지 매니지입니다. 여기까지 하면 러스트 설치가 끝났습니다.

### IntelliJ에서 Rust 사용하기 그리고 asdf 사용 시에 IntelliJ가 Path를 못 찾는 이슈

저는 개발을 좀 더 편하게 하기 위해서 IDE를 사용하는데요. IntelliJ를 사용하고 있습니다. IntelliJ에 러스트 플러그인을 설치만 하면 바로 사용할 수 있습니다. (CLion도 같은 방식으로 플러그인을 추가할 수 있습니다.) 그런데, asdf를 사용해서 그런가 IntelliJ가 프로젝트를 생성할 때, rust를 찾지 못하는 것 같아요. 이를 어떻게 해결했는지 쓰겠습니다.

 **1. Plugins에서 Rust 검색 후 다운로드**

![](https://blog.kakaocdn.net/dn/l1BXk/btsbQRluIcE/gKu7klUAuMUvwEKUzNyiIk/img.png)

 **2.  Toolchain location을 바로 찾을 수 없음.**  
 asdf로 설정한 러스트 경로를 찾지 못해서, 아래 스크린샷 왼쪽처럼, 프로젝트를 만들 수 없습니다. 이 경로를 직접 설정하면 되지 않을까 해서, bin이 있는 곳을 찾아서, 경로를 넣었습니다. 그러면 아래 스크린샷 오른쪽처럼, 정상적으로 작동합니다. 하지만 매번 프로젝트마다, 직접 경로를 찾아서 넣어주는 것은 불편합니다. 이것은 어떻게 해결할 수 있을까요?

```
$ cd ~/.asdf/installs/rust/1.67.1/toolchains/1.67.1-aarch64-apple-darwin/bin
pwd

# pwd에 나온 결과를 붙여 넣기
# 예시
#/Users/psychehose/.asdf/installs/rust/1.67.1/toolchains/1.67.1-aarch64-apple-darwin/bin
```

![](https://blog.kakaocdn.net/dn/b7tjcf/btsbSK6Uj6N/tk3lzlsDrgkaXmrPl5F0n0/img.png)![](https://blog.kakaocdn.net/dn/uvzmo/btsbSfGaREc/vKJt2xgIqDD7eyDMiV0WlK/img.png)

 **3. Tool-chain location 디폴트 설정 바꾸기**

저는 이것을 asdf-shims와 IntelliJ 기본 설정을 바꿔서 해결했습니다. 

> (IntelliJ의) Preference -> Languages & Frameworks -> Rust

![](https://blog.kakaocdn.net/dn/bkUIU0/btsbSgd77OO/oiBzpVLOCvck39evdMPYe0/img.png)

 그러면 항상 새로운 프로젝트에 들어갔을 때,  기본설정으로 들어가 있게 됩니다. 이제 IntelliJ로 러스트 프로젝트를 시작할 수 있게 되었어요.

### Ref.

[https://github.com/intellij-rust/intellij-rust/issues/5315](https://github.com/intellij-rust/intellij-rust/issues/5315)