
작년부터 진행했던 사이드 프로젝트가 있습니다. 최근에 버전 2를 만들기 위해서 기획 & 디자이너 친구들이 앱 업데이트 요청을 했어요. 그래서 과거에 짰던 코드를 보면서 Dev 설정으로 애플리케이션을 살펴보고 있었습니다. 근데 카카오톡으로 로그인하기를 눌렀는데 권한요청 alert 떴어요. 여기에 타겟이름이 그대로 있다는 것을 발견했습니다.

![](https://blog.kakaocdn.net/dn/dpQuzW/btslZ5NKkqr/n9zrpFvuy6sENHXX9QE0UK/img.png)

이 문제를 해결하는 방법은 **빌드 환경을 단일 타겟과 스킴으로 나누지 않았다면** PRODUCT_NAME을 수정하면 됩니다.

근데 저희 팀이 작업했던 빌드 환경은 달라서 언급을 먼저 하겠습니다.

애플리케이션의 **Configuration**은 Debug(Development), Debug(Staging), Debug(Production), Release(Development), Release(Staging), Release(Production)로 구성되어 있어요.

그리고 **단일 Target**에, **세가지 Scheme**(Dev, Staging, Prod)로 구성되어 있고 xcconfig 파일을 이용해 Build Setting 값을 넣었습니다. xcconfig 파일에서 CFBundleDisplayName를 설정했기 때문에 단일 타겟으로 다른 이름으로 세 가지 앱을 빌드할 수 있었어요. (각각 Hous-, Hous-Dev, Hous-Staging)

그래서, 앱을 처음 실행하자마자 나오는 푸시알림 권한요청 팝업, 그리고 애플로 로그인하기 팝업은 CFBundleDisplayName을 CFBundleDisplayName을 바라보기 때문에 해당 값이 잘 뜹니다.

![](https://blog.kakaocdn.net/dn/bhUESq/btsl0jrs0xY/rUbDYOsHMKawDmWPiW8AgK/img.png)

그러나, 카카오톡으로 로그인하기는 CFBundleDisplayName을 바라보는 것이 아니었고, PRODUCT_NAME(=CFBundleName)이 뜨는 상태였습니다. 처음에 이걸 봤을 때, 마음 편하게 Build Setting 값을 확인했고 ProductName = $(TARGET_NAME)이었기 때문에 각 xcconfig에서 TARGET_NAME을 환경에 맞게 수정했었어요. 결과는 성공적이었고 이렇게 해결된 줄 알았습니다.

문제는 단위테스트에서 발생했어요. Command + U를 눌렀는데 세상에서 가장 보기 싫은 에러가 떴습니다.

![](https://blog.kakaocdn.net/dn/c8zoME/btsl1WPOda7/0raQoWTD67KqOF4HVsgnOk/img.png)

stackoverflow에 나오는 여러가지 해결 방법을 시도해 봤는데 대부분 안되었지만, 어떤 글에서 지나가듯이 import 하는 모듈의 이름이 PRODUCT_MODULE_NAME이라는 것을 알게 되었습니다. 그래서, PRODUCT_MODULE_NAME을 플래그 봤는데 ($PROUDCT_NAME:c99extidentifier)였습니다. 이것을 보고, 테스트 클래스로 가서 자동완성은 안되지만, PRODUCT_MODULE_NAME을 그대로 @testable import를 했습니다. 그리고 Command + U를 눌렀는데

![](https://blog.kakaocdn.net/dn/TjEGd/btsl0Xn8JPf/JuuQyn8AT9LNWnXkMwIzJ1/img.png)

놀랍게도 No such module 에러는 계속 떴지만, 테스트는 진행 되었어요.ㅋㅋ 하지만 자동완성은 안되었어요. 근데 여기서 조금 더 발전해서 PRODUCT_MODULE_NAME을 xcconfig로 다루면 해결할 수 있지 않을까 싶어서 시도해 봤습니다. 그전에 문제점을 다시 정리하고 가겠습니다.

1. 빌드환경 구성을 단일타겟과 세 가지 스킴, xcconfig로 했다.
2. 카카오톡 로그인 Alert에 뜨는 앱이름을 바꾸기 위해서는 PRODUCT_NAME을 고쳐야 한다.
3. 따라서 PRODUCT_MODULE_NAME(이 플래그는 PRODUCT_NAME을 참조하고 있음)이 변경되어서 기존의 Target NAME을 import 할 수 없다.

#### 해결방법

xcconfig에서 PRODUCT_MODULE_NAME_FIX 라는 플래그를 선언하고 적당한 값(이게 모듈 이름이 될 거예요)을 넣었어요. 그리고 Build Setting으로 가서 PRODUCT_MODULE_NAME에 PRODUCT_MODULE_NAME_FIX를 넣었습니다.

![](https://blog.kakaocdn.net/dn/bAaFdy/btslZF9Womg/mkM2xJM0tHL3ICyazbzWQK/img.png)

그리고 위에서 적당한 값을 넣었잖아요? @testable import를 해줍시다. 그리고 Command + U를 누르면 테스트가 진행되네요. 

![](https://blog.kakaocdn.net/dn/ET2As/btslZSOHOL6/XlcRMxk0xt7DTw0TuAjFDk/img.png)

#### 번외

왜 카카오 로그인의 Alert는 CFBundleDisplay가 아닌 CFBundleName을 보고 있을까? 혹시 카카오SDK에서 이것을 다루는 프로퍼티가 있을까 싶어서 카카오 SDK 코드 구경을 했었습니다. 

```
public func presentationAnchor(for session: ASWebAuthenticationSession) -> ASPresentationAnchor
```

사용자가 카카오로 로그인을 누를 때 위의 함수를 호출(애플도 같은 함수를 호출하지만 파라미터가 다름) 하게 되는데 파라미터가 ASWebAuthenticationSession입니다. 이 부분에서 버그가 있다고 다른 레포에서 이슈가 올라왔었습니다.  ~~사실 문제는 아닌 거 같고 그냥 업데이트를 안 하고 있는 것 같아요.~~

아무튼 끝. 읽어주셔서 감사합니다.