## What is Unit Test?

유닛 테스트는 테스트 케이스를 작성하는 절차로써 특정 모듈이 정확히 작동하는지 검증하는 절차입니다.

## What is advantages of Unit Test?

1. 문제점을 발견하기 쉽다  
    -> 프로그램을 독립적인 작은 단위로 쪼개서 검사하기 때문에 잘못된 점을 빠르게 발견할 수 있다. → 디버깅 시간의 단축
2. 변경이 쉽다.  
    -> 코드 리팩토링시에 유용하다. 리팩토링 후에도 의도대로 작동하고 있음을 유닛 테스트를 통해 확신할 수 있다.
3. 통합이 쉽다.  
    -> 유닛 테스트는 유닛 자체의 불확실성을 제거해 준다. 그렇기 때문에 유닛들의 통합들을 검증하는 Integration Test에서 유용하다.

### Unit Test 시 좋은 습관

Testing 할 때, 가장 좋은 관행(Practice)은 FIRST이다.

1. **Fast:** 테스트들은 빠르게 실행되어야만 한다.
2. **Independent / Isolated:** 테스트들은 서로 state를 공유해서는 안된다. (독립적)
3. **Repeatable:** 검사를 할 때마다 동일한 결과를 얻어야 한다.
4. **Self-validating:** 테스트들은 automated 여야 한다. 결과는 "Pass or Fail" 이여야 한다.
5. **Timely:** 이상적으로 Production Code를 만들기 전에, Test Code부터 작성해야 한다.(TDD)

---

## TDD (Test Driven Development)란?

TDD는 테스트 주도 개발이라는 뜻으로, 테스트를 먼저 작성하고 그것이 통과하는 코드를 작성하는 개발 방법론입니다. TDD는 다음과 같은 기본적인 원칙에 따라 개발됩니다.

