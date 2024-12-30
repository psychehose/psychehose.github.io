
UGameplayStatics는 Unreal Engine에서 제공하는 유틸리티 클래스로, 게임플레이와 관련된 다양한 정적(static) 함수들을 포함하고 있습니다. 이 클래스는 게임 개발 시 자주 사용되는 여러 기능들을 쉽게 접근할 수 있게 해줍니다. 주요 기능들은 다음과 같습니다:

1. 게임 관리:
    - GetGameMode, GetGameState, GetGameInstance 등을 통해 현재 게임의 주요 객체들에 접근
    - OpenLevel을 통해 새로운 레벨 로드
2. 플레이어 관리:
    - GetPlayerController, GetPlayerCharacter, GetPlayerPawn 등을 통해 플레이어 관련 객체에 접근
    - CreatePlayer를 통해 새로운 플레이어 생성
3. 액터 관리:
    - SpawnActor를 통해 새로운 액터 스폰
    - GetAllActorsOfClass를 통해 특정 클래스의 모든 액터 찾기
4. 사운드 및 오디오:
    - PlaySound2D, PlayDialogueAtLocation 등을 통해 사운드 재생
5. 시간 관리:
    - GetTimeSeconds, GetRealTimeSeconds 등을 통해 게임 시간 정보 얻기
6. 저장 및 로드:
    - SaveGameToSlot, LoadGameFromSlot 등을 통해 게임 저장 및 로드
7. 물리 및 추적:
    - LineTraceSingleByChannel 등을 통해 물리적 충돌 검사
8. UI 및 HUD:
    - GetPlayerCameraManager를 통해 카메라 관리자에 접근
    - ProjectWorldToScreen을 통해 3D 위치를 2D 스크린 좌표로 변환
9. 디버깅:
    - PrintString을 통해 화면에 디버그 메시지 출력

이러한 기능들을 통해 UGameplayStatics는 게임 개발 과정에서 자주 필요한 작업들을 편리하게 수행할 수 있게 해줍니다. 특히 이 클래스의 함수들은 대부분 정적이므로, 객체를 생성하지 않고도 직접 호출할 수 있어 사용이 간편합니다.