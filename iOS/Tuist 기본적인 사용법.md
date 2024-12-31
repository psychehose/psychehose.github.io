 요즘 회사에서 Tuist를 사용하고 있다. 왜냐하면 iOS는 보일러 플레이트 코드가 거의 없다시피 하기 때문에 새로운 프로젝트를 할 때 상당히 귀찮기 때문이다. info.plist도 원하는대로 설정할 수 있고 3rd Party Library를 멋지게 관리할 수 있기 때문이다. Tuist는 이외에도 엄청나게 많은 기능을 제공하는 것 같다. 물론 많은 기능을 제공하는 만큼 제대로 사용하기 위해서는 학습이 필요하다. 어차피 설정은 프로젝트 동안에 거의 한번만 하면 되는데 Project 생성 도구를 학습 해야 하나 싶기도 하지만 이것은 Mono Project일 때이고 앱 규모가 크고 모듈화를 염두에 두고 있다면 Tuist는 매우 강력한 도구가 될 것 같다.

 개발을 Module 단위로 진행하고 있다면, 많은 Project를 만들어야 하는데 Xcode에서 Project 만들기는 너무 번거롭고 해야할 것이 많다. 가령 iOS deployment target을 13으로 잡았는데 생성시 최신버전으로 생성 되기때문에 Target Setting에서 바꿔야 한다거나 Framework 생성시 default가 dynamic framework이기 때문에 static으로 바꿔줘야 한다던가 매우 번거롭다. 이 때 Tuist를 사용할 경우 코드로 Project를 생성하기 때문에 한번 만들어 놓으면 재사용이 가능하고 훨씬 더 효율적으로 Project를 생성하고 유지보수가 가능해진다. 아무튼 그렇다고 한다.

---

```
$ curl -Ls <https://install.tuist.io> | bash
```

먼저 Tuist 부터 설치를 하자.

Terminal을 열고 위 명렁어를 입력하면 Tuist를 설치할 수 있다. 그리고 Tuist로 Project를 생성한다.

```
$ mkdir 'TuistPractice'  #폴더 생성
$ cd 'TuistPractice'
$ tuist init --platform ios # 이렇게 하면 tuist로 Project를 관리할 수 있는 환경이 만들어짐.
```

```
$ tuist generate # tuist로 project 생성
```

그러면 프로젝트가 생성된다. 아주 멋지게 잘 생성된 거 같다. App을 실행해보면 나오는 ㅂ구조가 Tuist가 제공하는 기본 Template이다. 그러면 어떻게 우리만의 Project 세팅을 할 수 있을까?

```
$ tuist edit
```

명령어를 입력하면 mainfests.xcodeproj가 열린다. 이건 Package.swift 같은 역할을 한다. 즉 Manifest.xcodeproj를 자기 입맛에 맞게 수정한 다음에 ‘tuist generate’를 하면 우리가 원하는 구조, 형태로 Project들을 세팅하고 생성할 수 있게 된다.

