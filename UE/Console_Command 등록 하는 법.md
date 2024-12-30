
#### 개념

*  IConsoleCommand: 콘솔 명령어
*  IConsoleManager: 콘솔 명령어를 실행하는 주체
*  RegisterConsoleCommand: 콘솔 명령어 등록
*  UnregisterConsoleObject: 콘솔 명령어 해제

```cpp

	virtual IConsoleCommand* RegisterConsoleCommand(const TCHAR* Name, const TCHAR* Help, const FConsoleCommandDelegate& Command, uint32 Flags) override;
	
	virtual IConsoleCommand* RegisterConsoleCommand(const TCHAR* Name, const TCHAR* Help, const FConsoleCommandWithArgsDelegate& Command, uint32 Flags) override;
	
	virtual IConsoleCommand* RegisterConsoleCommand(const TCHAR* Name, const TCHAR* Help, const FConsoleCommandWithWorldDelegate& Command, uint32 Flags) override;
	
	virtual IConsoleCommand* RegisterConsoleCommand(const TCHAR* Name, const TCHAR* Help, const FConsoleCommandWithWorldAndArgsDelegate& Command, uint32 Flags) override;
	
	virtual IConsoleCommand* RegisterConsoleCommand(const TCHAR* Name, const TCHAR* Help, const FConsoleCommandWithWorldArgsAndOutputDeviceDelegate& Command, uint32 Flags) override;
	
	virtual IConsoleCommand* RegisterConsoleCommand(const TCHAR* Name, const TCHAR* Help, const FConsoleCommandWithOutputDeviceDelegate& Command, uint32 Flags) override;
	
	virtual IConsoleCommand* RegisterConsoleCommand(const TCHAR* Name, const TCHAR* Help, uint32 Flags) override;

    virtual void UnregisterConsoleObject( IConsoleObject* Object, bool bKeepState) override;

```


```cpp
/** Console command delegate type (takes no arguments.)  This is a void callback function. */
DECLARE_DELEGATE( FConsoleCommandDelegate );

/** Console command delegate type (with arguments.)  This is a void callback function that always takes a list of arguments. */
DECLARE_DELEGATE_OneParam( FConsoleCommandWithArgsDelegate, const TArray< FString >& );

/** Console command delegate type with a world argument. This is a void callback function that always takes a world. */
DECLARE_DELEGATE_OneParam( FConsoleCommandWithWorldDelegate, UWorld* );

/** Console command delegate type (with a world and arguments.)  This is a void callback function that always takes a list of arguments and a world. */
DECLARE_DELEGATE_TwoParams(FConsoleCommandWithWorldAndArgsDelegate, const TArray< FString >&, UWorld*);

/** Console command delegate type (with a world arguments and output device.)  This is a void callback function that always takes a list of arguments, a world and output device. */
DECLARE_DELEGATE_ThreeParams(FConsoleCommandWithWorldArgsAndOutputDeviceDelegate, const TArray< FString >&, UWorld*, FOutputDevice&);

/** Console command delegate type with the output device passed through. */
DECLARE_DELEGATE_OneParam( FConsoleCommandWithOutputDeviceDelegate, FOutputDevice& );

```

*  FConsoleCommandDelegate: 인자 없이 실행되는 가장 단순한 콘솔 명령을 위한 델리게이트입니다.

*  FConsoleCommandWithArgsDelegate: 문자열 배열(TArray<FString>&)을 인자로 받는 콘솔 명령을 위한 델리게이트입니다. 사용자가 입력한 추가 인자를 처리할 수 있습니다.
  
*  FConsoleCommandWithWorldDelegate: UWorld* 타입의 인자를 받는 콘솔 명령을 위한 델리게이트입니다. 현재 게임 월드에 접근해야 하는 명령에 유용합니다.
  
* FConsoleCommandWithWorldAndArgsDelegate: 문자열 배열과 UWorld* 포인터를 인자로 받는 콘솔 명령을 위한 델리게이트입니다. 인자와 함께 게임 월드에 접근해야 하는 명령에 사용됩니다.

* FConsoleCommandWithWorldArgsAndOutputDeviceDelegate: 문자열 배열, UWorld* 포인터, 그리고 FOutputDevice& 참조를 인자로 받는 콘솔 명령을 위한 델리게이트입니다. 출력 장치를 통해 결과를 직접 제어할 수 있는 고급 명령에 사용됩니다.

