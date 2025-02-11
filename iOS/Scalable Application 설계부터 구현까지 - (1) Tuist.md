
확장 가능한 어플리케이션은 모듈을 빠르게 붙이고 뗄 수 있어야 합니다. 이것을 용이하게 해주는 것이 바로 Xcode Project 관리 도구인 Tuist입니다.

#### What is Tuist?

> Tuist is a command line tool (CLI) that aims to facilitate the generation, maintenance, and interaction with Xcode projects. It's distributed as a binary, which you can easily install and use without having to depend on other tools to manage dependencies (like you would do if the tool was written in other programming languages such as Ruby, or Java)  
>   
> Tuist는 Xcode 프로젝트의 생성, 유지보수 및 상호 작용을 용이하게 하는 것을 목표로 하는 CLI(명령줄 도구)입니다. 종속성을 관리하기 위해 다른 도구에 의존하지 않고 쉽게 설치하고 사용할 수 있는 이진 파일로 배포됩니다(Ruby 또는 Java와 같은 다른 프로그래밍 언어로 작성된 경우처럼).  
> .

설치 방법과 간단한 사용 방법은 Tuist 공식 홈페이지를 참고해 주세요.

[https://tuist.io/](https://tuist.io/)

Tuist is a tool that helps developers manage large Xcode projects by leveraging project generation. Moreover, it provides some tools to automate most common tasks, allowing developers to focus on building apps.

tuist.io](https://tuist.io/)

### Tuist를 사용하는 이유

 Tuist는 Xcode 프로젝트를 빠르게 구성하는 걸 도와주는 아주 유용한 툴입니다. 높은 러닝 커브라는 단점이 존재하지만 그것을 무색하게 할 만큼 많은 장점들이 있습니다. 제가 직접 체감한 건 다음과 같아요.

1. Git Conflict이 적습니다. iOS 개발할 때 .xcodeproj에서 생각보다 많이 충돌이 일어나는데 Tuist를 사용하면 이를 현저히 줄여줍니다. 정신 건강에 좋아요.  
      
    
2. 모듈화를 하는데 용이합니다.(Xcode Project에서 SPM을 추가해서 모듈화를 해본 적이 있는데 확실히 Tuist를 이용해서 Project 단위로 모듈화를 하는 게 더 간단한 것 같아요.)  
      
    
3. Swift로 프로젝트 설정이 가능합니다. 비슷한 툴인 XcodeGen은 yaml로 파일을 작성한다고 합니다. yaml을 당연히 학습해야겠죠?  
      
    
4. 구조를 확장할 때 용이합니다. 어플리케이션이 커질수록 설정들도 많아지긴 마련인데 이를 플러그인을 이용해서 가독성 있게 프로젝트를 관리할 수 있습니다.  
      
    
5. 모듈 간의 관계를 나타내는 그래프 기능 덕분에 구조 파악이 용이합니다.

그리고 시간을 들여 Tuist Template를 만들어 놓으면 다음부터 빠르게 어플리케이션 개발을 세팅된 상태에서 할 수 있습니다.

### Tuist 시작하기

아래 명령어를 이용해서, Tuist를 설치합니다. 

```
curl -Ls https://install.tuist.io | bash
```

프로젝트를 생성할 폴더를 생성하고 그 폴더로 이동해서 tuist project를 시작합니다.

```
mkdir tuist-template-for-scalable-app
cd tuist-template-for-scalable-app
tuist init --platform ios
```

tree 명령어를 이용하면 아래와 같이 폴더 및 파일들이 생성된 걸 알 수가 있어요. 

```
tree

.
├── Plugins
│   └── TuistTemplateForScalableApp
│       ├── Package.swift
│       ├── Plugin.swift
│       ├── ProjectDescriptionHelpers
│       │   └── LocalHelper.swift
│       └── Sources
│           └── tuist-my-cli
│               └── main.swift
├── Project.swift
├── Targets
│   ├── TuistTemplateForScalableApp
│   │   ├── Resources
│   │   │   └── LaunchScreen.storyboard
│   │   ├── Sources
│   │   │   └── AppDelegate.swift
│   │   └── Tests
│   │       └── AppTests.swift
│   ├── TuistTemplateForScalableAppKit
│   │   ├── Sources
│   │   │   └── TuistTemplateForScalableAppKit.swift
│   │   └── Tests
│   │       └── TuistTemplateForScalableAppKitTests.swift
│   └── TuistTemplateForScalableAppUI
│       ├── Sources
│       │   └── TuistTemplateForScalableAppUI.swift
│       └── Tests
│           └── TuistTemplateForScalableAppUITests.swift
└── Tuist
    ├── Config.swift
    └── ProjectDescriptionHelpers
        └── Project+Templates.swift
```

```
tuist edit
```

이 디렉토리에서 tuist edit을 입력하면, Xcode 창이 뜨는데요. 이곳에서 우리가 개발할 어플리케이션에 대한 설정을 하는 겁니다. 기능들과 모듈들에 대한 것들은 Manifests Project에서 작성할 것입니다. 이것을 작성하기 전에 기능, 모듈들을 구성하기 쉽게 해주는 Plugins부터 작성하도록 하겠습니다.

### Plugins

플러그인을 이용하면 가독성 있게 쉽게 프로젝트를 관리할 수 있습니다. Plugins를 끝까지 열면 ProjectDescription 폴더가 있는데 이곳에 **swift file을 생성하고 코드를 작성합니다.**

여기에서 작성한 코드들을 실제 모듈을 생성하는 코드를 작성할 때 사용하게 됩니다. 이 코드가 유용한 모듈을 생성하는 함수가 String을 parameter로 많이 필요로 합니다. 이게 모듈이 증가하게 되면 관리하기가 불편해져서, extension을 추가해서 사용하면 아주 편해집니다.

한번 맛보기로 비교를 해볼까요. 왼쪽이 플러그인 사용 전이고, 오른쪽이 후입니다. 사실 왼쪽에 있는 코드도 extension을 이용해서 많이 줄인 것임에도 불구하고 오른쪽에 비하면 예쁘지 않네요. String을 그대로 넣다 보니까 재사용하기도 불편해 보이기도 하고요.

![](https://blog.kakaocdn.net/dn/DBaUZ/btr7gSU98iI/dM9KZsQN5ZAAnhkZXErBSK/img.png)![](https://blog.kakaocdn.net/dn/An4BP/btr7e1STAEj/ZsPgiwRdJKdXagRryfzwY1/img.png)

플러그인 사용 전(왼), 플러그인 사용 후(오)

그러면 플러그인에 어떤 코드를 작성해야 할까요? 많은 Sugar 코드가 있지만 TargetDependency에 관한 extension이 가장 중요하다고 생각합니다. 그도 그럴 것이 Tuist를 이용해서 App을 만든다는 것은 보통 모듈화를 염두에 두었다는 것이고 이는 필연적으로 의존성을 잘 핸들링해야 한다는 것을 의미하게 됩니다.

첫 번째로 할 것은 이름이 너무 긴 TargetDependency를 typealias 걸어서 Dep으로 하겠습니다. 그러면 앞으로 우리는 TargetDependency라는 것 대신에 Dep이라는 이름으로 사용할 수 있게 되겠죠? 그다음으로는 Dep에 대한 Path 설정을 자동 완성으로 쉽게 접근하고 싶기 때문에 Path + Extension을 만들도록 하겠습니다.

```
// Alias.swift

import Foundation
import ProjectDescription

public typealias Dep = TargetDependency

import Foundation
import ProjectDescription


// Path + Extension.swift

public extension ProjectDescription.Path {
  static func relativeToModule(_ pathString: String) -> Self {
    return .relativeToRoot("Projects/Modules/\(pathString)")
  }
  static func relativeToFeature(_ pathString: String) -> Self {
    return .relativeToRoot("Projects/Features/\(pathString)")
  }
  static func relativeToApplication(_ pathString: String) -> Self {
    return .relativeToRoot("Projects/Application/\(pathString)")
  }
  static var app: Self {
    return .relativeToRoot("Projects/Application")
  }
}
```

그리고 Dep(TargetDependency)는 Project가 될 수도 있고, xcframework가 될 수도 있고, target이 될 수도 있고 등등이 될 수 있습니다. 위에서 언급했듯이, Project로 구성할 것이기 때문에 이와 관련된 코드도 작성하도록 하겠습니다.

```
// Dep + Extension.swift

extension Dep {
  static func module(name: String) -> Self {
    return .project(target: name, path: .relativeToModule(name))
  }
  static func feature(name: String) -> Self {
    return .project(target: name, path: .relativeToFeature(name))
  }
}
```

다음으로는 Application이 의존성을 가질 Feature들에 대한 것을 확장을 할거예요. Feature들도 당연히 Dep(TargetDependency)입니다. 위 왼쪽 사진에서 보면 .project(~~~) 이런식으로 들어가는데, 오른쪽은 깔끔하게 . 을 이용해 깔끔하게 접근할 수 있는 걸 볼 수 있어요. 

```
// Dep + Project.swift

import Foundation
import ProjectDescription

extension Dep {
  public struct Project {
    public struct Feature { }
    public struct Module { }
  }
}


public extension Dep.Project.Feature {
  static let Main                                  = Dep.feature(name: "Main")

}

public extension Dep.Project.Module {
  static let Util                                  = Dep.module(name: "Util")
  static let ThirdPartyWrapper                     = Dep.module(name: "ThirdPartyWrapper")
  static let Networkit                             = Dep.module(name: "NetworkKit")
  static let Design                                = Dep.module(name: "Design")
}
```

마지막으로 외부의존성에 관한 확장 코드를 작성할 거에요. Tuist에서 공식적으로 지원하는 외부 의존성은 크게 네 가지예요.

1. Carthage
2. Xcode Swift Package Manager (Package)
3. Tuist Swift Package Manager (External)
4. Framework

SPM이 두 개라서 조금 이상한 것 같기도 합니다.

Carthage와 **Tuist** Swift Package Manager는 Dependencies.swift에서 작성하면 Tuist가 이를 처리해서 프레임워크로 만들어서 사용합니다. **Xcode** Swift Package Manager는 Project에 Package가 바로 들어갑니다. (우리가 평소 사용하는 SPM을 떠올리시면 될 것 같아요.) Tuist SPM을 사용하고 external을 하면 그래프에 안 그려지는 이슈가 있어서 저는 package를 이용하도록 하겠습니다. Carthage나 local xcframework를 사용하는 경우에도 같은 방식으로 확장해 주시면 되겠습니다.

```
// Dep+SPM.swift

import Foundation
import ProjectDescription

extension Dep {
  public struct SwiftPM { }
}


public extension Dep.SwiftPM {
  static let Alamofire             = Dep.package(product: "Alamofire")
}

public extension Package {
  static let Alamofire             = Package.package(url: "https://github.com/Alamofire/Alamofire.git", from: "5.6.4")
}
```

여기까지가 유용한 기본적인 플러그인입니다. 이제 모듈을 생성하는 코드를 작성해 보도록 합시다.

### Manifests

 기억이 정확한 건 아닌데, 객체지향프로그래밍 수업을 들을 때 교수님께서 manifest를 언급하셨던 것 같습니다. A를 맞기 위해 6~7년 전 자바를 열심히 했던 기억이 있네요.  매니페스트는 무엇일까요?

> 매니페스트 파일(manifest file)은   
> 컴퓨팅에서 집합의 일부 또는 논리 정연한 단위인 파일들의 그룹을 위한 메타데이터를 포함하는 파일  
> 이다. 예를 들어, 컴퓨터 프로그램의 파일들은 이름, 버전 번호, 라이선스, 프로그램의 구성 파일들을 가질 수 있다. - 위키백과  
>   

Manifests에 파일을 생성하는 것은 모듈에 관한 메타데이터를 작성하는 것입니다. tuist generate를 통해서 실제로 생성하는 것이고요.

그러면 Manifest file을 작성합시다. 

Tuist에서 manifest file은 무조건 Project.swift라는 이름으로 생성해야 해요. 그래서 모듈을 하나 추가한다고 하면 폴더를 하나 만들고 그 안에 Project.swift를 만들면 되는 것입니다. 

그전에 file을 하나 추가하도록 할게요. 위치는 Mainfests Project 안에 있는 Tuist/ProjectDescriptionHelper에 생성하면 됩니다. 이 파일은 Project에 기본값을 추가해서 반복적인 코드를 좀 줄여줄 수 있어요. 쭉 읽어보면 어떤 내용인 지 감 잡으실 수 있을 것 같아요.

```
// Project+Templates.swift

import ProjectDescription
import MyPlugin

public extension Project {
  static func project(name: String,
                      organizationName: String = "YourOrganizationName",
                      options: Options = .options(),
                      packages: [Package] = [],
                      product: Product,
                      platform: Platform = .iOS,
                      deploymentTarget: DeploymentTarget? = .iOS(targetVersion: "15.0", devices: .iphone),
                      dependencies: [Dep] = [],
                      infoPlist: [String: InfoPlist.Value] = [:],
                      sources: SourceFilesList = ["Sources/**"],
                      resources: ResourceFileElements = ["Resources/**"],
                      scriptAction: [TargetScript] = [],
                      resourceSynthesizers: [ResourceSynthesizer] = []) -> Project {

    let settings: Settings = Settings.settings()


    let bundleID = product == .app
    ? "com.YourAppName"
    : "com.\(organizationName).\(name)"



    let isEnableResource = (product == .app || product == .framework)

    let target = Target(name: name,
                        platform: platform,
                        product: product,
                        bundleId: bundleID,
                        deploymentTarget: deploymentTarget,
                        infoPlist: .extendingDefault(with: infoPlist),
                        sources: sources,
                        resources: isEnableResource ? resources : [],
                        scripts: scriptAction,
                        dependencies: dependencies)


    let testTargetDependencies: [Dep] = [.target(name: "\(name)")]

    let testTarget = Target(name: "\(name)Tests",
                            platform: platform,
                            product: .unitTests,
                            bundleId: "\(bundleID)Tests",
                            deploymentTarget: deploymentTarget,
                            infoPlist: .default,
                            sources: "\(name)Tests/**",
                            dependencies: testTargetDependencies)


    let targets: [Target] = [target, testTarget]


    return Project(name: name,
                   organizationName: organizationName,
                   options: options,
                   packages: packages,
                   settings: settings,
                   targets: targets,
                   resourceSynthesizers: resourceSynthesizers)
  }
}
```

이제 본격적으로 진짜로 Manifest file(Project.swift)를 작성해 봐요. Manifest 폴더 아래에 Projects라는 폴더를 생성할게요. 이곳에 모듈에 관한 Project.swift 들을 만들어줄 거예요. 그리고 Projects 폴더 안에 Application 폴더를 만들고 그 하위에 Project.swift 파일을 만들겠습니다. 다음처럼 되겠습니다.

![](https://blog.kakaocdn.net/dn/Jq0wH/btr7gy3Kjpl/7LRPMwWPXYkhAY7NAzjnak/img.png)

```
// Project.swift (Application 폴더)

import Foundation
import ProjectDescription
import ProjectDescriptionHelpers
import MyPlugin

let project = Project.project(
  name: "Application",
  product: .app,
  dependencies: [
    Dep.Project.Feature.Main
  ]
)
```

이렇게 하면 모듈 하나의 메타데이터를 작성한 것이고 tuist generate를 하게 되면 모듈 하나를 생성할 수 있게 됩니다. 여기까지 오는데 플러그인도 만들고 힘들었지만 그 대가로 앞으로 엄청 편하게 모듈을 확장할 수 있게 되었어요.  같은 방식으로 Main 기능 모듈을 만들겠습니다.

```
// Project.swift (Projects/Features/Main 안에)

import Foundation
import ProjectDescription
import ProjectDescriptionHelpers
import MyPlugin

let project = Project.project(
  name: "Main",
  product: .framework,
  dependencies: [
  ]
)
```

이번에는 product가 .framework 입니다. 이것은 개발에 따라 달라지는 거라 적절히 선택해 주시면 되겠습니다. 그러면 Project.swift 파일에 메타데이터들을 다 입력했으니 generate를 하면 되겠네요. 그전에. xcodeproj를 한 번에 관리하고 싶으니 xcworkspace부터 만들어주도록 하겠습니다. tuist edit을 해서 Manifests 폴더 아래에 Projects, Tuist 폴더와 같은 레벨로 Workspace.swift를 생성해 주시면 됩니다.

```
import Foundation
import ProjectDescription

let workspace = Workspace(
    name: "Application",
    projects: [
      "Projects/Application"
    ],
    generationOptions: .options(
      enableAutomaticXcodeSchemes: false,
      autogeneratedWorkspaceSchemes: .disabled,
      lastXcodeUpgradeCheck: nil,
      renderMarkdownReadme: false
    )
)
```

그러고 나서 이제 어플리케이션을 생성해 보도록 할게요. 'tuist generate'를 입력하면

```
The target Application has the following invalid source files globs:
- The directory "/tuist-template-for-scalable-app/Projects/Application/Sources" defined in the glob pattern "/tuist-template-for-scalable-app/Projects/Application/Sources/**" does not exist.
Consider creating an issue using the following link: https://github.com/tuist/tuist/issues/new/choose
```

이런 에러가 발생할 거예요. 왜냐하면 모듈을 생성할 때, 소스 파일들이 필요하고 그것들의 위치를 정했는데 해당 Path에 소스파일들이 없어서입니다. 해당 Path로 가서 파일들을 붙여 넣기 하면 해결되는 문제입니다. 다만 모듈이 엄청 많다고 가정하면 일일이 붙여넣기 하는 게 귀찮을 수가 있어요. 그래서 tuist가 제공하는 scaffold 기능을 이용하면 됩니다. scaffold 기능을 이용하면 템플릿을 만들어놓고 터미널 명령어로 인자를 받아서 해당 path에 원하는 파일들을 생성할 수가 있습니다.

#### tuist scaffold

먼저 템플릿부터 만들어야 합니다. Tuist 폴더 아래에 Templates 폴더를 만들고 그 아래에 어떤 단어로 명령어를 만들지 정해서 .swift 파일을 만들면 되겠습니다. 저는 dummy라는 명령어로 만들겠습니다. tuist scaffold dummy -name

```
// Templates/dummy/dummy.swift
import ProjectDescription

let nameAttribute: Template.Attribute = .required("name")

let template = Template(
  description: "Custom template",
  attributes: [
    nameAttribute
  ],
  items: [
    .file(
      path: "Projects/Features/\(nameAttribute)/Sources/dummy.swift",
      templatePath: "dummy.stencil"
    ),
    
    .file(
      path: "Projects/Features/\(nameAttribute)/Resources/dummy.swift",
      templatePath: "dummy.stencil"
    ),
    .file(
      path: "Projects/Features/\(nameAttribute)/\(nameAttribute)Tests/\(nameAttribute)Tests.swift",
      templatePath: "Tests.stencil"
    ),

  ]
)
```

Attribute는 인자값 설정입니다. required는 필수로 입력해줘야 하고, optional은 말 그대로 해도 되고 안 해도 되는데, 여기에서는 required로 하나만 사용하겠습니다. items에 생성할 file들의 path, 어떤 template를 사용할지 해당 stencil file의 Path를 작성합니다.  {{ name }}이 아까 인자로 받은 그 값을 여기에 넣어주겠다는 것을 의미해요.

> * 단! stencil file에 어떤 것이라도 입력해야 scaffold 할 때 생성되는 것 같아요. 

```
import XCTest

final class {{ name }}Tests: XCTestCase {

    override func setUpWithError() throws {
        // Put setup code here. This method is called before the invocation of each test method in the class.
    }

    override func tearDownWithError() throws {
        // Put teardown code here. This method is called after the invocation of each test method in the class.
    }

    func testExample() throws {
        // This is an example of a functional test case.
        // Use XCTAssert and related functions to verify your tests produce the correct results.
        // Any test you write for XCTest can be annotated as throws and async.
        // Mark your test throws to produce an unexpected failure when your test encounters an uncaught error.
        // Mark your test async to allow awaiting for asynchronous code to complete. Check the results with assertions afterwards.
    }

    func testPerformanceExample() throws {
        // This is an example of a performance test case.
        self.measure {
            // Put the code you want to measure the time of here.
        }
    }

}
```

이제 scaffold를 위한 작업을 끝마쳤으니 파일들을 생성해 보도록 할게요.

```
tuist scaffold dummy -name Main
```

![](https://blog.kakaocdn.net/dn/370tB/btr7hyCUiWz/H8kgsejIf3Ztky0jNTMkzk/img.png)

다음과 같이 잘 생성되었어요. Application도 비슷한 방식으로 만들어주겠습니다. 이번에는 인자를 넣지 않았어요.

```
// Template/app/app.swift


import ProjectDescription

let template = Template(
  description: "Custom template",
  attributes: [
  ],
  items: [
    .file(
      path: "Projects/Application/Sources/AppDelegate.swift",
      templatePath: "AppDelegate.stencil"
    ),
    .file(
      path: "Projects/Application/Resources/dummy.swift",
      templatePath: "../dummy/dummy.stencil"
    ),
    .file(
      path: "Projects/Application/ApplicationTests/ApplicationTests.swift",
      templatePath: "AppDelegateTest.stencil"
    ),

  ]
)
// Template/app/AppDelegate.stencil

import UIKIt
@main
class AppDelegate: UIResponder, UIApplicationDelegate {

  var window: UIWindow?

  func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey : Any]? = nil
  ) -> Bool {

    let vc = UIViewController()

    let window = UIWindow(frame: UIScreen.main.bounds)
    window.rootViewController = vc
    window.makeKeyAndVisible()
    self.window = window

    return true
  }
}

// Template/app/AppDelegateTests.stencil

import XCTest

final class AppDelegateTests: XCTestCase {

    override func setUpWithError() throws {
        // Put setup code here. This method is called before the invocation of each test method in the class.
    }

    override func tearDownWithError() throws {
        // Put teardown code here. This method is called after the invocation of each test method in the class.
    }

    func testExample() throws {
        // This is an example of a functional test case.
        // Use XCTAssert and related functions to verify your tests produce the correct results.
        // Any test you write for XCTest can be annotated as throws and async.
        // Mark your test throws to produce an unexpected failure when your test encounters an uncaught error.
        // Mark your test async to allow awaiting for asynchronous code to complete. Check the results with assertions afterwards.
    }

    func testPerformanceExample() throws {
        // This is an example of a performance test case.
        self.measure {
            // Put the code you want to measure the time of here.
        }
    }

}
```

```
$ tuist scaffold app
```

![](https://blog.kakaocdn.net/dn/xfbnE/btr7C2JhRaS/eqQZlpsYTkp8W7WclmKlXk/img.png)

잘 생성되었네요. 이제 정말로 프로젝트들을 생성할 수 있어요.

```
$ tuist generate
```

![](https://blog.kakaocdn.net/dn/cFwBVJ/btr7s4OwCrR/cZy3vU0LGv5iZeYmMiab4K/img.png)

프로젝트가 제대로 만들어졌고, 이제 이것을 바탕으로 Scalable Application을 설계하고 개발할 수 있게 되었습니다.

정말 많은 것들을 했는데 한번 잘 만들어놓으면 다른 개발을 할 때 빠르게 시작할 수 있을 것 같습니다. 최대한 상세히 적다 보니 포스팅이 꽤 길어진 것 같아요. 저도 제 지식을 점검할 수 있어서 좋았고 scaffold 기능을 좀 더 잘 이용해 봐야겠다는 생각이 듭니다. 

#### GitHub

[https://github.com/psychehose/tuist-template-for-scalable-app](https://github.com/psychehose/tuist-template-for-scalable-app)