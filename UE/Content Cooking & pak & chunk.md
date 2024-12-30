게임 실행파일과 필수 에셋: pak0

pak0을 설정하는 방법

1. DefaultGame.ini에서 DirectoriesToAlwaysCook로 설정
2. DataAsset에서 chunk id를 0로 설정한다.


언리얼 엔진 4.27의 DataAsset에 있는 해당 옵션들

1. Apply Recursively (재귀적 적용): 이 옵션을 활성화하면 선택한 디렉토리 내의 모든 하위 폴더와 파일에 대해서도 변경 사항이 적용됩니다. 즉, 현재 폴더뿐만 아니라 그 안에 포함된 모든 폴더와 에셋에 대해 동일한 설정이 적용됩니다.
2. Label Assets in My Directories (내 디렉토리의 에셋 라벨링): 이 옵션을 선택하면 현재 프로젝트의 콘텐츠 디렉토리 내에 있는 에셋들에 대해서만 라벨을 적용합니다. 엔진이나 플러그인의 콘텐츠는 제외됩니다.
3. Is Runtime Label (런타임 라벨 여부): 이 옵션을 활성화하면 해당 라벨이 게임 실행 중에도 사용 가능하도록 설정됩니다. 런타임에 라벨을 통해 에셋을 검색하거나 필터링해야 하는 경우에 유용합니다.

이 옵션들은 주로 에셋 관리와 조직화, 그리고 런타임 시 에셋 접근성을 향상시키는 데 사용됩니다. 프로젝트의 규모와 요구사항에 따라 적절히 설정하면 에셋 관리를 더욱 효율적으로 할 수 있습니다.

##### DirectoriesToAlwasysCook 과 DataAsset에서 Chunk id를 0으로 설정하는 것의 관계

- 보완적 사용: 두 방식은 서로 보완적으로 사용될 수 있음. DirectoriesToAlwaysCook가 전체 디렉토리를 다룬다면, 0RequiredPak은 더 세밀한 제어가 필요한 개별 에셋을 관리함
  
- 중복 가능성: 0RequiredPak에 명시된 에셋이 DirectoriesToAlwaysCook에 지정된 디렉토리 내에 있을 수 있음. 이 경우 해당 에셋은 두 번 쿠킹되지 않고, 한 번만 처리됨
  
- 우선순위: 일반적으로 0RequiredPak의 설정이 더 구체적이므로, 충돌이 있을 경우 이 설정이 우선적으로 적용됨.
  
- pak0 생성: 두 설정 모두 chunk id가 0인 에셋들을 지정하므로, 이들은 함께 pak0의 내용을 구성하는 데 기여








#### ISSUE 
1.  The required attribute "Include" is empty ...
```
error MSB4035: The required attribute "Include" is empty or missing from the element <ModulesToBuild>.
```
	
RunUAT 실행할 때 -compile 옵션을 없앴음. RunUAT에서 선택적으로 cooking이 되는 거 같지 않음
출처: https://forums.unrealengine.com/t/command-line-compilation-error-uat/467076/9

2. 현재 구조에서 맵이 없이 패키징 하는 것이 가능한가?
	1. 게임 실행 파일 + pak0만 쿠킹하고 싶음
	2. 많은 에셋 쿠킹이 오버라이드를 통해서 진행되어서 파악이 어려움
	3. DefaultGame.ini와 Tables, DataAsset 여러개가 엮어있음
	4. 예를들면 CC데이터테이블을 DefaultGame.ini에서 always cook으로 사용함 ->
	맵이 패키징됨


3. Error: CDO Constructor (Canvas): EngineResources/WhiteSquareTexture
	-> DefaultGame.ini에서 +DirectoriesToAlwaysCook=(Path="/Game/StarterContent")로 변경






시간이 금.....요일 할 게 있으니까 먼저 우직하게 workspace 2개 만들고

맵 없이, 맵 있는 채로


먼저 해야할 것




젠킨스로 s3 브라우저 자동 업로드 하는 방법 및 Manifest 생성