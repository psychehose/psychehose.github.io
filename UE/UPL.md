
### UPL이란?

UPL은 언리얼 엔진에서 플랫폼별 네이티브 코드와 설정을 통합하기 위한 XML 기반의 스크립팅 언어입니다. 주로 Android와 iOS 플랫폼에서 사용되며, 빌드 과정에서 플랫폼별 설정을 자동으로 적용할 수 있게 해줍니다.


---

### when to use

1. 네이티브 라이브러리 통합시 (jar, aar, framework)
2. 플랫폼별 권한 및 설정
	*  Android Manifest
	*  iOS info.plist

---


### 주요 태그


* Android
	* `proguardAdditions` - Proguard 난독화 규칙 추가
	* `androidManifestUpdates` - 권한, 액티비티, 서비스, 리시버 등 추가 / 수정
	*  `buildscriptGradleAdditions` - 플러그인 클래스패스, 저장소
	*  `buildGradleAdditions` - 앱 레벨 build.gradle 파일을 수정
	*  `gameActivityImportAdditions` - GameActivity.java 파일에 import 구문을 추가
	*  `gradleProperties` -  gradle.properties 파일을 수정,  주로 빌드 설정, 메모리 옵션, 프로젝트 전역 변수 설정

* iOS
	* `iosPListUpdates` - info.plist 값 업데이트