* FConsoleCommandWithOutputDeviceDelegate: FOutputDevice& 참조만을 인자로 받는 콘솔 명령을 위한 델리게이트입니다. 출력 제어가 필요하지만 다른 인자는 필요 없는 명령에 사용됩니다.

#### 언리얼 엔진의 콘솔 변수(Console Variable) 시스템에서 사용되는 플래그들을 정의하는 열거형(enum)

```cpp
enum EConsoleVariableFlags
{
	/* Mask for flags. Use this instead of ~ECVF_SetByMask */
	ECVF_FlagMask = 0x0000ffff,

	/**
	 * Default, no flags are set, the value is set by the constructor 
	 */
	ECVF_Default = 0x0,
	/**
	 * Console variables marked with this flag behave differently in a final release build.
	 * Then they are are hidden in the console and cannot be changed by the user.
	 */
	ECVF_Cheat = 0x1,
	/**
	 * Console variables cannot be changed by the user (from console).
	 * Changing from C++ or ini is still possible.
	 */
	ECVF_ReadOnly = 0x4,
	/**
	 * UnregisterConsoleObject() was called on this one.
	 * If the variable is registered again with the same type this object is reactivated. This is good for DLL unloading.
	 */
	ECVF_Unregistered = 0x8,
	/**
	 * This flag is set by the ini loading code when the variable wasn't registered yet.
	 * Once the variable is registered later the value is copied over and the variable is destructed.
	 */
	ECVF_CreatedFromIni = 0x10,
	/**
	 * Maintains another shadow copy and updates the copy with render thread commands to maintain proper ordering.
	 * Could be extended for more/other thread.
 	 * Note: On console variable references it assumes the reference is accessed on the render thread only
	 * (Don't use in any other thread or better don't use references to avoid the potential pitfall).
	 */
	ECVF_RenderThreadSafe = 0x20,

	/* ApplyCVarSettingsGroupFromIni will complain if this wasn't set, should not be combined with ECVF_Cheat */
	ECVF_Scalability = 0x40,

	/* those cvars control other cvars with the flag ECVF_Scalability, names should start with "sg." */
	ECVF_ScalabilityGroup = 0x80,

	// ------------------------------------------------

	/* Set flags */
	ECVF_SetFlagMask =				0x00ff0000,

	// Use to set a cvar without calling all cvar sinks. Much faster, but potentially unsafe. Use only if you know the particular cvar/setting does not require a sink call
	ECVF_Set_NoSinkCall_Unsafe =	0x00010000,

	// ------------------------------------------------

	/* to get some history of where the last value was set by ( useful for track down why a cvar is in a specific state */
	ECVF_SetByMask =				0xff000000,

	// the ECVF_SetBy are sorted in override order (weak to strong), the value is not serialized, it only affects it's override behavior when calling Set()

	// lowest priority (default after console variable creation)
	ECVF_SetByConstructor =			0x00000000,
	// from Scalability.ini (lower priority than game settings so it's easier to override partially)
	ECVF_SetByScalability =			0x01000000,
	// (in game UI or from file)
	ECVF_SetByGameSetting =			0x02000000,
	// project settings (editor UI or from file, higher priority than game setting to allow to enforce some setting fro this project)
	ECVF_SetByProjectSetting =		0x03000000,
	// per project setting (ini file e.g. Engine.ini or Game.ini)
	ECVF_SetBySystemSettingsIni =	0x04000000,
	// per device setting (e.g. specific iOS device, higher priority than per project to do device specific settings)
	ECVF_SetByDeviceProfile =		0x05000000,
	// consolevariables.ini (for multiple projects)
	ECVF_SetByConsoleVariablesIni = 0x06000000,
	// a minus command e.g. -VSync (very high priority to enforce the setting for the application)
	ECVF_SetByCommandline =			0x07000000,
	// least useful, likely a hack, maybe better to find the correct SetBy...
	ECVF_SetByCode =				0x08000000,
	// editor UI or console in game or editor
	ECVF_SetByConsole =				0x09000000,

	// ------------------------------------------------
};

```


