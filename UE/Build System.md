
> Source 폴더 -> Unreal Build Tool (C#) -> 각 플랫폼 컴파일러 실행


실제 Build를 하는 것은 C#으로 진행함 그래서 닷넷을 설치하는 것


Source 폴더 구조

Source
	- Module 폴더 (보통 Project 이름)
		  - 소스코드(.h, .cpp)
		  - {모듈이름}.Build.cs  
	- 타겟 설정 파일: 전체 솔루션이 다룰 빌드 대상 지정
	> {프로젝트 이름}.Target.cs: 게임 빌드 설정
	> {프로젝트 이름}Editor.Target.cs: 에디터 빌드 설정

Build.cs는 모듈마다 들어가는 모듈 설정 파일


모듈.cpp, 모듈.cpp로 지정

매크로를 이용해서 모듈의 뼈대를 제작
- IMPLEMENT_MODULE: 일반 모듈
- IMPLEMENT_GAME_MODULE: 게임 모듈
- IMPLEMENT_PRIMARY_GAME_MODULE: 주 게임 모듈
  

### 모듈의 공개와 참조

외부로 공개할 클래스 선언에는 {모듈이름}\_DLL 매크로를 붙임
Build.cs에서 참조 관계 설정
서브 모듈을 플러그인으로 분리할 수 있음