![](https://blog.kakaocdn.net/dn/SLAXa/btr3XzeOz6m/jHDqIOdvfH0lcWCpgrOKBK/img.png)

1. 자동화된 테스트 케이스를 작성한다.
2. < RED >: 테스트를 실행하면 실패하는 것을 확인한 후 코드를 작성한다.
3. < GREEN >: 작성한 코드를 테스트를 통과시킨다.
4. < REFATOR>: 리팩토링을 통해 중복 코드를 제거하고 더 나은 코드를 만든다.

**장점**

1. 코드의 품질을 높일 수 있고, 버그를 줄일 수 있다
2. 기능 추가나 수정을 할 때 기존 코드에 영향을 미치지 않도록 하며 코드의 유지보수가 용이해진다.

**단점**

1. 초기에 작성하는 테스트 케이스와 코드의 품질이 좋지 않을 경우, 전체적인 개발 프로세스가 더욱 지연될 수 있다.
2. 프로그램 개발 전에, 테스트 코드부터 작성하므로 전체적인 생산성이 저하할 수 있다

## iOS Programming에서 Testing을 적용하는 방법

아이디 유효성 검증을 하는 Validator를 구현해야 한다고 합시다. 먼저 **요구사항**을 살펴보고 Validator를 구현하도록 합시다.

> 요구사항: 아이디는 반드시 알파벳 소문자로 시작해야 하며, 알파벳 소문자와 숫자를 포함하여 5자 이상 12자 이하로 한다.

Validator를 쉽게 테스트하기 위해서 객체로 만듭니다. 그리고 유효성 검증 패턴을 정규 표현식으로 작성해 주고 이에 어긋날 시 false, 만족하면 true를 리턴하는 함수를 만듭니다. 우리는 이 Validator 객체를 신뢰하기 위해서 함수를 테스트하면 됩니다.

```swift
// Validator.swift

import Foundation

public struct Validator {
  public init() { }
  private let idValidationPattern: String = "^[a-z]{1}[a-z0-9]{4,11}$"
}

public extension Validator {
  func idValidateByClient(_ input: String) -> Bool {
    guard
      let regex = try? NSRegularExpression(
        pattern: idValidationPattern,
        options: []
      ) else {
      assertionFailure("Regex not valid")
      return false
    }
    let regexFirstMatch = regex
      .firstMatch(
        in: input,
        options: [],
        range: NSRange(location: 0, length: input.count)
      )
    return regexFirstMatch != nil
  }
}
```

Project에서 command + 6을 누르고 왼쪽 하단에 + 버튼을 클릭해서 유닛 테스트를 추가합니다.

테스트하고 싶은 모듈을 @testable Import 하고 setUpWithError에서 테스트하고 싶은 객체를 선언해 줍니다.

```swift
import XCTest
@testable import Validator

final class ValidatorTests: XCTestCase {

  var sut: Validator!

  override func setUpWithError() throws {
    try super.setUpWithError()
    sut = Validator()
  }

  override func tearDownWithError() throws {
    sut = nil
    try super.tearDownWithError()
  }
}
```

TDD 방법론에 따라서 먼저 실패하는 케이스를 적용해 보면 되겠습니다. 어떤 케이스가 실패를 할까요? TDD의 핵심은 바로 이런 테스트 시나리오를 짜는 것이라고 생각합니다.

테스트 시나리오를 쉽게 생각하기 위해서는 **Given**, **When**, **Then**으로 나눠서 생각하는 것이 좋다고 합니다.

다음과 같은 테스트 케이스(**Given**)는 테스트시에(**When**) 실패(**Then**)해야만 합니다.

- 1psychehose → 숫자로 시작함
- _psychehose → 특수문자로 시작함
- Psychehose → 대문자로 시작함
- 김psychehose → 한글로 시작함
- psychehose! → 특수문자 포함
- a → 1글자
- abcd → 4글자
- abcde12345678 → 13글자

테스트 함수 코드는 **무조건 앞에 test**가 붙어야 합니다. 그리고 테스트 결과를 알기 위해 **XCTAssertEqual** 함수를 이용할 건데, 사용하기 정말 쉽습니다. 매개변수 1과 매개변수 2를 비교해서 같으면 성공, 다르면 실패합니다. 따라서 실패 케이스를 작성할 때 **XCTAssertEqual(result, true)**는 실패해야 합니다. 그러면 위의 실패하는 테스트 케이스를 바탕으로 코드를 작성해 보겠습니다.

```swift
import XCTest
@testable import Validator

final class ValidatorTests: XCTestCase {

  var sut: Validator!

  override func setUpWithError() throws {
    try super.setUpWithError()
    sut = Validator()
  }

  override func tearDownWithError() throws {
    sut = nil
    try super.tearDownWithError()
  }

  func testFailFirstNumberID() throws {
		// Given
    let id = "1psychese"

		// When
    let result = sut.idValidateByClient(id)

		// Then
    XCTAssertEqual(result, true)
  }

  func testFailFirstSpecialSymbolID() throws {
    let id = "_psychehose"

    let result = sut.idValidateByClient(id)
    XCTAssertEqual(result, true)
  }

  func testFailFirstCaptialID() throws {
    let id = "Psychehose"
    let result = sut.idValidateByClient(id)

    XCTAssertEqual(result, true)
  }

  func testFailFirstKoreanID() throws {
    let id = "김psychehose"
    let result = sut.idValidateByClient(id)

    XCTAssertEqual(result, true)
  }

  func testFailOneLetterID() throws {
    let id = "p"
    let result = sut.idValidateByClient(id)

    XCTAssertEqual(result, true)
  }

  func testFailFourLetterLetterID() {
    let id = "abcd"
    let result = sut.idValidateByClient(id)

    XCTAssertEqual(result, true)
  }
  func testFailThirteenLetterID() throws {
    let id = "abcde12345678"
    let result = sut.idValidateByClient(id)
    XCTAssertEqual(result, true)
  }

  func testFailIDWithSymbol() throws {
    let id = "ab!cdef"
    let result = sut.idValidateByClient(id)
    XCTAssertEqual(result, true)
  }
}
```

![](https://blog.kakaocdn.net/dn/8lEPE/btr3WaTHqhQ/P30kT6HEIA3m5KweKrzusk/img.png)

원하는 대로 실패하는 결과값이 잘 나온 것 같습니다. 만약 여기에서, 원하는 결과값이 나오지 않는다면 코드 리팩토링을 진행하면 되겠습니다. 그러면 이제 통과하는 케이스를 바탕으로 코드를 작성하겠습니다.

다음과 같은 테스트 케이스(**Given**)는 테스트시에(**When**) 성공(**Then**)해야만 합니다.

- abcde
- psychehose
- abcdeabcdeab
- abcd1234
- a1b2c3d4

```swift
import XCTest
@testable import Validator

final class ValidatorTests: XCTestCase {


		// ...
		// ...

  func testSuccessFiveLetterID() throws {
    let id = "abcde"
    let result = sut.idValidateByClient(id)

    XCTAssertEqual(result, true)
  }
  func testSuccessID() throws {
    let id = "psychehose"
    let result = sut.idValidateByClient(id)

    XCTAssertEqual(result, true)
  }

  func testSuccessTwelveLetterID() throws {
    let id = "abcdeabcdeab"
    let result = sut.idValidateByClient(id)

    XCTAssertEqual(result, true)
  }

  func testSuccessIDWithNumber1() throws {
    let id = "abcd1234"
    let result = sut.idValidateByClient(id)

    XCTAssertEqual(result, true)
  }
  func testSuccessIDWithNumber2() throws {
    let id = "a1b2c3d4"
    let result = sut.idValidateByClient(id)

    XCTAssertEqual(result, true)
  }
}
```

![](https://blog.kakaocdn.net/dn/bqktVJ/btr35EePQWY/sOPslnmvnaKImxBXk1xLv0/img.png)

이번에도 원하는 대로 성공하는 결괏값이 잘 나온 것 같습니다. 만약 여기에서 실패한다면 코드 리팩토링을 해야만 합니다. 그리고 다시 실패 케이스 단계로 돌아가야만 합니다. 이 과정을 반복하면서 개발을 완료하도록 합니다.

### (Optional) 비동기 처리 테스트와 고찰

> 주의) 이 단락은 저의 생각을 많이 담고 있습니다. 정답이 아니고 제 개인적인 생각이니 주의해서 읽어주세요. 틀린 점이 있거나 자신의 생각과 다른 점이 있으면 알려주시면 감사하겠습니다.

비동기처리의 대표적인 예는 서버 통신이 있습니다. 사용자가 회원가입을 하는 상황을 생각해 봅시다. 아이디란에 아이디를 입력하고 위에서 작성한 아이디 유효성 검사를 무사히 통과했습니다. 이게 끝이 아닌 걸 압니다. 앱은 이 아이디를 서버에 보내고 서버는 데이터베이스에서 조회해 아이디 중복 검사를 합니다. 우리가 중복검사를 하고자 하는 아이디를 서버에 보내고 그 결과가 올 때까지 우리는 기다려야 합니다. 즉 우리는 비동기 함수를 작성해야 합니다. 그러면 비동기 함수는 어떻게 테스트하면 될까요? 

사실 이 부분을 넣을까 고민했었습니다. 'Validator의 역할(책임)을 어디까지로 잡을 것인가' 대해 생각해 봤을 때 의문이 들었습니다.

1. 위의 상황처럼 Validator에서 서버 통신을 하고 백엔드에서 최종적으로 유효성 검사(아이디 중복여부, 아이디 형식)를 해서 회원가입 여부를 Response 값으로 던져준다. 즉 Validator는 **네트워크 의존성**을 가지고 서버 통신 테스트를 작성해야 함
2. Validator는 로컬에서 아이디 형식에 대한 유효성 검사만 한다. - 이 경우에 위의 상황 테스트는 Validator와 Network를 같이 알고 있는 상위 모듈이나 ViewModel, Interactor 같은 곳에서 테스트하게 됨.

![](https://blog.kakaocdn.net/dn/boF5CY/btr5AAwdUlO/sfnmCVkgUo1ZVbZCHjKaG0/img.png)![](https://blog.kakaocdn.net/dn/cVpKcQ/btr5AOHJmHa/wNgkM1gMHkOhEugnQKpJ80/img.png)

왼쪽: 1번 경우, 오른쪽: 2번 경우

따라서 어떤 '**구조**'를 가지고 있느냐가 어떤 **테스트 코드**를 작성할지 **결정하는** 것 같습니다.

저는 네트워크 모듈을 따로 만들어서 사용하고 Validator는 어떠한 의존성도 가지지 않게 개발했습니다. 그렇기에 Validator에서 비동기를 처리하는 함수를 만들지 않았습니다. 

다만 이 포스팅은 유닛테스트 전반적인 것에 대해서 쓰고 싶기 때문에, 1번 경우로 가정(회원가입 여부: Boolean을 받음)을 하고 테스트 코드를 작성하겠습니다.

서버 통신 테스트 방법은 두 가지가 있어요. 

1. 앱에서 만든 가짜 서버와 통신하는 것 - Mock Test
2. 실제 서버와 통신하는 것 - Slow Test

실제 작업을 할 때,  프로젝트 시작 단계에서 기획 문서 하나만 있는 경우가 있을 수도 있습니다. 이때 Mock을 만들어서 서버 통신을 테스트할 수 있습니다. 먼저 Validator에 코드 추가를 하겠습니다.

### Validator에 서버 통신 코드 추가하기

Response 결과는 [Bool] type으로 받겠습니다. ~~(key 이름 짓기 귀찮아서)~~

``` swift
// Validator.swift
import Foundation

public struct Validator {
  public init(configuration: URLSessionConfiguration = .default) {
    self.configuration = configuration
  }

  private let configuration: URLSessionConfiguration
  private let idValidationPattern: String = "^[a-z]{1}[a-z0-9]{4,11}$"

  private var session: URLSession {
    return URLSession(configuration: self.configuration)
  }
}

public extension Validator {
  func isValidationFromServer(_ url: URL) async throws -> Bool {
    let url = url
    let urlRequest = URLRequest(url: url)
    let (data, _) = try await session.data(for: urlRequest)
    
    guard let isValidate = try JSONDecoder().decode([Bool].self, from: data).first else {
      throw NSError(domain: "No Data", code: -1)
    }

    return isValidate
  }
}
```

Validator에서 서버 통신을 할 것이기 때문에, Validator가 URLSession을 가지고 있어야 해요. 이 때 Session Configuration을 주입받을 수 있게 해 주세요.

이제 **isValidationFromServer** 함수를 한번 살펴볼게요. 다른 함수와 다르게 async 키워드가 매개변수 오른쪽에 붙어있는 것을 확인할 수 있습니다. 이것은 비동기 함수라는 뜻입니다.

let (data, _) = try **await** session.data()를 보면 await 키워드를 확인할 수 있어요. await는 무엇일까요?  애플이 구현해놓은 session.data() 함수를 한번 확인하면 알 수 있습니다.

``` swift
 public func data(
 for request: URLRequest,
 delegate: URLSessionTaskDelegate? = nil
 ) async throws -> (Data, URLResponse)

    /// Convenience method to load data using an URL, creates and resumes an URLSessionDataTask internally.
    ///
    /// - Parameter url: The URL for which to load data.
    /// - Parameter delegate: Task-specific delegate.
    /// - Returns: Data and response.
```

awai는 문자 그대로  '기다린다'라는 뜻입니다. 그러면 무엇을 기다릴까요? 애플이 구현해놓은 .data(for:)에서 **async** 키워드가 보이시나요? await는 async를 기다린다고 할 수 있겠네요.

### Mock Unit Test를 위해 MockURLProtocol 생성 및 테스트 코드 작성

Mock으로 Test를 진행해야 하는데요. 이것은 Validator를 생성할 때 주입하는 **URLSessionConfiguration**의 property인 **protocolClasses** 값을 설정하면 값을 하이재킹 할 수 있습니다. MockURLProtocol을 구현하도록 하겠습니다. 

``` swift
// MockURLProtocol.swift

import Foundation
typealias CustomResponse = Result<Data, Error>

class MockURLProtocol: URLProtocol {
  static var responseHandler: ((URLRequest) throws -> (response: HTTPURLResponse, data: CustomResponse))?
  override class func canInit(with request: URLRequest) -> Bool {
    return true
  }
  override class func canonicalRequest(for request: URLRequest) -> URLRequest {
    return request
  }

  override func startLoading() {
    guard let handler = MockURLProtocol.responseHandler else {
      return
    }
    do {
      let result = try handler(request)
      let httpResponse = result.response
      let customResponse = result.data
      switch customResponse {

      case .success(let data):
        client?.urlProtocol(self, didReceive: httpResponse, cacheStoragePolicy: .notAllowed)
        client?.urlProtocol(self, didLoad: data)
        client?.urlProtocolDidFinishLoading(self)

      case .failure(let error):
        client?.urlProtocol(self, didFailWithError: error)
      }
    }
    catch {
      client?.urlProtocol(self, didFailWithError: error)
    }
  }
  override func stopLoading() {
    debugPrint("Stop Loading")
  }
}
```

여기까지 구현하면 드디어 Mock으로 Unit Test를 할 수 있게 됩니다.

유닛테스트 파일을 생성하고 테스트 코드를 작성하면 되겠습니다. 저는 ValidatorMockTest.swift로 파일을 만들게요.

``` swift
import Foundation
import XCTest
@testable import Validator

final class ValidatorMockTest: XCTestCase {

  var sut: Validator!

  override func setUpWithError() throws {
    try super.setUpWithError()
    let config = URLSessionConfiguration.ephemeral
    config.protocolClasses = [MockURLProtocol.self]
    sut = Validator(configuration: config)

  }

  override func tearDownWithError() throws {
    sut = nil
    try super.tearDownWithError()
  }

  func test_available_from_mock() async throws {

    // Mock을 생성 -> 이것이 우리가 받을 결과임
    MockURLProtocol.responseHandler = { request in
      let response = HTTPURLResponse(
        url: request.url!,
        statusCode: 200,
        httpVersion: nil,
        headerFields: nil
      )!

      let mockResult: [Bool] = [true]
      let mockResponseData = try! JSONEncoder().encode(mockResult)

      let responseData = CustomResponse.success(mockResponseData)

      return (response, responseData)
    }
//     URL에 "https://" 이런식으로 말이 되는 URL 넣으면 됩니다. scheme host 다 있어야함

    let testTargetValue = try await sut.isValidationFromServer(URL(string: "https://www.test11111.com")!)
    XCTAssertEqual(testTargetValue, true)
  }

  func test_no_available_from_mock() async throws {

    MockURLProtocol.responseHandler = { request in
      let response = HTTPURLResponse(
        url: request.url!,
        statusCode: 200,
        httpVersion: nil,
        headerFields: nil
      )!

      let mockResult: [Bool] = [false]
      let mockResponseData = try! JSONEncoder().encode(mockResult)

      let responseData = CustomResponse.success(mockResponseData)

      return (response, responseData)
    }

    let testTargetValue = try await sut.isValidationFromServer(URL(string: "https://www.test11111.com")!)
    XCTAssertEqual(testTargetValue, false)
  }

  func test_empty_error_from_mock() async throws {

    MockURLProtocol.responseHandler = { request in
      let response = HTTPURLResponse(
        url: request.url!,
        statusCode: 404,
        httpVersion: nil,
        headerFields: nil
      )!

      let mockResult: [Bool] = []
      let mockResponseData = try! JSONEncoder().encode(mockResult)
      let responseData = CustomResponse.failure(NSError(domain: "No data. Empty", code: -10))

      return (response, responseData)
    }

    let testTargetValue = try await sut.isValidationFromServer(URL(string: "https://www.test11111.com")!)
    XCTAssertEqual(testTargetValue, false)
  }
}
```

이제 코드 보시면 완전히 이해가 갈 것에요. 주의할 점은 setupWithError에서 Validator를 생성할 때 Configuration의 protocolClassese에 **[MockURLProtocol.self]**를 꼭 넣어줘야 합니다. 그래야 값을 Mock Response 값을 받을 수 있어요.

![](https://blog.kakaocdn.net/dn/dY99TU/btr5N3StqIK/mklb1GFX0duc6c1T3qR9Ek/img.png)

### API 테스트 코드 작성

마지막으로 실제 서버로 하는 테스트 코드를 작성해 볼게요. 자신이 가지고 있는 실제 서버가 없으면 Postman을 통해서 Mock Server를 손쉽게 만들 수 있습니다. 'postman mock server'로 검색하면 다양한 포스팅을 확인할 수 있을 거예요.

![](https://blog.kakaocdn.net/dn/6BdrF/btr5N3LLvql/fXd4uG18FlgqUnRlOOSXK1/img.png)

저는 총 세 가지 경우를 테스트할 거예요.

(실제 API 테스트이기 때문에 config의 protocolClassess 넣는 코드를 주석처리 하거나 삭제해야 합니다. 그리고 아래 코드에서 url에 자신의 서버 URL을 입력하시면 됩니다.)

1.  회원가입 가능: endpoint - available, 결과 - [true]
2.  회원가입 불가능: endpoint - noavailable, 결과 - [false]
3.  에러: endpoint - empty, 결과 - []

``` swift
import Foundation
import XCTest
@testable import Validator

final class ValidatorSlowTest: XCTestCase {

  var sut: Validator!

  override func setUpWithError() throws {
    try super.setUpWithError()
    let config = URLSessionConfiguration.ephemeral
    sut = Validator(configuration: config)

  }

  override func tearDownWithError() throws {
    sut = nil
    try super.tearDownWithError()
  }

  func test_available_api_test() async throws{

    guard
      let url = URL(
        string: ""
      )
    else {
      return
    }

    let result = try await sut.isValidationFromServer(url)

    XCTAssertEqual(result, true)
  }
  func test_no_available_api_test() async throws{

    guard
      let url = URL(
        string: ""
      )
    else {
      return
    }

    let result = try await sut.isValidationFromServer(url)

    XCTAssertEqual(result, false)
  }
  func test_empty_api_test() async throws{

    guard
      let url = URL(
        string: ""
      )
    else {
      return
    }

    let result = try await sut.isValidationFromServer(url)

    XCTAssertEqual(result, true)
  }
}
```

![](https://blog.kakaocdn.net/dn/b7vDo2/btr5PbWC4Zu/3igzX1RKFfgQUyGKAjRgxk/img.png)

#### 고찰

 Unit Test에 대해 전반적으로 알아봤습니다. GitHub을 구경하거나, 회사 공고를 보면 대부분 MVVM, VIPER(RIBs) 패턴을 사용하고 있는 것 같습니다. 그렇기 때문에 저것들을 공부했었는데요. 왜 사용하냐고 물어보면 로직이 분리되어 있어서 코드 관리가 용이하다, 테스트가 용이하다 같은 말들을 하곤 했습니다. 그러다가 실제로 테스트 코드를 작성해야 할 일이 생겼는데 '테스트 코드를 대체 어떻게 작성해야 하지?' 같은 생각이 들어 멘탈이 나간 적이 있어요. 그때 중요한 걸 놓쳤다는 생각이 들었습니다. 이번 포스팅을 통해서, 테스트 코드와 조금 친해진 거 같아서 꽤나 뿌듯합니다. 테스트와 베스트 프렌드가 되고 싶기 때문에 조만간 MVVM 테스트에 대해 공부하고 포스팅해보고 싶습니다.

[https://github.com/psychehose/UnitTestForValidator](https://github.com/psychehose/UnitTestForValidator)


####   
Reference

[https://developer.apple.com/videos/play/wwdc2018/417/](https://developer.apple.com/videos/play/wwdc2018/417/)

[https://swiftsenpai.com/swift/async-await-network-requests/](https://swiftsenpai.com/swift/async-await-network-requests/)

[https://www.swiftbysundell.com/articles/unit-testing-code-that-uses-async-await/](https://www.swiftbysundell.com/articles/unit-testing-code-that-uses-async-await/)

[https://xtring-dev.tistory.com/entry/Postman-Postman-Mock-Server를-구축하기-Front-end-선개발하기협업](https://xtring-dev.tistory.com/entry/Postman-Postman-Mock-Server%EB%A5%BC-%EA%B5%AC%EC%B6%95%ED%95%98%EA%B8%B0-Front-end-%EC%84%A0%EA%B0%9C%EB%B0%9C%ED%95%98%EA%B8%B0%ED%98%91%EC%97%85)[https://www.kodeco.com/21020457-ios-unit-testing-and-ui-testing-tutorial](https://www.kodeco.com/21020457-ios-unit-testing-and-ui-testing-tutorial)