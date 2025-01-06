SPM으로 RxSwift를 가져오고 Unit Test를 작성한 후 테스트를 실행했을 때 다음과 같은 에러를 만난 적이 있습니다.  
  

```
Missing required module 'RxCocoaRuntime'
```

### ****문제 상황 재현

프로젝트를 만들고 Swift Package Manager로 RxSwift를 추가하고 RxSwift, RxCocoa, RxRelay 라이브러리만을 앱 타겟에 추가합니다. 그리고 테스트 타겟은 Target Dependencies로 앱 타겟을 가지고 있고, 어떠한 라이브러리도 링킹하고 있지 않습니다. 

![](https://blog.kakaocdn.net/dn/RdgNd/btr8uTeqxjZ/YOZ0YRoqBNvxqE6hKVB9gK/img.png)

![](https://blog.kakaocdn.net/dn/cwsx5k/btr8vHqZp1v/ZgdvWLkIyayepWxb2MLli1/img.png)

  
그리고 AppDelegate에서 RxSwift를 import하고 RxSwift 코드를 작성합니다. 이제 애플리케이션을 빌드하면 빌드가 성공합니다.  
  

```
import RxSwift
import UIKit

@main
class AppDelegate: UIResponder, UIApplicationDelegate {

  func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
  ) -> Bool {

    Observable.just(1).subscribe(onNext: {
      print($0)
    })
    .dispose()
    return true
  }
```

이제 UnitTest 클래스로 가서, Command + U를 눌러 테스트를 실행해보도록 할까요? 문제없이 테스트가 성공할 것입니다.  
그러면 import RxCocoa를 시도해보면 어떨까요? 그 순간 아래와 같이 에러가 발생할 거예요. 

![](https://blog.kakaocdn.net/dn/wKYQe/btr8u3BgSJE/9mYuSIp7sq7TNKCKlZscKK/img.png)

우리가 설정한 걸로는 아직 테스트 타겟에서 RxCocoa를 import 하지 못합니다. 그렇기 때문에 테스트 클래스에서 RxCocoa를 지우겠습니다. 그러면 위의 에러는 당연히 사라질 거예요. 다시 AppDelegate로 돌아가서 RxCocoa를 import 하고 AppDelegate에서 RxCocoa가 가지고 있는 Trait인 BehaviorRelay를 작성하도록 할게요.

```
import RxCocoa
import RxSwift
import UIKit

@main
class AppDelegate: UIResponder, UIApplicationDelegate {

  func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
  ) -> Bool {

    Observable.just(1).subscribe(onNext: {
      print($0)
    })
    .dispose()

    BehaviorRelay(value: 2)
      .subscribe(onNext: {
        print($0)
    })
      .dispose()

    return true
  }
```

애플리케이션을 빌드하면 실행이 되고 print도 잘 찍힙니다. 이걸로 봐서 앱 타겟에는 문제가 없습니다. 다시 테스트 클래스로 돌아가서 테스트를 실행해 볼까요? 

![](https://blog.kakaocdn.net/dn/C9D03/btr8lmBPXiT/efZVLhCFNcoQjVsYeSoTp0/img.png)

테스트 클래스에서 RxCocoa를 import 하지 않았는데도 RxCocoaRuntime Error가 발생했습니다.

### 왜 이런 에러가 발생할까? 

테스트 타겟에서 Target Dependency로 앱 타겟을 가지고 있기 때문입니다. 그런데 앱 타겟에서 RxCocoa를 사용하고 있어요. 문제점은 여기에서 발생하는 것 같습니다. 테스트를 실행하면, 테스트 타겟에 RxCocoaRuntime의 코드가 링킹 되지 않는 것 같아요. (근본적인 문제는 아직 잘 모르겠습니다. RxCommunity에서도 오랫동안 제기 되어온 문제이고 SPM 이슈라고 합니다) 그래서 위의 에러가 발생하는 것 같습니다. 그렇다면 테스트 타겟에 RxCocoaRuntime를 링킹 하면 문제가 해결될 것 같아요. 

> Project -> Test Target 클릭 -> Build Phrase -> Link Binary with Libraries -> RxCocoa 넣기.

![](https://blog.kakaocdn.net/dn/pox84/btr8uRnuyqS/qFYBbQf8c5W7blnOoAclsK/img.png)

![](https://blog.kakaocdn.net/dn/snY2F/btr8uSs8KUd/XwFDicPwSOwQn663zZTHA0/img.png)

예상대로 에러는 사라지고 테스트는 성공했습니다. 다만 엄청나게 많은 경고들이 생성되었는데 코드가 중복으로 올라갔다는 경고입니다. 테스트 타겟에서 RxCocoa 코드가 올라오고 테스트 타겟이 Target Dependency로 가지고 있는 앱 타겟에서 RxCocoa 코드가 다시 한번 올라오기 때문입니다. 이제는 이 중복 경고를 지우려고 노력해 보겠습니다.

RxCocoa 구현 코드에 보면, Runtime이라는 폴더가 존재합니다. 우리가 필요한 것은 Runtime에 있는 것들이니까 Link Binary with Libraries에 저것들을 넣어주면 될 것 같습니다. 이 폴더는 그대로 RxTest에 존재합니다. 그러면 테스트 타겟에 RxTest와, RxBlocking을 넣어주면 에러가 사라지지 않을까요?  
  

![](https://blog.kakaocdn.net/dn/LhJkD/btr8lN7y6mF/2ENFkbrfG0lRyqMtvlL611/img.png)![](https://blog.kakaocdn.net/dn/cKdACd/btr8vtNoZgD/qf6HkEldHPDPwmjwXLK12K/img.png)

![](https://blog.kakaocdn.net/dn/kSoQd/btr8wR770kI/kHzTCf6oRbxGoNvMPhVTD1/img.png)

아쉽게도 에러가 사라지지 않았습니다. 논리적으로 사라질 거라고 생각했는데 그러지 않네요. 이러한 이슈는 위에 언급한 것처럼 오래전부터 있어 왔습니다. 물론 해결 방법은 있습니다. 해결 방법도 꽤나 다양합니다. 무튼 해결 방법은 있는데 근본적으로 어떤 문제가 있는지는 언급되지는 않네요. SPM 이슈일 것이라고만 나와있습니다. 그 말에 충분히 신빙성이 있는 게 같은 환경에서 .xcframework로 링킹 하면 해당 에러는 발생하지가 않습니다. 예상 가능한 건 RxCocoaRuntime이 중요 포인트다 라는 것 정도인 것 같습니다. 구현된 RxCocoa를 살펴보면 SPM인 경우에 import RxCocoaRuntime을 하는 걸 볼 수 있기 때문입니다. 

![](https://blog.kakaocdn.net/dn/89GV2/btr8LLUimFH/rjYwZmkK4zW57rakC1hXG0/img.png)

###   Missing required module 'RxCocoaRuntime' 해결방법 총정리

RxSwift Issue와 구글 검색을 통해서 몇 가지 해결 방법을 찾았습니다. 근데 다수의 해결 방법들은 코드가 중복 적재되어서 경고가 뜹니다.

참고해 주세요.  
  

 1. 테스트 타겟에 Link Binary with Libraries에 RxCocoa를 추가한다. 중복 적재 경고(O)  
위에서 사용했던 방법입니다.

> Project -> Test Target 클릭 -> Build Phrase -> Link Binary with Libraries -> RxCocoa 넣기

  
2.  **TEST_HOST를 바꿔주는 방법**입니다. 중복 적재 (X) 
```
# Test Target -> Build Setting -> TEST_HOST 검색 -> '$(BUNDLE_EXECUTABLE_FOLDER_PATH)/' 지우기  

$(BUILT_PRODUCTS_DIR)/<YOUR APPNAME>.app/<YOUR APPNAME>
```


이걸로 추측하는 건데, Bundle Loader 부분이 조금 다르게 처리되어 있는 듯합니다. 이상하게 앱 타겟에서는 Bundle Loader 부분이 설정이 안되어 있고, 테스트 타겟에서는 Bundle Loader 값이 $(TEST_HOST) 입니다. 그래서 TEST_HOST 값을 설정해 주면 RxCocoaRuntime을 가지고 올 수 있는 것이 아닐지 예상합니다.


![](https://blog.kakaocdn.net/dn/b1epQr/btr8Ima3ymS/GGWPfB0bGI15LIqGfmld2k/img.png)

  
  
3. 전처리 사용해서 해결 - 중복적재 (O)

Other Swift Flags는 Swift 컴파일러에게 pass 하라고 명령하는 것입니다. 이걸 입력하지 않을 경우 SPM에 구현되어 있는 RxCocoa에서 import RxCocoaRuntime 할 때 import를 못했으나, Other Swift Flags를 설정하면 컴파일하기 전에 코드를 올리기 때문에 에러가 발생하지 않는 것 같습니다. 테스트 타겟에 RxTest와 RxBlocking을 추가하면 코드 중복 적재 경고가 뜨는데요. 위의 이유 때문입니다. C언어에서 `#include`와 똑같은 것입니다.

> Test Target -> Build Setting -> Other Swift Flags 검색 -> -Xcc -fmodule-map-file=$(PROJECT_TEMP_ROOT)/GeneratedModuleMaps-$(PLATFORM_NAME)/RxCocoaRuntime.modulemap

![](https://blog.kakaocdn.net/dn/rK1Pi/btr8Jo621JB/5cfUXLTnju2vW2zbnLgZK0/img.png)

  
4.  앱 타겟과 테스트 타겟에서 RxCocoa 대신에 RxCocoa-Dynamic을 사용한다- 중복 적재 (X)

 문제로 돌아가서 단순하게 그냥 해결하겠습니다. 우리가 달성하고 싶은 건 다음과 같습니다.

-  테스트 타겟은 RxCocoaRuntime이 필요함 (-> RxCocoa를 Link Binary wtih Libraries에 넣음)
- 그러면 코드가 중복 적재되는데 이를 해결하고 싶음 (-> dynamic을 사용하면 됨)

보통 SPM은 static library(.o type)으로 코드를 가져오는데, 코드 자체를 Heap 영역에 때려 넣습니다. 따라서 코드 자체 말고 Dynamic library reference를 Heap 영역에 넣어 필요할 때마다 Stack 영역에서 가져오는 dynamic으로 변경 시에 중복 적재 경고가 뜨지 않습니다.  
  

![](https://blog.kakaocdn.net/dn/bLcSx2/btr8IJcErfK/n0F7LhRlFUIgfNsgVCDK1K/img.png)![](https://blog.kakaocdn.net/dn/CWUS5/btr8KHkMfIj/g3AxUDBhU3MycwdBI2B2Jk/img.png)

  
5. Local SPM을 만들어서 ThirdPartyLibrary를 관리하는 ThirdPartyManager를 만드는 것입니다. - 중복적재 (X)

![](https://blog.kakaocdn.net/dn/caj5Qa/btr8J4m6q6o/qhgCquDO1aqVGYm1Jkw3Ok/img.png)

그래프를 그리면 다음과 같이 되겠습니다. Local SPM 코드를 다음과 같이 작성할 수 있어요.

```
// swift-tools-version: 5.7
// The swift-tools-version declares the minimum version of Swift required to build this package.

// Package.swift

import PackageDescription

let package = Package(
  name: "ThirdPartyManager",
  platforms: [.iOS(.v14)],
  products: [
    .library(
      name: "ThirdPartyManager",
      type: .dynamic,
      targets: ["ThirdPartyManager"]
    ),
    .library(
      name: "TestResolver",
      targets: ["TestResolver"]
    )
  ],
  dependencies: [
    .package(url: "https://github.com/ReactiveX/RxSwift.git", exact: "6.5.0")
  ],
  targets: [
    .target(
      name: "ThirdPartyManager",
      dependencies: [
        .product(name: "RxSwift", package: "Rxswift"),
        .product(name: "RxCocoa", package: "Rxswift")
      ]
    ),
    .target(
      name: "TestResolver",
      dependencies: [
        .product(name: "RxTest", package: "Rxswift"),
        .product(name: "RxBlocking", package: "Rxswift")
      ]
    ),
    .testTarget(
      name: "ThirdPartyManagerTests",
      dependencies: ["ThirdPartyManager"]),
  ]
)
```

Local SPM으로 RxSwift를 가져온 후에, target을 만들어줍니다. 이것을 library로 외부에 노출하면 되겠습니다. 이때, ThirdPartyManager(library)는 dynamic library로 지정하겠습니다. 그리고 앱을 실행해 보고, 테스트를 실행하면 무사히 통과하고 중복 적재 경고도 없음을 알 수 있습니다. 코드 중복은 dynamic을 적절히 사용하면 해결할 수 있겠네요. Local SPM을 사용해서 에러를 해결하는 코드는 아래 GitHub을 참고해 주세요.

![](https://blog.kakaocdn.net/dn/btgkkT/btr8MpRhc9s/xN9BkJxxkyCrB2DMc75m21/img.png)![](https://blog.kakaocdn.net/dn/NA8tG/btr8I1qNiyC/CCn3uUazzNLgSmfSnStWOk/img.png)

#### 생각

SPM 사용 시에 RxCocoaRuntime 모듈을 찾을 수 없는 에러에 대해서 이제 다양한 방법으로 해결할 수 있게 되었습니다. 처음에 왜 뜨는지 조차 이유를 몰랐었는데, 어느 정도 감을 잡은 후에는 즐겁게 해결할 수 있었어요. 라이브러리에 이슈에 대해서 공부할 때마다 Mach -O Type의 중요성을 깨닫는 것 같습니다.

#### GitHub

[https://github.com/psychehose/SPMRxCocoaRuntimeError](https://github.com/psychehose/SPMRxCocoaRuntimeError)

####  Reference

[https://github.com/apple/swift-package-manager/issues/4581](https://github.com/apple/swift-package-manager/issues/4581)

[https://github.com/ReactiveX/RxSwift/issues/2127](https://github.com/ReactiveX/RxSwift/issues/2127)

[https://stackoverflow.com/questions/58125428/missing-required-module-xyz-on-unit-tests-when-using-swift-package-manager](https://stackoverflow.com/questions/58125428/missing-required-module-xyz-on-unit-tests-when-using-swift-package-manager)

[https://forums.swift.org/t/missing-required-modules-when-importing-an-spm-framework/24856](https://forums.swift.org/t/missing-required-modules-when-importing-an-spm-framework/24856)

[https://github.com/ReactiveX/RxSwift/issues/2057](https://github.com/ReactiveX/RxSwift/issues/2057)

