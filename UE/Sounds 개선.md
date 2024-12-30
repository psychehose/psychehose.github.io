
## 목차

* 개요
* 컨텐츠 폴더 구조 (기획 ,디자인, 개발)
* U2SoundDataTable - SoundType 카테고리 (기획, 디자인)
* U2SoundManager - Init, Load (개발)
* Audio Stream Caching 적용 (개발)
* Sound 출력 Rule (기획, 디자인, 개발)
* 결과
* P.S 사운드 작업 시 살펴봐야할 에셋들

## 개요

 기존에 사운드 관리자의 부재로 인해 유지보수가 지속적으로 이뤄지지 않았음. 기존의 사운드 시스템은 게임 시작 버튼 클릭시에 로딩바에서 SoundsDataTable에 있는 모든 사운드 에셋을 로딩하고, 이것들을 TMap에 저장한 후에 필요할 때 출력하는 방식으로 구성되어 있었음. 유지보수 하기 쉽고 안정적이지만 사용하지 않는 사운드 모두 메모리에 올리기 때문에 로비에서 메모리가 낭비되는 결과를 가져올 수 있음. 

## Content 폴더 구조

![](foldering_to_be.png)

#### Ann
1. ann_crash: 볼 맞추고 판정소리 ()
   * 아나운서가 벙커, OB 이런거 내주는 소리
   * SoundType:인게임
   * 모두 ShotResult로 시작함.
   * ReplayState, U2GolfGameSoundComponent, UIIngameStrokePage에서 사용


2. ann_score: 찬스랑 결과 말해주는 소리
    * 총 세가지 유형:  찬스, 홀인 결과, 컨시드 결과
    * SoundType: 인게임
    * 찬스는 ScoreChane. 로 시작, UU2CaddieComponent에 종속
    * 홀인 결과와 컨시드 결과는 UU2GolfGameSoundComponent에 종속
    * 홀인원은 UMG(홀인원 결산)에서도 쓰임


3. ann_shot
   * 스윙했을 때 나는 소리들 (굿샷, 나이스어프로치, 나이스온, 나이스플레이, 나이스샷)
   * SoundType: 인게임
   * 모두 ShotResult. 로 시작함
   * ReplayState, U2GolfGameSoundComponent, UIIngameStrokePage에 종속
   * ann_crash와 거의 같음.


5. ambient 
   * UU2GolfGameSoundComponent에 귀속
   * SoundType: 인게임

6. ball_crash
   * BallCollidedDataTable에서 참조
   * SoundType: 인게임

7. bgm
   * BGM
   * SoundType: BGM

8. button
   * 버튼 클릭 소리 
   * SoundType: Common

9.  event
   * 이벤트 소리
   * 애니메이션에 넣지 못하는 소리 (분기처리 때문에 코드로 출력해야하는 사운드)
   * 코드로 호출
   * SoundType: Common 또는 인게임

10. gallery
   * 갤러리 사운드 (박수, 환호, 야유)
   * 인게임

11. swing_shot
   * 클럽별 임팩트 소리
   * 인게임

12. swing_control - 전부 인게임
   * 스윙게이지 소리
   * 인게임

13. animation
   * UMG 애니메이션에 넣는 소리
   * SoundType: Animation


## U2SoundDataTable - SoundType 카테고리

![](u2_soundtype.png)


1. None
   사용 안함 표시 -> 게임에서 사용하지 않겠다.

2. In Game
   * 게임 시작시에 로딩 됨
   * 게임중에만 사용
   * 스윙 임팩트, 스코어 사운드 등등
   * 코드로 호출

3. BGM
   * BGM
   * 코드로 호출

4. Animation
   * UMG에서 Animation - Audio Track에 추가할 사운드
   * 코드로 호출 불가
 
5. Common
   * 버튼 클릭 사운드
   * 코드로 호출할 필요가 있는 사운드 (클럽, 의상 갓챠는 시네마틱 분기가 안되어서 어쩔 수 없이 여기 넣음)
   * 코드 호출 가능



## U2SoundManager - Init, Load

#### Init

U2SoundManager 클래스는 게임 내 사운드 관리를 담당합니다. Init() 메서드를 통해 초기화되며, 다음과 같은 작업을 수행함

1. 메시지 매니저 초기화

2. 사운드 테이블 로드

3. 콘솔 명령어 설정 (에디터 모드에서만)

```cpp

void UU2SoundManager::Init()
{
    InitMessageManager();
    LoadSoundTable();

    #if WITH_EDITOR
    // 에디터에서만 콘솔 명령어 설정
    if (CMD_LogSound == nullptr)
    {
        CMD_LogSound = UU2ConsoleManager::Instance()->RegisterConsoleCommand(
            DEF_CSC_LOG_SOUND,
            TEXT("테스트용: 사운드 로그"),
            FConsoleCommandWithArgsDelegate::CreateUObject(this, &UU2SoundManager::CONSOLE_LogSound),
            ECVF_Default
        );
    }
    #endif
}

```
등록된 콘솔 명령어는 런타임에서 사용할 수 있음.

