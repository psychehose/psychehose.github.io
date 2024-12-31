Build Setting 중 만난 에러프로젝트를 진행하면서 나중에 개발서버와 상용서버를 나눌 필요가 생길 거 같아서 빌드 세팅을 진행했다.

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

확실하진 않지만, 심증으로는 애플에서 디버그 용도는 32bit 아키텍쳐 ( 배포용도는 64bit도 )만을 지원하기 때문인 것 같다. 위에서 말한 것과 같이, 초기 세팅은 ARCHS_STANDARD_64_BIT 였기 때문이다.