* ECVF_Default (0x0): 기본값으로, 아무 플래그도 설정되지 않은 상태입니다.
  
* ECVF_Cheat (0x1): 치트 플래그입니다. 최종 릴리스 빌드에서는 콘솔에 숨겨지고 사용자가 변경할 수 없습니다.
  
* ECVF_ReadOnly (0x4): 읽기 전용 플래그입니다. 사용자가 콘솔에서 변경할 수 없지만, C++ 코드나 ini 파일을 통해 변경 가능합니다.
  
* ECVF_Unregistered (0x8): UnregisterConsoleObject()가 호출되어 등록 해제된 상태를 나타냅니다.
* ECVF_CreatedFromIni (0x10): ini 파일에서 로드되었지만 아직 등록되지 않은 변수를 나타냅니다.
* ECVF_RenderThreadSafe (0x20): 렌더 스레드에서 안전하게 사용할 수 있는 변수임을 나타냅니다.
* ECVF_Scalability (0x40): 스케일러빌리티 설정과 관련된 변수임을 나타냅니다.
* ECVF_ScalabilityGroup (0x80): 스케일러빌리티 그룹을 제어하는 변수임을 나타냅니다.
* ECVF_Set_NoSinkCall_Unsafe (0x00010000): 콜백 없이 빠르게 설정할 수 있는 플래그입니다.
* ECVF_SetByXXX 플래그들: 변수가 어디서 설정되었는지를 나타내는 플래그들입니다. 예를 들어 ECVF_SetByConsole은 콘솔에서 설정되었음을 의미합니다.

이 플래그들은 비트 마스크로 사용되어 콘솔 변수의 특성과 동작을 제어합니다. 예를 들어, 치트 변수를 만들려면 ECVF_Cheat 플래그를, 읽기 전용 변수를 만들려면 ECVF_ReadOnly 플래그를 사용할 수 있습니다.

#### Example

1. ECVF_Default (0x0):
   ```cpp
   static FConsoleVariableRef CVarDefaultExample(
       TEXT("example.Default"),
       DefaultValue,
       TEXT("An example of a default console variable"),
       ECVF_Default
   );
   ```
   사용 사례: 특별한 제한이나 동작이 필요 없는 일반적인 콘솔 변수에 사용됩니다.

2. ECVF_Cheat (0x1):
   ```cpp
   static FConsoleVariableRef CVarInfiniteAmmo(
       TEXT("cheat.InfiniteAmmo"),
       bInfiniteAmmo,
       TEXT("Enables infinite ammo"),
       ECVF_Cheat
   );
   ```
   사용 사례: 치트나 디버그 목적으로 사용되는 변수에 적용됩니다.

3. ECVF_ReadOnly (0x4):
   ```cpp
   static FConsoleVariableRef CVarBuildVersion(
       TEXT("game.BuildVersion"),
       BuildVersion,
       TEXT("Current build version"),
       ECVF_ReadOnly
   );
   ```
   사용 사례: 사용자가 수정해서는 안 되는 정보를 저장하는 변수에 사용됩니다.

4. ECVF_Unregistered (0x8):
   ```cpp
   // This flag is typically set internally by the engine
   SomeConVar->SetFlags(SomeConVar->GetFlags() | ECVF_Unregistered);
   ```
   사용 사례: 콘솔 변수가 언레지스터되었음을 나타냅니다. 주로 내부적으로 사용됩니다.

5. ECVF_CreatedFromIni (0x10):
   ```cpp
   // This flag is typically set internally by the engine when loading from INI
   NewConVar->SetFlags(NewConVar->GetFlags() | ECVF_CreatedFromIni);
   ```
   사용 사례: INI 파일에서 생성된 콘솔 변수를 표시합니다. 주로 내부적으로 사용됩니다.

6. ECVF_RenderThreadSafe (0x20):
   ```cpp
   static FConsoleVariableRef CVarShadowQuality(
       TEXT("r.ShadowQuality"),
       ShadowQuality,
       TEXT("Controls shadow quality (0-4)"),
       ECVF_RenderThreadSafe
   );
   ```
   사용 사례: 렌더 스레드에서 안전하게 접근할 수 있는 변수에 사용됩니다.

