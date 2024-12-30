# 데이터테이블에서 에셋을 참조하는 것을 방지하는 방법

문제상황:

> 모든 사운드 에셋 USound는 USoundDatable에서 관리한다. USoundDatable은 TSoftObjectPtr로 사운드큐를 직접 가지고 있다. 그리고 다른 특정상황에서 사용하는 데이터테이블이 있다. (CollidedSoundDataTable - 이를 C데이터테이블이라 칭함.) C데이터테이블도 마찬가지로 사운드큐를 가지고 있다. 만약 충돌 소리가 변경된다면 USoundDataTable 사운드큐도 바꿔줘야하고, C데이터테이블 사운드큐도 바꿔줘야한다.


C데이터테이블에서 USoundDataTable의 특정 Row를 참조하면 된다.

```cpp
USTRUCT()
struct U2CLIENT_API FU2BallCollidedDataTable : public FU2TableRowBase
{
    GENERATED_USTRUCT_BODY()
public:

    UPROPERTY(EditAnywhere)
    ESurfaceType surfaceType = ESurfaceType::None;

    UPROPERTY(EditAnywhere)
    FDataTableRowHandle SoundRowHandle;

    UPROPERTY(EditAnywhere)
    TSoftObjectPtr<UParticleSystem> PS_Collided = nullptr;
};

```

결과

![](datahandle_row_result.png)

코드에서 꺼내 쓰는 법.
```cpp
FU2SoundDataTable* row = resDT->SoundRowHandle.GetRow<FU2SoundDataTable>(TEXT("BallCollisionSound"));

		if (row == nullptr)
			return;

		if (row->bUseSound)
		{
			UU2SoundManager::Instance()->PlaySound(row->KeyValue);
		}
```
