
 언리얼 엔진에서 int32와 같은 언리얼 엔진 데이터형이 따로 있는 이유는 다양한 플랫폼에 대응하기 때문이다.
언리얼은 문자 인코딩 방식 UTF-16을 사용하고 있음. 이것도 역시 다양한 플랫폼을 위해서..
윈도우는 CP949를 사용해서 언리얼엔진에서 한글 사용시 깨진다.  소스코드를 UTF-8로 저장하면 한글 제대로 나온다.


유니코드를 위한 언리얼 표준 Character 타입은 TCHAR이다.
언리얼 엔진에서 문자열을 사용할 때 TEXT 매크로를 사용해 생성하고 다뤄야한다.
TEXT 매크로로 감싸면 TCHAR 배열로 만들어짐!

FString은 단순하게 TCHAR를 가르키고 있는 포인터임. 그래서 FString을 출력하고 싶으면 디레퍼런스 연산자 \*를 사용해야함.

FString을 자르거나, 찾거나 등 문자열을 다루는 함수들은 내부적으로 FCString으로 타입이 변환한 후 처리된다. 이는 저수준의 C 함수를 사용해야하기 때문이다. 실제로 Atoi나 Atof 같은 함수를 사용하기 위해서는 FCString을 이용한다. 

```cpp
#include "MyGameInstance.h"
void UMyGameInstance::Init()
{  
	Super::Init();
	TCHAR LogCharArray[] = TEXT("Hello World");  
	UE_LOG(LogTemp, Log, TEXT("%s"), LogCharArray);
	
	FString LogCharString = LogCharArray;
	UE_LOG(LogTemp, Log, TEXT("%s"), *LogCharString);
	
	const TCHAR* LongCharPtr = *LogCharString;
	UE_LOG(LogTemp, Log, TEXT("%s"), LongCharPtr);
	
	TCHAR* LogCharDataPtr = LogCharString.GetCharArray().GetData();  
	UE_LOG(LogTemp, Log, TEXT("%s"), LogCharDataPtr); 
	TCHAR LogCharArrayWithSize[100];
	FCString::Strcpy(LogCharArrayWithSize, LogCharString.Len(), *LogCharString);  
	UE_LOG(LogTemp, Log, TEXT("%s"), LogCharArrayWithSize);
	
	// Find, Slice
	if (LogCharString.Contains(TEXT("World"), ESearchCase::IgnoreCase))
	{  
		int32 Index = LogCharString.Find(TEXT("World"),ESearchCase::IgnoreCase); 
		FString EndString = LogCharString.Mid(Index);
		UE_LOG(LogTemp, Log, TEXT("Find Test: %s"), *EndString);
	} 
	
	FString Left, Right;
 
	if (LogCharString.Split(TEXT(" "), &Left, &Right))  
	{  
		UE_LOG(LogTemp, Log, TEXT("%s and %s"), *Left, *Right);  
	}
	
	// Int, Float -> String
	int IntValue = 32;
	float FloatValue = 3.141592; // 스트링으로 전환

	FString FloatIntString = FString::Printf(TEXT("Int: %d, Float: %f"), IntValue, FloatValue);
	FString FloatString = FString::SanitizeFloat(FloatValue);
	FString IntString = FString::FromInt(IntValue);

	UE_LOG(LogTemp, Log, TEXT("%s"), *FloatIntString);  
	UE_LOG(LogTemp, Log, TEXT("Int:%s, Float:%s"), *IntString, *FloatString);
	
	int32 IntValueFromString = FCString::Atoi(*IntString);
	float FloatValueFromString = FCString::Atof(*FloatString);
	FString FloatIntFromString = FString::Printf(TEXT("Int: %d, Float: %f"), IntValueFromString, FloatValueFromString);

	UE_LOG(LogTemp, Log, TEXT("%s"), *FloatIntFromString); 

// FNAME  
// 에셋관리를 위한 것  
// 대소문자 구분 x  
// 한번 선언되면 바꿀 수 없음 key로 만들어짐 Key - Vlaue // 팁  
// 위의 코드는 overhead 발생 -> 아래코드 staticonyonce를 이용하기  
  
	for (int i = 0; i < 10000; ++i)  
	{  
		FName SerachInNamePool = FName(TEXT("pelvis"));  
		const static FName StaticOnlyOnce(TEXT("pelvis"));  
	}


```