![](https://blog.kakaocdn.net/dn/HL4gh/btrH1bsSD2d/gPuie03lbnXlZPqRGLKFA1/img.png)

Tuist가 제공하는 기본 template

Project.swift에 Project.app()이 있는데 이 app() method는 Project + template.swift에 extension에 작성되어 있다. return 값은 Project (Tuist에서 정의된 Type)이다. 문자 그대로 Project다.

Tuist는 Project.swift를 찾아서 그 안에 작성되어 있는 대로 Project를 생성한다.

즉 우리는 Project.swift에 우리가 원하는 프로젝트 코드를 작성하면 된다.

보통 Project 코드를 작성할 때 템플릿처럼 Extension을 만들어서 사용하는 것 같다. 물론 나도 Extension을 만들어놓고 필요한 부분에서 바로 사용한다. 하지만 처음이니까 Extenstion을 만들지 않고 바로 사용해보자.

Project는 생성자는 다음과 같다.

1. name: String
2. organizationName: String
3. options: Project.Options
4. packages: [Package]
5. settings: Settings?
6. targets: [Target]
7. schemes: [Scheme]
8. fileHeaderTemplate: FileHeaderTemplate?
9. additionalFiles: [FileElement]
10. resourceSynthesizers: [ResourceSynthesizer]

생성자에 관한 자세한 설명은 Tuist Docs에서 확인 할 수 있다. Tuist는 이렇듯 코드로 많은 것들을 설정할 수 있게 한다. 이 부분은 아직 내 수준에서 그렇게까지 만질 필요가 없을 것 같기도 해서 정확하게 공부 해보지는 않았다.

우리에게 필요한 것은

3번 Options에 있는 Indent와 tab의 width 설정을 2로 바꾸는 것이었고 (xcode 기본설정은 4임!)

4번에서는 사용법을 알기 위해서 Alamofire를 넣도록 하고

5번에서는 Project 단위로 Project Settings를 사용하지는 않을 것이다.

7번 개발 앱과 상용앱을 나누기 위해서 필요한 기능이지만 여기서는 사용하지 않을 것이다.

8번, 9번, 10번은 잘 모르겠다. 어떻게 사용하는 지.

다만 내가 생각하기에 가장 중요한 것은 6번이라고 생각한다. 왜냐하면 우리는 실제로 여기서 작업을 하고 이것을 실행하기 때문이다.

따라서 우리는 간단하게 살펴보기 위해서 대부분 다 기본값으로 설정하고 필요한 부분만 작성해서 생성할 것이다.

이제 Project를 생성해보자

```
// Project.swift

let project = Project(
    name: "TuistPractice",
    organizationName: "com.yourCompany",
    packages: [],
    settings: .settings(),
    targets: [

    ]
)
```

package는 swift package다. 사용법을 알기 위해서 예시로 가장 유명한 라이브러리 중 하나인 Alamofire를 fetch 해봤다. 위에서 언급한 것처럼 가장 중요한 것은 target이다. targets 안에 target Target Type instance를 넣으면 되는데 보통 Main Target과, Test Target을 넣는데 Main Target만 넣도록 하자.

```
// Project.swift

let plist: [String: InfoPlist.Value] = [
    "CFBundleShortVersionString": "1.0",
    "CFBundleVersion": "1",
    "UIMainStoryboardFile": "",
    "UILaunchStoryboardName": "LaunchScreen",
    "NSAppTransportSecurity": .dictionary([
        "NSAllowsArbitraryLoads": .boolean(true)
    ])
]

let appTarget = Target(
    name: "TuistPractice",
    platform: .iOS,
    product: .app,
    bundleId: "YourBudleID",
    deploymentTarget: .iOS(targetVersion: "14.0", devices: [.iphone, .ipad]),
    infoPlist: .extendingDefault(with: plist),
    sources: ["Sources/**/*.swift"],
    resources: ["Resources/**"],
    settings: .settings(
        configurations: [
            .debug(
                name: "Development",
                settings: [:],
                xcconfig: .init("Configuration/TuistPractice-Development.xcconfig")
            ),
            .debug(
                name: "Staging",
                settings: [:], xcconfig: .init("Configuration/TuistPractice-Staging.xcconfig")
            ),
            .release(
                name: "Release",
                settings: [:],
                xcconfig: .init("Configuration/TuistPractice-Production.xcconfig")
            )
        ]
    )

```

Target Type도 마찬가지로 엄청나게 많은 생성자를 가지고 있는데 역시 이것도 자세한 설명은Tuist Docs에서 확인할 수 있다. Project에서 가장 중요한 것은 Target 이라고 했는데, Target에서 가장 중요한 것들은 product인 것 같다.

product에는 app, staticLibrary, staticFramework, framework, dynamic framework 등등이 있다. Tusit를 이용 하는 이유 중 하나는 모듈화 때문이기도 한데, 모듈화를 하기 위해서는 보통 모듈화 프로젝트 Main Target의 product를 static Library, staticFramework, framework로 만든다. 우리는 모듈화를 하기에 앞서, App Project를 만들어야 하기 때문에 product를 app으로 설정한다.

```
$ tuist fetch 
// 3rd party를 사용하고 있다면 프로젝트를 생성 하기전에 이 명령어를 입력해줘야 한다
$ tuist generate
```

![](https://blog.kakaocdn.net/dn/zl8Ah/btrH5fngRIJ/gJUyyvffudGvDKtJW1p5W1/img.png)

여기까지 했다면 다음과 같이 뜨면서 생성이 안될텐데 저 에러메세지를 잘 읽어보고 해결하면 된다. 우리가 Target을 만들 때 Source 파일들을 ["Sources/**/*.swift"] 이렇게 입력했기 때문에 이 path에 .swift 파일이 있어야 한다. 당연히 .xcconfig 파일도 마찬가지다.

![](https://blog.kakaocdn.net/dn/bbpiHp/btrH1Brklij/zTJSdTvBtBbQjZQRljnQk0/img.png)

![](https://blog.kakaocdn.net/dn/cqPBIG/btrH2hTEqBE/wqX37LiYkoGGsEMn54TPi0/img.png)

![](https://blog.kakaocdn.net/dn/nEBrR/btrH5d33ZBv/ylruykXbWknTH3KQWdlYC0/img.png)

```
$ tuist generate
```

![](https://blog.kakaocdn.net/dn/JgeLu/btrH6Hqi4LS/Mar5lLKlsYblYtqRTqedSK/img.png)

Project가 아주 잘 생성되었다. 참고로 Sources 폴더 안에 있는 것들은 프로젝트 생성 후 잘되는 지 테스트하기 위해 만든 것이다.

AppDelegate.swift, ViewController.swift 파일을 만들고 실행을 해보면

```
// AppDelegate.swift
import UIKit

@main
class AppDelegate: UIResponder, UIApplicationDelegate {

  var window: UIWindow?

  func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey : Any]? = nil
  ) -> Bool {

    let window = UIWindow(frame: UIScreen.main.bounds)
    window.backgroundColor = .white
    window.rootViewController = ViewController()
    window.makeKeyAndVisible()

    self.window = window

    return true
  }
}

// ViewController.swift

import UIKit

final class ViewController: UIViewController {
  override func viewDidLoad() {
    super.viewDidLoad()

    print("Hello Tuist")
    view.backgroundColor = .purple
  }
}

```

![](Build Setting 중 만난 에러프로젝트를 진행하면서 나중에 개발서버와 상용서버를 나눌 필요가 생길 거 같아서 빌드 세팅을 진행했다.

[https://ios-development.tistory.com/660](https://ios-development.tistory.com/660) 을 따라가며 진행했는데, 이런 오류를 만났다. 이것은, 실기기로 빌드 했을 때 뜨는 것이고, (기억 상) 시뮬레이터로 빌드를 하게 되면 Could not find module ‘Alamofire’ for target ‘i386-apple - ios - simulator가 떴다.

![](https://blog.kakaocdn.net/dn/dYH3Sc/btrH6Hw3dUq/vgTYdeRV9NEDh7tVvux9qk/img.png)

### TL; DR

```
// podfile

post_install do |installer|
  installer.pods_project.targets.each do |target|
    target.build_configurations.each do |config|
      config.build_settings.delete('ARCHS')
    end
  end
end
```

podfile에 다음과 같이 적으면 에러 해결!

### Procedure

문제는 Alamofire다.

위 스크린 샷의 에러가 Moya + Alamofire에서 발생하고, Alamofire를 사용하는 모든 Library에서 Compile error가 발생했다. (KakaoSDK, 커스텀한 Auth library ( inner DevPods))

빌드 세팅을 하기 전에는 발생하지 않았던 오류인데 빌드 세팅을 하고나서 오류가 발생했다는 점에서 일단 매우 이상했다. 아무튼 이 에러를 해결 하기 위해, 무수한 삽질을 했다.

Alamofire.framework를 /library/developer ~~~ /Alamofire/~~~ 뭐 이런 경로를 쭉 타고 가서 target을 확인 해봤는데 ‘x86_64’와 ‘arm64’ 두개가 있었다. 즉 armv7-apple-ios가 없어서 오류가 발생한 것이 맞다. (빌드 세팅을 하기 전에 단일 세팅일 떄는 왜 오류가 안났는 지는 아직도 의문이다.)

그래서, 네비게이션 영역을 통해서 Pods project로 들어가서 Alamofire의 Architecture 부분을 확인하였다.

Architectures의 부분이 $(ARCHS_STANDADRD_64_BIT)로 되어 있었다. 이 부분은을 STANDARD ARCH(arm7, arm64)로 고쳤을 때, 에러는 해결되었다. 다만 이렇게 설정했을 때, pod install을 다시하게 되면 본래대로 $(ARCHS_STANDADRD_64_BIT)로 바뀌어서 같은 에러가 발생한다.

그래서 pod install 할 때, architecture setting을 지워주는 코드를 추가 했다.

```
post_install do |installer|
  installer.pods_project.targets.each do |target|
    target.build_configurations.each do |config|
      config.build_settings.delete('ARCHS')
    end
  end
end
```

그리고 나서 Pod project의 Alamofire를 들어갔을 때, Architectures가 사라졌고 에러는 해결되었다.

### Why?

확실하진 않지만, 심증으로는 애플에서 디버그 용도는 32bit 아키텍쳐 ( 배포용도는 64bit도 )만을 지원하기 때문인 것 같다. 위에서 말한 것과 같이, 초기 세팅은 ARCHS_STANDARD_64_BIT 였기 때문이다.)

아주 잘 작동한다. 가장 기본적인 사용 방식은 이렇듯 매우 간단하다. 실제 진행하는 프로젝트에서는 모듈화를 하기 때문에 이것보다 더 복잡한 방식으로 사용하지만 기본적인 사용방식을 벗어나지는 않는다. 많은 iOS 개발자들이 Tuist를 사용해 봤으면 좋겠다.

끝.