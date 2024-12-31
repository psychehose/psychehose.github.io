
Tuist를 이용해서 새로운 프로젝트를 시작했다. 어떤 방식으로 프로젝트를 설계할까에 대해 고민을 하고 많은 블로그들을 읽었고, 고민을 거듭한 끝에 다음 같이 구성했다. 처음에 설계한 방식은 그래프에 나와있지는 않지만 CoreKit 아래에 RxPackage를 가지고 있었다.

![](https://blog.kakaocdn.net/dn/yMkiy/btrBJMGgTgu/nkPRqkuukxwk7EW5249F1k/img.png)

```
// Package.swift
// swift-tools-version: 5.5
// The swift-tools-version declares the minimum version of Swift required to build this package.

import PackageDescription

let package = Package(
    name: "RxPackage",
    products: [
        // Products define the executables and libraries a package produces, and make them visible to other packages.
        .library(
            name: "RxPackage",
            targets: ["RxPackage"]),
    ],
    dependencies: [
        .package(url: "https://github.com/ReactiveX/RxSwift.git", from: "6.5.0"),
        .package(url: "https://github.com/uber/RIBs.git", from: "0.9.0"),
    ],
    targets: [
        // Targets are the basic building blocks of a package. A target can define a module or a test suite.
        // Targets can depend on other targets in this package, and on products in packages this package depends on.
        .target(
            name: "RxPackage",
            dependencies: [
                "RxSwift",
                "RIBs"
            ]),
        .testTarget(
            name: "RxPackageTests",
            dependencies: ["RxPackage"]),
    ]
)
```

RIBs는 RxSwift와 RxRelay에 의존성을 가지기 때문에 Package에서 url을 통해 fetch 하고 의존성을 해결하였다. 여기서 문제가 발생했다.

## 문제점 1. Module에서 RxCocoa를 import 할 수 없다.

Feature RIBs에서 RxCocoa의 rx.tap과 같은 기능들을 사용할 수가 없었다. 왜냐하면 RxPackage는 dependency로 RxCocoa를 가지고 있지 않기 때문. 위의 Package.swift를 다음처럼 조금만 더 수정하면 되지 않을까?

```
// Package.swift
// swift-tools-version: 5.5
// The swift-tools-version declares the minimum version of Swift required to build this package.

import PackageDescription

let package = Package(
    targets: [
       // 생략
        .target(
            name: "RxPackage",
            dependencies: [
                "RxSwift",
                "RIBs",
                .product(name: "RxCocoa", package: "RxSwift")
            ]),
			// 생략
    ]
)
```

## 문제점 2. Missing required module RxCocoaRuntime

import RxCocoa가 되고 RxCocoa의 기능을 사용할 수 있었다.(자동완성) 그런데 빌드를 하고 나면 Missing required module RxCocoaRuntime이라는 Error가 발생하고 앱이 죽었다. 검색을 통해서 RxSwift Issue에서 관련된 두 가지 질문을 찾을 수 있었다.

[https://github.com/ReactiveX/RxSwift/issues/2210](https://github.com/ReactiveX/RxSwift/issues/2210) 이 이슈에서는 UITest target에서 발생한 Issue이고 해당 답변에서 SPM Bug라고 한다.

[https://github.com/ReactiveX/RxSwift/issues/2057](https://github.com/ReactiveX/RxSwift/issues/2057)

다음은 아래와 같은 해결책을 제시하였는데, 적용을 해봤지만 해결이 되지 않았다.

![](https://blog.kakaocdn.net/dn/bxMQPw/btrBLVChZ5D/0NKg3NZ18iPPgW5nyq8dw0/img.png)

왜 안되는 지 정확하게 원인을 파악할 수 없었기 때문에 과감하게 Package를 이용해서 RIBs를 사용하는 방식을 포기하고 다른 방법을 모색하기 시작했다.

## Carthage로의 이주

많은 iOS 개발자들이 Carthage를 사용하기 때문에 tuist도 Carthage를 아주 잘 지원한다. (CocoaPod은 이제 지원 안 한다고 한다. 많은 사람들이 아직 많이 사용하는데...)

Carthage를 이용하기 위해서 tuist edit 명렁어를 입력하고 Dependencies 파일에 Rx와 RIBs를 추가했다.

그러고 나서 RxPackageKit이라는 Project를 만들고 이것은 ‘RxSwift’, ‘RIBs’ Dependencies를 가지고 있다.

- (Project.dynamicFramework()는 extension 입니다.)

```
//
//  Dependencies.swift
//  ProjectDescriptionHelpers
//
//  Created by 
//

import ProjectDescription

let dependencies = Dependencies(
    carthage: [
					// ...
        .github(path: "ReactiveX/RxSwift", requirement: .exact("6.5.0")),
        .github(path: "uber/RIBs", requirement: .branch("main")),
					// ...
    ],
    swiftPackageManager: [
    ],
    platforms: [.iOS]
)
```

```
/
//  Project.swift
//  ProjectDescriptionHelpers
//
//  Created by 김호세 on 2022/05/09.
//

import ProjectDescription
import ProjectDescriptionHelpers

let project = Project(
    name: "RxPackageKit",
    organizationName: "com.jstock",
    packages: [],
    targets: Project.dynamicFramework(
        name: "RxPackageKit",
        frameworkDependencies: [
            .external(name: "RxSwift"),
            .external(name: "RxCocoa"),
            .external(name: "RIBs")
        ],
        testDependencies: []
    ),
    schemes: []
)
```

다음으로 ‘Tuist 디렉토리가 있는’ 디렉토리에서 tuist fetch를 한다.

```
$ tuist fetch
```

![](https://blog.kakaocdn.net/dn/xrHws/btrBLGle4wn/SIKUMuTPcciFSVCFzTzB81/img.png)

빌드에 실패하게 된다. 다음과 같은 오류를 뱉었으면, Tuist/Dependencies/Carthage/Build를 확인하자.

![](https://blog.kakaocdn.net/dn/A8iOt/btrBKwbHofk/jSOA0SKLWhs8yMNVsNcfwk/img.png)

위처럼 Build 폴더에 Rx~.xcframework들이 있으면 성공이다.

---

### Xcode 12? Bug?

이렇지 못한 경우가 있는데 개인 노트북(M1 - XCode 12)으로 작업하는 경우 Carthage Bug(?)가 있어서 Build를 할 수 없다. 이때 Scirpt를 만들어서 carthage를 빌드해줘야 한다.

[https://github.com/Carthage/Carthage/blob/master/Documentation/Xcode12Workaround.md](https://github.com/Carthage/Carthage/blob/master/Documentation/Xcode12Workaround.md) 을 참고해서 carthage.sh를 생성하고 (./는 스크립트 생성한 위치)

```
$ ./carthage.sh build RxSwift Tuist/Dependencies --platform iOS --use-xcframeworks --no-use-binaries --use-netrc --cache-builds --verbose
$ ./carthage.sh build RIBs Tuist/Dependencies --platform iOS --use-xcframeworks --no-use-binaries --use-netrc --cache-builds --verbose
```

를 입력하면 Rx~.xcframework들이 생성된다

---

이제 Tuist/Dependencies/Carthage/Checkouts/RIBs/ios/RIBs.xcodeproj를 열고 RIBs Target을 확인해보면 RxRelay.framework, RxSwfit.framework에 의존성을 가지는데 RxSwift를 우리는 xcframework로 받았기 때문에 이것을 xcframework로 교체한다.(왼쪽 아래에서 add 하면 됨)

![](https://blog.kakaocdn.net/dn/b38Nya/btrBOuwTj1G/uZgenHeGp6646RqN3gt6Ik/img.png)

![](https://blog.kakaocdn.net/dn/Ok2FB/btrBMa0ecow/f2cATHbznLXv4vVF9O6GeK/img.png)

그런 다음에 RIBs를 다시 Build 한다.( Xcode 12인 경우 위처럼 스크립트로 실행해야 함.)

```
$ carthage build RIBs --project-directory Tuist/Dependencies --platform iOS --use-xcframeworks --no-use-binaries --use-netrc --cache-builds --verbose
```

![](https://blog.kakaocdn.net/dn/bBe8s0/btrBJMzw2Zw/agb4OgqaPAntmpqKJMfTaK/img.png)

그러면 빌드에 성공한다. 프로젝트를 생성해주고

```
$tuist generate
```

![](https://blog.kakaocdn.net/dn/c5YQHD/btrBOup9phI/tabJrFaHT48lhlhKcyRRyk/img.png)

다음과 같은 예쁜 모양을 만들 수 있다. 그래프로 출력하게 되면 (-t 옵션은 테스트 타겟들 제외) 그래프가 예쁘게 나온다.

```
$ tuist graph -t
```

![](https://blog.kakaocdn.net/dn/bUUrsq/btrBMg0JE4a/V0Klqw0WIxAoPgJBcjXgyK/img.png)

프로젝트를 열고 App을 실행하면 RxCocoa도 성공적으로 import 할 수 있고, 빌드도 할 수 있게 된다.

### Source

> [http://minsone.github.io/mac/ios/ios-clean-architecture-with-ribs-reactorkit-and-modularization-using-tuist-2](http://minsone.github.io/mac/ios/ios-clean-architecture-with-ribs-reactorkit-and-modularization-using-tuist-2) [http://minsone.github.io/mac/ios/ios-clean-architecture-with-ribs-reactorkit-and-modularization-using-tuist-1](http://minsone.github.io/mac/ios/ios-clean-architecture-with-ribs-reactorkit-and-modularization-using-tuist-1) [https://github.com/minsOne/iOSApplicationTemplate](https://github.com/minsOne/iOSApplicationTemplate) [https://github.com/Carthage/Carthage/blob/master/Documentation/Xcode12Workaround.md](https://github.com/Carthage/Carthage/blob/master/Documentation/Xcode12Workaround.md) [https://okanghoon.medium.com/xcode-프로젝트-관리를-위한-tuist-알아보기-6a92950780be](https://okanghoon.medium.com/xcode-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-%EA%B4%80%EB%A6%AC%EB%A5%BC-%EC%9C%84%ED%95%9C-tuist-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0-6a92950780be)