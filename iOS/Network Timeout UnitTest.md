
서버 통신을 담당하는 NetworkManager에서 Timeout Error를 시뮬레이팅 하는 법은

URLProtocol을 Confirm하고, Error를 Response로 넘겨주는 방식으로 처리하는 것이 간편

```swift

class TimeoutURLProtocol: URLProtocol {
  override class func canInit(with request: URLRequest) -> Bool {
    return true
  }

  override class func canonicalRequest(for request: URLRequest) -> URLRequest {
    return request
  }

  override func startLoading() {
    // Introduce a delay (simulating a timeout)

    DispatchQueue.global().asyncAfter(deadline: .now() + 3.0) {
      // Respond with an error
      self.client?.urlProtocol(self, didFailWithError: NSError(domain: NSURLErrorDomain, code: NSURLErrorTimedOut, userInfo: nil))
      self.client?.urlProtocolDidFinishLoading(self)
    }
  }

  override func stopLoading() {
    // Clean up or additional actions, if needed
  }
}
```

그런 다음에 test code 안에서, URLSessionConfiguration이  protocol을 컨펌함

```swift
func testURLSessionTimeout() {
    let expectation = XCTestExpectation(description: "Time out expectation")
    let config = URLSessionConfiguration.ephemeral

    config.protocolClasses = [TimeoutURLProtocol.self]
    // Timeout Interval, 단위는 seconds
    config.timeoutIntervalForRequest = 3

    let session = URLSession(configuration: config, delegate: nil, delegateQueue: nil)

    let task = session.dataTask(with: url) { data, response, error in
      if let error = error {
        // Time out error
        print("Error: \(error)")

        // Error 발생 기대 충족
        expectation.fulfill()
        return
      }

      guard let data = data else {
        let noDataError = NSError()
        // No Data Error
        return
      }

      // Success, There is Data.
      print(data)
    }
    task.resume()
    wait(for: [expectation], timeout: 4)
  }

```

그러면  TimeoutURLProtocol, config.timeoutIntervalForRequest과 wait()을 통해서 타임아웃 에러를 시뮬레이팅할 수 있음.