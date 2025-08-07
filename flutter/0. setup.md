
vscode를 이용해서 설치

1. 플러터 익스텐션 설치
2. command + shift + p
3. flutter 검색
4. Flutter: New Project 선택
5. Download SDK 선택 후 위치는 ~/Develop으로 지정


```zsh
$ flutter doctor -v
```

찾을 수 없는 명령어면 환경변수에 지정

``` zsh
$ echo 'export PATH=$PATH:~/Develop/flutter/bin' >> ~/.zshrc
```


그런 다음에 flutter doctor -v를 하면 플러터 개발시 필요한 도구들을 알려줌.

그에 맞게 업데이트 하거나 설치를 하면 됨.

나 같은 경우는 CocoaPods의 버전을 업데이트 (1.12.1 -> 1.16.2)로 해줘야하고, 안드로이드 툴체인을 설치 해야한다.

#### CocoaPods update

나는 예전에 cocoapods을 설치했을 때 homebrew를 이용해서 설치했기 때문에 homebrew를 통해서 업그레이드 했다.

```zsh
$ brew upgrade cocoapods
```

#### Android toolchain
안드로이드 툴체인은 안드로이드 스튜디오를 다운로드 하고 아래 구성요소를 설치하면 됨

- Android SDK Platform, API 35.0.2
- Android SDK Command-line Tools
- Android SDK Build-Tools
- Android SDK Platform-Tools
- Android Emulator