U2.Log.Sound 0 - Sound 로그 가리기

U2.Log.Sound 1 - Sound 로그 보이기

#### Load & UnLoad

LoadSoundTable() SoundsDataTable의 모든 행을 읽음.

각 행의 SoundType으로 각 사운드 에셋을 저장할 Map을 지정함.

코드에서 None, Animation은 break; 처리함.

None은 사용하지 않기 때문에 Map에 저장하지 않아도 되기 때문이고, Animation은 UMG에서 로드하고, 언로드하기 때문에 Map에 저장하지 않는다. (코드로 호출할 필요가 없기 때문)


InGame, BGM, Common은 코드로 호출할 필요가 있기 때문에, 지정된 Map에 저장을 한다. 그 중 InGame 사운드는 사운드 큐를 넣을 곳 (Value) nullptr을 넣는다. 이유는 로비에서 InGame 사운드를 사용하지 않기 때문에 미리 로드하지 않기 때문이다. BGm, Common은 LoadSound를 하고 지정된 Map의 Value에 사운드 큐를 저장한다.

```cpp
void UU2SoundManager::LoadSoundTable()
{
	if (IsAlreadyCalling)
	{
		return;
	}

	IsAlreadyCalling = true;

	TArray<FU2SoundDataTable*> RowArray;

	UU2DBManager::Instance()->GetDataTableRows<FU2SoundDataTable>(TEXT("SoundsDataTable"), RowArray);

	for (FU2SoundDataTable* it : RowArray)
	{
		switch (it->SoundType)
		{
		case EU2SoundType::None:
			break;

		case EU2SoundType::InGame:
			InGameSounds.Emplace(it->KeyValue, nullptr);
			break;

		case EU2SoundType::BGM:
			LoadSound(it->KeyValue, it->Sound, it->SoundType);
			break;

		case EU2SoundType::Animation:
			break;

		case EU2SoundType::Common:
			LoadSound(it->KeyValue, it->Sound, it->SoundType);
			break;

		default:
			break;
		}
	}
}

```

인게임 사운드 로드와 언로드는 UU2RoundLoadingState와, UU2GameToLobbyState에서 아래 함수를 통해서 이뤄진다.

```cpp
void UU2SoundManager::LoadInGameSounds()
{
	for (auto& Elem : InGameSounds)
	{
		if (!Elem.Value)
		{
			FU2SoundDataTable* Row = UU2DBManager::Instance()->GetDataTableRow<FU2SoundDataTable>(TEXT("SoundsDataTable"), FName(Elem.Key));
			TSoftObjectPtr<USoundBase> SoundPtr(Row->Sound);
			LoadSoundInternalAsync(Elem.Key, SoundPtr, EU2SoundType::InGame);
		}
	}
}

void UU2SoundManager::UnloadGameOnlySounds()
{
	for (auto& Elem : InGameSounds)
	{
		if (Elem.Value)
		{
			Elem.Value = nullptr;
			OutGameSounds.Remove(Elem.Key);
		}
	}
}

```

## Audio stream caching 적용

Audio Stream Caching은 Unreal Engine에서 오디오 성능을 최적화하기 위한 기능

* 효과
  * 쿡 타임에서 이 기능을 활성화하면 거의 모든 압축 오디오 데이터가 USoundWave 자산에서 분리되어 .pak 파일의 끝에 배치됨. 이를 통해 오디오가 어느 시점에서든 메모리에 로드되고 최근에 사용되지 않았을 때 다시 해제되는 것이 가능

* Audio Stream Caching 사용법
  * 각 Platform → iOS / Android → Audio → CookOverrides → Use Stream Caching
  * Max Cache Size 설정

* Prime On Load 적용 
  * Sound 재생 초기 부분을 미리 메모리에 올린다 → 재생 지연 방지 
  * Sound Class - Loading 에서 적용 가능


## Sound 출력 Rule

BGM: 코드로 출력 (보통 State에서 출력함)

Animation: UMG에서 오디오 트랙을 이용해서만 호출 (코드로 호출 X) 

In Game: 코드로 호출
   swing impact, crash 등등: 코드로만 호출
   Ingame UI: UMG 오디오 트랙으로 호출 가능

Common
   현재는 코드로 호출
   추후 BP를 이용해서 호출할 예정

## 결과

* 적용 전

로비, 게임: 132.275 MiB
![](insight_before.png)


* 적용 후 (Audio Stream Caching 사용)

Lobby ~ InGame 결과
초기화 ~ Lobby ~ 인게임: 132.275 MiB →  20~ 30 MiB (감소율 81.01%)
![](insight_after.png)
