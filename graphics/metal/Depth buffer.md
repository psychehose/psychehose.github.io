
3D 공간에서 렌더링을 할 때 여러 오브젝트가 겹쳐 있을 때, 어떤 것이 앞에 있고 뒤에 있는지 판단이 필요하다.

깊이 버퍼를 사용하기 위해서 렌더링이 되는 metalView의 깊이 버퍼 정밀도를 설정해야한다.

```swift
// 깊이 버퍼의 정밀도를 32bit 부동 소수점 사용한다. (높은 정밀도)
// 깊이 값 범위은 0.0(가까움) ~ 1.0 (멈)
metalView.depthStencilPixelFormat = .depth32Float
```


그런 다음에 깊이 테스트 규칙을 정의 해야 한다. 겹쳐 있을 때 판단할 근거가 필요하기 때문이다. 이때 사용 하는 것이 **MTLDepthStencilState** 이다. 이 객체를 통해서 어떤 조건에서 픽셀을 그릴지 결정한다. 즉 깊이 테스트 규칙을 정의한다.

다음은 MTLDepthStencilState를 생성하는 방법이다.

```swift
let descriptor = MTLDepthStencilDescriptor() // 깊이 테스트 규칙 설정 템플릿
descriptor.depthCompareFunction = .less // 더 가까운 것 우선시
descriptor.isDepthWriteEnabled = true // 픽셀 그릴 때 깊이 값 업데이트
return device.makeDepthStencilState(descriptor: descriptor)
```

아래는 depthCompareFunction의 규칙 enum이다.

```swift
.never        // 절대 그리지 않음
.less         // 더 가까우면 그림 (일반적)
.equal        // 같은 깊이면 그림
.lessEqual    // 가깝거나 같으면 그림
.greater      // 더 멀면 그림 (역순)
.notEqual     // 다른 깊이면 그림
.greaterEqual // 멀거나 같으면 그림
.always       // 항상 그림 (깊이 무시)
```
