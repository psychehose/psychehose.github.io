TL ;DR
언리얼 깃헙에서 브랜치 4.27plus를 받아서 빌드하기. 그리고 윈도우에서 remote 빌드 거는 게 더 합리적인 것 같음. 어차피 Objc 자동완성 사용 불가능


UE4 4.27.2 버전을 사용해서 tag가 4.27.2 버전을 다운로드 받음

https://github.com/EpicGames/UnrealEngine/tree/4.27.2-release


### [Mac](https://github.com/EpicGames/UnrealEngine/tree/4.27.2-release#mac)

1. Install **[GitHub for Mac](https://mac.github.com/)** then **[fork and clone our repository](https://guides.github.com/activities/forking/)**. To use Git from the Terminal, see the [Setting up Git](https://help.github.com/articles/set-up-git/) and [Fork a Repo](https://help.github.com/articles/fork-a-repo/) articles. If you'd rather not use Git, use the 'Download ZIP' button on the right to get the source directly.
    
2. Install the latest version of [Xcode](https://itunes.apple.com/us/app/xcode/id497799835).
    
3. Open your source folder in Finder and double-click on **Setup.command** to download binary content for the engine. You can close the Terminal window afterwards.
    
    If you downloaded the source as a .zip file, you may see a warning about it being from an unidentified developer (because .zip files on GitHub aren't digitally signed). To work around it, right-click on Setup.command, select Open, then click the Open button.
    
4. In the same folder, double-click **GenerateProjectFiles.command**. It should take less than a minute to complete.
    
5. Load the project into Xcode by double-clicking on the **UE4.xcworkspace** file. Select the **ShaderCompileWorker** for **My Mac** target in the title bar, then select the 'Product > Build' menu item. When Xcode finishes building, do the same for the **UE4** for **My Mac** target. Compiling may take anywhere between 15 and 40 minutes, depending on your system specs.
    
6. After compiling finishes, select the 'Product > Run' menu item to load the editor.


### 첫번째 이슈

MacOS에서 언리얼 엔진 소스 설치시 (4.27.2) 버그Setup.command를 클릭하면 디펜던시 설정을 함 근데remote server error가 발생함.

(403)Checking dependencies...  
Updating dependencies: 0% (0/63485)...  
Failed to download '[http://cdn.unrealengine.com/dependencies/UnrealEngine-12372779-e1515af26c634d2a8ade60b1afd1f065/01bb78539fc8dda386d45f9b5615f9a1e8ca5d94](http://cdn.unrealengine.com/dependencies/UnrealEngine-12372779-e1515af26c634d2a8ade60b1afd1f065/01bb78539fc8dda386d45f9b5615f9a1e8ca5d94)': The remote server returned an error: (403) Forbidden. (WebException)

이 경우에 Engine/Build/Commit.gitdeps.xml을  [https://github.com/EpicGames/UnrealEngine/tree/4.26/Engine/Build](https://github.com/EpicGames/UnrealEngine/tree/4.26/Engine/Build) 버전의 Engine/Build/Commit.gitdeps.xml으로 교체해주면 Setup.command를 해결할 수 있음.

참고: [https://github.com/carla-simulator/carla/issues/6486](https://github.com/carla-simulator/carla/issues/6486)

![GitHub](https://slack-imgs.com/?c=1&o1=wi32.he32.si&url=https%3A%2F%2Fa.slack-edge.com%2F80588%2Fimg%2Funfurl_icons%2Fgithub.png)GitHub

[Unreal Setup.bat (failed to download error) · Issue #6486 · carla-simulator/carla](https://github.com/carla-simulator/carla/issues/6486) (51kB)


### 2번째 이슈

ShaderCompileWorker 빌드 시에 아래와 같은 버그가 발생

![[var_error.jpg]]

Xcode가 빌드할 때 컴파일 옵션을 좀 더 타이트하게 잡아서 발생 하는 이슈
컴파일러에게 -Wno-unused-but-set-variable 설정을 하면 해결할 수 있음. UE4는 씨샾 코드를 빌드하기 때문에 { 언리얼 엔진 경로 }/Source/Programs/UnrealBuildTool/Platform/Mac/MacToolChain.cs에서

GetCompileArguments_Global 함수를 찾아서 Result가 선언된 다음 라인에 코드를 작성하면 됨

```cs
	Result += " -Wno-unused-but-set-variable";
```

여기까지 하면 ShaderCompileWorker 빌드 성공할 수 있음

### 3번째 이슈

이제 UE4 빌드를 해야 하는데 info.plist가 없어서 빌드 실패하는 이슈가 발생

이것은 UE4 Project - Build Setting에서 UE4GENERATE_INFOPLIST_FILE 옵션을 YES로 설정하기


### 4번째 이슈

Engine/Plugins/Media/BinkMedia/Source/SDK/lib/BinkUnrealMac.a가 없다는 이슈가 발생


/Users/choeseung-in/Downloads/UnrealEngine-4.27.2-release/clang:1:1: no such file or directory: '/Users/choeseung-in/Downloads/UnrealEngine-4.27.2-release/Engine/Plugins/Media/BinkMedia/Source/SDK/lib/BinkUnrealMac.a'

이건 해당 경로에 static library가 없다는 이슈다.
Setup.command와 GenerateProjectFiles.command 과정은 디펜던시를 설정하고 다운로드 해와서 필요한 경로에 넣는 역할인데 이 부분이 잘못 되어서 그런 것으로 추정된다. 
그래서 Issue를 찾아보던 중에 branch에 4.27 plus가 있는 것을 발견하였고 계속 유지보수가 이뤄지고 있어서 다시 엔진 소스를 받아서 실행하니 빌드를 할 수 있었다.