7. ECVF_Scalability (0x40):
   ```cpp
   static FConsoleVariableRef CVarViewDistance(
       TEXT("sg.ViewDistanceQuality"),
       ViewDistanceQuality,
       TEXT("Controls view distance quality (0-3)"),
       ECVF_Scalability
   );
   ```
   사용 사례: 스케일러빌리티 설정의 일부로 사용되는 변수에 적용됩니다.

8. ECVF_ScalabilityGroup (0x80):
   ```cpp
   static FConsoleVariableRef CVarScalabilityGroup(
       TEXT("sg.ResolutionQuality"),
       ResolutionQuality,
       TEXT("Controls the resolution quality"),
       ECVF_ScalabilityGroup
   );
   ```
   사용 사례: 다른 스케일러빌리티 설정을 제어하는 그룹 변수에 사용됩니다.

9. ECVF_SetByConstructor (0x00000000):
   ```cpp
   // This is typically handled internally by the engine
   SomeConVar->Set(InitialValue, ECVF_SetByConstructor);
   ```
   사용 사례: 변수가 생성자에 의해 초기화되었음을 나타냅니다.

10. ECVF_SetByScalability (0x01000000):
    ```cpp
    // This would be set when loading scalability settings
    TextureQuality->Set(ScalabilityTextureQuality, ECVF_SetByScalability);
    ```
    사용 사례: Scalability.ini 파일에서 설정된 값임을 나타냅니다.

11. ECVF_SetByGameSetting (0x02000000):
    ```cpp
    // This would be set when the user changes a setting in the game menu
    AudioVolume->Set(UserSelectedVolume, ECVF_SetByGameSetting);
    ```
    사용 사례: 게임 내 설정에서 변경된 값임을 나타냅니다.

12. ECVF_SetByProjectSetting (0x03000000):
    ```cpp
    // This would be set when loading project settings
    MaxFPS->Set(ProjectMaxFPS, ECVF_SetByProjectSetting);
    ```
    사용 사례: 프로젝트 설정에서 정의된 값임을 나타냅니다.

13. ECVF_SetBySystemSettingsIni (0x04000000):
    ```cpp
    // This would be set when loading from Engine.ini or Game.ini
    GravityZ->Set(IniGravityZ, ECVF_SetBySystemSettingsIni);
    ```
    사용 사례: 시스템 설정 INI 파일에서 로드된 값임을 나타냅니다.

14. ECVF_SetByDeviceProfile (0x05000000):
    ```cpp
    // This would be set when applying device-specific settings
    MobileQualitySettings->Set(DeviceSpecificQuality, ECVF_SetByDeviceProfile);
    ```
    사용 사례: 특정 디바이스 프로필에 의해 설정된 값임을 나타냅니다.

15. ECVF_SetByConsoleVariablesIni (0x06000000):
    ```cpp
    // This would be set when loading from ConsoleVariables.ini
    PoolSize->Set(IniPoolSize, ECVF_SetByConsoleVariablesIni);
    ```
    사용 사례: ConsoleVariables.ini 파일에서 설정된 값임을 나타냅니다.

16. ECVF_SetByCommandline (0x07000000):
    ```cpp
    // This would be set when parsing command line arguments
    bStartInFullscreen->Set(CommandLineFullscreen, ECVF_SetByCommandline);
    ```
    사용 사례: 명령줄 인자로 설정된 값임을 나타냅니다.

17. ECVF_SetByCode (0x08000000):
    ```cpp
    // This would be set directly in game code
    AILogicInterval->Set(CalculatedInterval, ECVF_SetByCode);
    ```
    사용 사례: 코드에서 직접 설정된 값임을 나타냅니다.

18. ECVF_SetByConsole (0x09000000):
    ```cpp
    // This would be set when a user enters a command in the console
    DrawDebugLines->Set(UserInputDebugLines, ECVF_SetByConsole);
    ```
    사용 사례: 콘솔에서 사용자가 직접 입력하여 설정된 값임을 나타냅니다.

이러한 플래그들은 콘솔 변수의 동작을 제어하고, 값이 어디서 설정되었는지를 추적하는 데 사용됩니다. 이를 통해 개발자는 게임의 다양한 설정과 동작을 효과적으로 관리하고 디버그할 수 있습니다.

