

## 목차 
1. 개요 
2. 내용
   


## 개요

기존 Wave M에서 FTP를 사용하는 기능은 펌웨어 업데이트 하나였음. 추가되는 R1 나스모 기능은 FTP를 이용함. FTPClient는 새로운 인스턴스를 만들어도, 플러그인 코드 내부 객체는 static으로 선언 되어 있음. FTP를 연결할 때 펌웨어와 R1 나스모는 다른 Username을 가지고 로그인하고 다른 역할을 수행함. 따라서 기존의 FTP에 R1 나스모에 필요한 기능을 추가하고 FTP 연결을 관리할 필요성이 생김.


## 내용

#### FTP Connection Type

```cpp
// U2Define.h에 정의
UENUM()
enum class EFTPAddress : uint8
{
	None = 0,
	Firmware,
	SwingMotion
};
```
FTP는 Firmware , SwingMotion 두 가지 타입을 가지고 있음.


#### 제공 함수

```cpp

// Connecting & Disconnecting
void ConnectToFTP(EFTPAddress InAddress);
void DisconnectFromFTP();

// Upload & Download
void UploadDataToFTP(const FString& LocalFilePath, const FString& RemoteFilePath);
void DownloadDataFromFTP(const FString& LocalFilePath, const FString& RemoteFilePath);

// Get List
void GotoDirectory(const FString& Directory);
void ListCurrentDirectory();

```

각 함수에 대한 결과를 바인딩을 통해 Callback으로 받을 수 있음. 아래 Delegate Binding을 참고


#### Delegate Binding

```cpp

// U2FTPManager.h에 정의

DECLARE_DYNAMIC_MULTICAST_DELEGATE_FourParams(FOnU2FTPConnection, const EFTPAddress, Address, const bool, success, const int32, code, const FString, serverMessage);
DECLARE_DYNAMIC_MULTICAST_DELEGATE_FourParams(FOnU2FTPStatus, const EFTPAddress, Address, const bool, success, const int32, code, const FString, serverMessage);
DECLARE_DYNAMIC_MULTICAST_DELEGATE_SevenParams(FOnU2FTPProgress, const EFTPAddress, Address, const bool, bUploaded, const bool, end, const float, megabyteLeft, const float, megabyteSent, const int32, percent, const float, speedInMegabit);
DECLARE_DYNAMIC_MULTICAST_DELEGATE_FiveParams(FOnU2FTPGotoDirectory, const EFTPAddress, Address, const bool, success, const int32, code, const FString, serverMessage, FString, currentDir);
DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams(FOnFTPList, const EFTPAddress, Address, const TArray<FString>&, files);

FOnU2FTPConnection OnU2FTPConnection;
FOnU2FTPStatus OnU2FTPStatus;
FOnU2FTPProgress OnU2FTPProgress;
FOnU2FTPGotoDirectory OnU2FTPGotoDirectory;
FOnFTPList OnU2FTPList;

```

* Connecting & Disconnecting에 대한 콜백을 받고 싶음 -> OnU2FTPConnection에 바인딩.

* Uploading & Downloading에 대한 콜백을 받고 싶음 -> OnU2FTPProgress에 바인딩.

* FTP에서 워킹 폴더를 변경에 대한 콜백을 받고 싶음 -> OnU2FTPGotoDirectory에 바인딩.

* FTP에서 해당 폴더에 있는 파일 이름들을 가져오고 싶음 -> OnU2FTPList에 바인딩


