

### 개요 - What is @testable

출시할 어플리케이션을 개발할 때 빌드 환경을 구성하는 것이 좋다고 생각합니다. 왜냐하면 추후에 개발 서버를 바라보고 있는 어플리케이션, 실제 서버를 바라보고 있는 어플리케이션을 파일 하나로 관리할 수 있어서요. .xcconfig file에 샥샥샥 하면 손쉽게 관리 가능합니다. 

요즘 저는 테스팅을 공부하고 있습니다. 제가 진행하고 있는 사이드 프로젝트에서도 테스트를 도입하고 싶어서 기존에 있는 기능에 테스트 코드를 작성하려고 했습니다. 유닛테스트 케이스를 생성하고 테스트를 하려고 하는데 예제에서 많이 보던 @testable import 'myProject'를 했는데 다음과 같은 에러가 발생했습니다.

![](https://blog.kakaocdn.net/dn/mbkn5/btr6tX3qHqO/aCEgvC9kkSEfKcWce61a6K/img.png)

먼저 @testable은 뭘까요? 우리가 기본적으로 Unit Test를 하기 위해서는 Unit Test Target을 만들어서 진행하게 됩니다.

우리가 테스트 코드를 작성할 Unit Test Class에서 우리의 App Target에 바로 접근할 수 없어요. 왜냐하면 Target들은 별도의 모듈로 처리 되기 때문입니다. 그래서 import를 해야 합니다. 따라서 역시 internal(아무것도 안 붙이면 자동으로 internal로 선언됨)에 접근할 수가 없어요.

```
import UIKit
	// TestableImport 타겟임!
	// ViewController.swift 
class ViewController: UIViewController {


  private let text: String

  init(text: String) {
    self.text = text
    super.init(nibName: nil, bundle: nil)
  }

  required init?(coder: NSCoder) {
    fatalError("init(coder:) has not been implemented")
  }

  override func viewDidLoad() {
    super.viewDidLoad()

    view.backgroundColor = .red
    guard let bundleID = Bundle.main.bundleIdentifier else {
      return
    }
   }
```

![](https://blog.kakaocdn.net/dn/o3sgZ/btr6qNNFQGh/4MYA16uMOU3f1hxsTM7xJ0/img.png)

### 문제 상황 - 자동 완성 잘됨, 테스트 잘됨, 빨간 줄 안사라짐?

하지만 우리는 소프트웨어 공학적으로, 모든 것을 open과 public으로 선언하지 않잖아요? 그래서 @testable이 필요한 겁니다.

```
@testable import TestableImport
```

이렇게 선언하면 우리의 Main Target에 있는 internal에 접근할 수 있어요.

이제 위에서 언급한 제 프로젝트에서 에러가 발생하는 상황을 설명할게요.

1. Project Configuration 설정  
    - 총 환경 3개를 사용 - Development, Staging, Production  
    - 각각 하나의 앱을 구성할 수 있기에 앱마다 Debug, Release가 있기 때문에 총 3 * 2 = 6개의 Configuration이 구성됨  
    - Debug(Development), Release(Development), etc..
2. 3개의 환경 (Development, Staging, Production)은 xcconfig file로 관리
3. Unit Test Target 생성
4. @testable import 'MainTarget'
5. internal 코드들 자동 완성 가능
6. 테스트 코드 작성 가능 테스트 성공
7. 그런데 위처럼 빨간 줄

테스트 코드 작성은 잘 되고, 테스트도 잘됩니다. 그런데 저렇게 계속 빨갛게 버그처럼 사라지지 않고 있습니다. 혹시 클린빌드를 하면 될까 Xcode를 껐다 켜면 없어질까 싶어서 해봤는데 되지 않았습니다. 그래서, Sample App을 만들어서 버그를 재현해 보도록 할게요.

### 버그 재현 - 빌드환경구성

먼저 App을 만들고 Build 환경을 구성합니다. 여기에서는 Staging을 안 쓰고 4개만 만들도록 할게요.

먼저 xcconfig 파일 두 개(Development, Production)를 만들고 Configuration을 펼쳐서 아래에 있는 +를 클릭하고 Duplicate Debug 한번, Duplicate Release 한번 눌러줍니다. 그리고 이름을 바꿔주세요.

![](https://blog.kakaocdn.net/dn/mbhAV/btr6g8Z4biV/FHgER9ZDkMrAwf9C3oyFIK/img.png)

빌드 설정이 잘 되어 있는 것을 신뢰하기 위해서, xcconfig에 코드를 작성하겠습니다. Bundle Identifier를 설정하는 코드입니다.

```
//
//  Development.xcconfig
//  TestableImport

PRODUCT_BUNDLE_IDENTIFIER = com.dev.hose.TestableImport.Development
```

```
//
//  Production.xcconfig
//  TestableImport

PRODUCT_BUNDLE_IDENTIFIER = com.dev.hose.TestableImport
```

그러고 나서 Project -> Target -> Signing & Capabilities에 가면 우리가 설정한 Configuration 총 4개가 있을 거예요. 

Bundle Identifier를 $(PRODUCT_BUNDLE_IDENTIFIER)로 입력합니다. 그러면 다음과 같이 될 거예요.

![](https://blog.kakaocdn.net/dn/cllnVE/btr6tXClClI/Ur4edaiK4nhLgId8jx1ktK/img.png)

이제 빌드 환경 설정은 거의 다 끝났어요. 이제 하나의 코드로 2개의 앱을 만들 수 있게 할 거예요. Scheme을 이용하면 됩니다.

Scheme 바꾸는 곳 ( 시뮬레이터 바꾸는 위치 왼쪽)을 누르고 Edit Scheme을 클릭하면 아래 왼쪽과 같이 뜰 겁니다.

그러면 왼쪽에 있는 것들 (Run, Test, Profile, Analyze, Archive) 눌러서 아래처럼 빌드 환경에 맞게 설정합니다. 

그런 다음에 Duplicate Sceme을 눌러서 'TargetName'(Development)으로 바꿔주고 이것 역시 빌드 환경을 설정합니다.

![](https://blog.kakaocdn.net/dn/bHVHdV/btr6pGBf201/hUMrxOmcc0nJQjkkcR2JrK/img.png)![](https://blog.kakaocdn.net/dn/D3yW9/btr6pGgYbXJ/UfMLbc3Zhk0NqBlJu8jcBk/img.png)

이렇게 하면 빌드 환경 설정은 끝이 납니다. 아래에서 확인할 수 있듯이 문제없이 잘 돌아가고 있어요.

![](https://blog.kakaocdn.net/dn/buPbNv/btr53IudVAb/BF60BOxkxUvdZ2iquHV0yK/img.png)![](https://blog.kakaocdn.net/dn/BNH8K/btr6eWlb2H4/JZx5R7wt54DPrDeK5ek97K/img.png)

이제 테스트 코드를 작성하는 곳으로 돌아가겠습니다. Unit Test Target라서, 별도의 타깃이기 때문에 @testable 어노테이션을 붙여주겠습니다. 테스트를 하기 위해서 위해서 ViewController에서 private으로 선언한 text를 internal로 선언하겠습니다. 그리고 테스트를 작성할게요. 테스트를 작성하고 테스트 (Command + U)를 하면 성공적으로 테스트할 수 있습니다. 당연하게 internal로 선언된 것들도 모두 접근할 수 있고요. 하지만 아래처럼 빨간 줄은 역시 사라지지 않습니다.

![](https://blog.kakaocdn.net/dn/DApwt/btr6oe6tR1U/Lp4FEw9Z7rw6tVdORcYCWk/img.png)

이상합니다.

TestableImport(Development)의 Test는 아까 Debug(Development)로 설정했고

TestableImprot의 Test도 역시 Debug(Production)으로 설정했습니다. 따라서 저 빨간 줄도 사라져야 된다고 생각합니다. 이 이슈에 대해서 'Module '' was not compiled for testing'으로 많은 시간을 검색해 봤는데 대부분 Enable Testability = YES로 설정하라는 답변이 대부분이었습니다.

![](https://blog.kakaocdn.net/dn/QX71d/btr6oXQMWqL/0qIq3atf7rNJUvPdg97hK0/img.png)

Debug는 디폴트로 Enable Testability가 Yes이기 때문에 의미가 없었습니다.

### 해결법들

stackoverflow, apple developer forum에서 제시한 해결책들을 시도했으나 다 실패했습니다.

그런데 이것저것 시도해 보다가 두 가지 방법을 찾았습니다.

첫 번째 방법은 Productuon에서도 Enable Testability = YES로 설정을 하는 것입니다. 그랬을 때 빨간 줄이 사라졌습니다.

![](https://blog.kakaocdn.net/dn/edVyIB/btr6qOTnJsH/ykkmDtmQ5FK6NhWlED3qjK/img.png)

![](https://blog.kakaocdn.net/dn/bkzvTy/btr6qrjEZSn/WP35oviZ1jvNi1lKM7UYm1/img.png)

Productuon에서도 Enable Testability를 지워주면 다시 빨간 줄이 생깁니다.

두 번째 방법은 Project Configuration에서 Debug를 하나 더 만드는 것입니다.

![](https://blog.kakaocdn.net/dn/cqrFvR/btr53ESUgpQ/2ifuQUTKT7ZSPQRMCqqRp0/img.png)

그리고 끝입니다? 테스트를 진행하면, 빨간 줄이 사라집니다. 그냥 Configuration을 만드는 것만으로 빨간 줄이 사라집니다.

### 자의적인 해석 그리고 찝찝함

> 이 문단은 상상의 나래입니다. 틀릴 확률이 매우 높습니다.

왜 위의 두 가지 방법이 빨간 줄을 없앤 걸까요?

첫 번째 방법은 모든 설정을 Testability가 가능하게 했으니, 컴파일러가 @testable에 대해서 신경 쓰지 않는 것 같습니다.

두 번째 방법은 Debug Configuration이 있는 것만으로 빨간 줄이 사라졌는데 Unit Test Target의 configuration가 debug로 설정되어 있는 것 같습니다. 그래서 컴파일러가 Debug Configuration이 없으면 Testable을 사용할 수 없다고 에러를 띄우는 것 같아요.

Swift Package Manager도 custom configuration을 적용할 수 없는 이슈가 있는데 이와 비슷한 이유이지 않을까 싶습니다. 

혹시 이것에 대해서 해결한 적이 있으시거나 이유를 아시는 분이면 알려주시면 감사하겠습니다. 쓰고 보니까 빌드 환경 구성 글 같아 보이네요.

#### GitHub

[https://github.com/psychehose/TestableImport](https://github.com/psychehose/TestableImport)