
환경 : Apple Silicon (M1), Xcode 14.3


Fat Framework 또는 Universal Framework는 xcframework와 다르다. xcframework가 나오기전에 프레임워크를 사용하는 방법은 .framework를 사용하는 것이었다.


Xcode에서 프로젝트로 framework를 생성하고 Logger.swift를 만들고 간단한 코드 작성

```swift

// Logger.swift

import Foundation

public class LoggerInFramework {

  public static func Logging(_ args: String) -> String {
  
    debugPrint(args)
    
    return args

  }

}


```


그러고나서 **시뮬레이터로** 빌드로 하고 Product - Show Build Folder in Finder를 클릭


![[framework_path.png]]

LoggerFramework.swiftmodule 안에 arm64-apple-ios-simulator가 있는 것을 확인

이제 이 프레임워크를 테스트 해보기 위해 새로운 앱 프로젝트를 하나 만듬

![[hostapp_file.png]]


위처럼 프레임워크를 네비게이터에 추가해주고 Project - Targets에서 Embed를 Embed & Sign으로 변경하기


```swift

// ViewController.swift

import UIKit
import LoggerFramework

class ViewController: UIViewController {

  override func viewDidLoad() {
    super.viewDidLoad()

    LoggerInFramework.Logging("Hello World")

  }  

}


```

Simulator로 빌드 시 성공, 그러나 실제 기기로 빌드 하면 다음과 같은 에러를 뱉음

>Could not find module 'LoggerFramework' for target 'arm64-apple-ios'; found: arm64-apple-ios-simulator, at: /Users/psychehose/Study/iOSLaboratory/HostingLoggingFrameworkApp/HostingLoggingFrameworkApp/LoggerFramework.framework/Modules/LoggerFramework.swiftmodule


swiftmodule 안에 arm64-apple-ios-simulator만 존재하기 때문

그렇다면 swiftmodule 안에 arm64-apple-ios와 x86_64-apple-ios-simulator가 있다면 M1 Mac과 Intel Mac에서 모두 시뮬레이터를 돌리고 실기기에 빌드할 수 있을 것이다.

framework에 저 위의 경우를 다 때려넣어서 만든 것이 Fat Framework이다.

Fat Framework를 만드는 법은 Framework 프로젝트에서 Aggreate 타겟을 만들고 script를 짜서 만드는 것이 가장 간단하다.

![[aggregate.png]]



Product 이름은 FatFramework로 만듬 
타겟에서 FatFramework - Build Phases에서 

1. Target Dependency Framework (LoggerFramework) 추가하기
2. + 버튼 누르고 run script 추가하기


```bash
#!/bin/sh 
UNIVERSAL_OUTPUTFOLDER=${BUILD_DIR}/${CONFIGURATION}-universal 

mkdir -p "${UNIVERSAL_OUTPUTFOLDER}" 

xcodebuild BITCODE_GENERATION_MODE=bitcode OTHER_CFLAGS="-fembed-bitcode" -target "${PROJECT_NAME}" ONLY_ACTIVE_ARCH=NO -configuration ${CONFIGURATION} -sdk iphoneos BUILD_DIR="${BUILD_DIR}" BUILD_ROOT="${BUILD_ROOT}" clean build 

xcodebuild BITCODE_GENERATION_MODE=bitcode OTHER_CFLAGS="-fembed-bitcode" -target "${PROJECT_NAME}" ONLY_ACTIVE_ARCH=NO -configuration ${CONFIGURATION} -sdk iphonesimulator BUILD_DIR="${BUILD_DIR}" BUILD_ROOT="${BUILD_ROOT}" EXCLUDED_ARCHS="arm64"  clean build 

cp -R "${BUILD_DIR}/${CONFIGURATION}-iphoneos/${PROJECT_NAME}.framework" "${UNIVERSAL_OUTPUTFOLDER}/" 

cp -R "${BUILD_DIR}/${CONFIGURATION}-iphonesimulator/${PROJECT_NAME}.framework/Modules/${PROJECT_NAME}.swiftmodule/."

```


그리고 FatFramework Scheme을 편집을 누르고 Run에서 Build Configuration을 Release로 변경하고 기기를 Any iOS Device로 하고 빌드한다

![[sandbox_error.png]]


위의 에러가 발생하면 Build Setting -  User Script Sandboxing을 No로 설정하고 다시 빌드

이제 framework의 swiftmodule을 보면 arm64-apple-ios, x86_64-apple-ios-simulator가 있게 된다.
(arm64-apple-ios-simulator가 없는 이유는 스크립트에서 EXCLUDED_ARCHS를 했기 때문)

![[fatframework_path.png]]


아까 만든 테스트앱에서 프레임워크를 교체하고 실기기를 빌드하면 성공한다. 그리고 intel mac에서도 simulator로 빌드하면 성공할 것임. 근데 Apple Silicon을 사용하는 맥에서 시뮬레이터로 빌드할 수 없다.

swiftmodule에 arm64-apple-ios-simulator를 추가할 수는 있다. 그런데, m1 맥 테스트 앱에서 시뮬레이터로 빌드하면 다음과 같은 에러가 뜬다.

![[simulator_device_error.png]]



왜 이런 이유가 발생할까?

Derived Data를 삭제하고, Aggregate가 아닌 프레임워크를 iOS 디바이스, 시뮬레이터 빌드를 각각 하고 프레임워크 path로 가서 손수 lipo 명령어를 사용하자

```bash
lipo -create -output UniversalFramework.framework/UniversalFramework \
    Debug-iphonesimulator/LoggerFramework.framework/LoggerFramework \
    Debug-iphoneos/LoggerFramework.framework/LoggerFramework

```

그러면 아래와 같은 에러가 발생할 것이다.

>fatal error: /Library/Developer/CommandLineTools/usr/bin/lipo: Debug-iphonesimulator/LoggerFramework.framework/LoggerFramework and Debug-iphoneos/LoggerFramework.framework/LoggerFramework have the same architectures (arm64) and can't be in the same fat output file

정확한 이유는 아니지만, 위의 에러 로그처럼 simulator와 실제 device의 아키텍쳐가 같아서 프레임워크를 호스팅하는 앱(Test App)이 arm64-apple-ios-simulator를 못찾는 것 같다.

그래서 M1 Mac이 등장한 이후로 Fat Framework를 잘 사용하지 않는 것 같다. 대신 대부분의 경우에 xcframework를 사용한다.