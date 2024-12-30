

Project - Target - General - Deployment Info에서 iPhone Orientation을 체크 / 체크해제를 할 수 있다.

이것을 체크 하게 되면 동일한 효과를 얻는다.


```swift

// AppDelegate.swift

func application(
	_ application: UIApplication,
	supportedInterfaceOrientationsFor window: UIWindow?
	) -> UIInterfaceOrientationMask {
	return // 체크한 항목들
  }


```


supportedInterfaceOrientations를 지정하면 아이폰에서 '자동 화면 회전'을 설정한 경우 아이폰을 어떻게 잡고 있느냐에 따라서 해당하는 InterfaceOrientation으로 화면이 회전 된다.


### Situation

 LandscapeRight로 고정을 했는데 SFSafariViewController에서 애플로그인을 하고 돌아온 다음에 화면이 Portrait이 되는 경우가 있다고 함.

### Task

 AppDelegate에 접근해서 InterfaceOrientation을 LandscapeRight로 다시 변경하면 된다.

### Action

```swift

	// AppDelegate.swift
import UIKit
@main

class AppDelegate: UIResponder, UIApplicationDelegate {
  @objc
  var allList: UIInterfaceOrientationMask = .portrait
 
  func application(
  _ application: UIApplication,
   supportedInterfaceOrientationsFor window: UIWindow?
   ) -> UIInterfaceOrientationMask {
   
    return allList

  }

}

```

Swift

```swift

  private func rotate() {
    let windowScene = UIApplication.shared.connectedScenes.first as? UIWindowScene
    (UIApplication.shared.delegate as! AppDelegate).allList = .landscapeRight

  

    if #available(iOS 16.0, *) {
      debugPrint("above 16.0")

      windowScene?.requestGeometryUpdate(.iOS(interfaceOrientations: .landscapeRight))

      self.setNeedsUpdateOfSupportedInterfaceOrientations()
    } else {
      debugPrint("below 16.0")
      let value = UIDeviceOrientation.landscapeRight.rawValue

      UIDevice.current.setValue(value, forKey: "orientation")
    }
  }
```

Objective-C

```objc
- (void)rotate:(UIButton *)button {

  AppDelegate *appDelegate = (AppDelegate *)[[UIApplication sharedApplication] delegate];

  

  UIInterfaceOrientationMask e = UIInterfaceOrientationMaskLandscapeRight;

  appDelegate.allList = e;

  

  NSNumber *value = [NSNumber numberWithInt:UIInterfaceOrientationLandscapeRight];
  [[UIDevice currentDevice] setValue:value forKey:@"orientation"];

}
```


### Result

샘플 프로젝트에서 SFSafariViewController에서 돌아올 때 화면 전환이 되는 버그를 재현할 순 없었지만 화면 전환은 위의 코드를 사용 했을 때 쉽게 바꿀 수 있었다. 

주의해야할 점은 AppDelegate에 있는 **supportedInterfaceOrientationsFor**를 먼저 바꿔주고 setValue를 하거나 화면 업데이트를 해줘야 한다는 점이다.

그리고 추가적으로 UIInterfaceOrientation과 UIInterfaceOrientationMask 차이점이다.

> **UIInterfaceOrientation** is Constants that specify the orientation of the app's user interface.

UIInterfaceOrientation는 enum value이고 App의 user interface라고 한다.

> **UIInterfaceOrientationMask** is Constants that specify a view controller’s supported interface orientations.

UIInterfaceOrientationMask ViewController의 interface orientation이라고 한다.

UIInterfaceOrientation은 처음 앱의 interface orientation이고 최상위 뷰컨트롤러인 UIWindowScene의 interface orientation이다. 

UIViewController에서 UIInterfaceOrientation인 interfaceOrientation은 deprecated여서  사용할 수 없고 아래처럼 windowScene에서 접근할 수 있다. read-only value라 바꿀 순 없다.

```swift
let windowScene = UIApplication.shared.connectedScenes.first as? UIWindowScene


windowScene.interfaceOrientation

```

이러한 interfaceOrientation을 바꾸기 위해서 사용하는게 바로 UIInterfaceOrientationMask다. 
UIInterfaceOrientationMask는 바꿀 방향을 표시하는 bitmask다. SDK에게 어떤 방향으로 바꿀거에요. 라고 알려줄 때 사용하는 프로퍼티다. 그리고 화면 전환을 하면 된다.

여담으로 이건 버그인 것 같은데, AppDelegate에서 supportedInterfaceOrientationsFor에서 portrait를 제외하면 추후 화면전환을 사용할 수 없다. 
