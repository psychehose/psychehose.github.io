
```bash
cd {EngineSource 디렉토리}/Engine/Source/ThirdParty/PhysX3/Lib/IOS
nm libPxFoundation.a

```


![[symbol_result.png]]

커스텀한 엔진 심볼 확인 불가 -> 커스텀한 PhysX lib .a overwrite 해야함.

6. PhysX 빌드

	 CMake Version: 3.28.0 (minimum: 3.0)
	 XCode: 14.1
	 MacOS: Ventura 13.3.1(a)
	 Machine: Apple M1 Ultra


Environment - Add Entry

Name: GW_DEPS_ROOT
Value: {PhysX Path} - 주의사항: PhysX3.4 아님 PxShared도 가지고 있는 Root 폴더

![[cmake_setting.png]]


Configure - Generate - Open Project
(Configure 할 때, output 폴더에 CMakeCache 있으면 지울 것 캐싱되어서 안바뀔 수도 있음)

Xcode 프로젝트가 열렸으면 타겟 All_BUILD로 변경 Edit 스킴을 눌러서 release, debug 각 config에 맞는 걸로 바꾼 후 실행하면 .a들 뽑힘. (PhysX3.4와 PxShared 라이브러리들)

![[change_scheme.png]]


해당 폴더로 가서 libPxFoundationDEBUG.a 

```bash
$ nm libPxFoundationDEBUG.a
```

![[symbol_find_result.png]]


뽑은 라이브러리들(PhysX3.4, PxShared) 엔진소스/ThirdParty/PhysX3/Lib/IOS로 가서 덮어쓰기