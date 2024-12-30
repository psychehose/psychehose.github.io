### UE 물리 파이프라인


커스텀 물리를 적용하기 위해서 해야할 것?
물리 시뮬레이션 중 onAdvance()을 하이재킹 한 후 그곳에 코드를 작성해야함.

#### 주요 개념

PhysX는 물리 시뮬레이션 - 실행단계 - 적용(인터그레이션)를 가짐.

```cpp
    // 예시코드
    PxScene scene;
    scene->simulate(timestep);
    scene->fetchResults(true);
```
1. 시뮬레이션 준비 단계

2. 시뮬레이션 실행
   * 실제 물리 계산 수행
   * 충돌 해결 및 제약 조건 처리
   * 힘과 토크 적용

3. 통합 단계
   * 계산된 물리 상태를 게임 객체에 반영
   * 이벤트 처리 (충돌 콜백 등)



#### PhysScene

PhysScene은 Unreal Engine에서 물리 시뮬레이션이 실행되는 독립된 공간 -> 게임 월드의 물리적 표현

PhysScene은 템플릿 클래스로 추상화 되어있음.

구현체: PhysScene_Chaos, PhysScene_PhysX

[1. 기본 초기화 및 업데이트 함수]
```cpp
void Init()
void Tick(float InDeltaSeconds)
```

Init(): 물리 씬의 초기 설정을 담당. 메모리 할당, 기본 파라미터 설정 등의 초기화 작업
Tick(): 매 프레임마다 호출되어 물리 시뮬레이션을 업데이트. InDeltaSeconds는 이전 프레임과의 시간 간격입니다.

[2. 키네마틱 및 프레임 관련 함수]
```cpp
void SetKinematicUpdateFunction(...)
void SetStartFrameFunction(...)
void SetEndFrameFunction(...)
```

SetKinematicUpdateFunction:
    키네마틱 객체들의 움직임을 제어하는 함수를 설정
    애니메이션이나 스크립트로 제어되는 물리 객체의 업데이트를 담당
    매개변수: 데이터, 델타 시간, 누적 시간, 반복 횟수

SetStartFrameFunction:
    각 물리 시뮬레이션 프레임 시작 시 실행될 함수 설정
    초기 상태 설정, 데이터 준비 등을 수행


SetEndFrameFunction:
    각 물리 시뮬레이션 프레임 종료 시 실행될 함수 설정
    결과 처리, 상태 저장 등을 수행



[3. 물리 객체 생성 및 파라미터 관리]
```cpp
void SetCreateBodiesFunction(...)
void SetParameterUpdateFunction(...)
```

SetCreateBodiesFunction:
    물리 객체(강체, 파티클 등)를 생성하는 함수 설정
    새로운 물리 객체가 필요할 때 호출됨
    매개변수: 물리 데이터 구조체 참조

SetParameterUpdateFunction:
    물리 시뮬레이션 파라미터 업데이트 함수 설정
    질량, 마찰, 탄성 등의 물리 속성 업데이트
    매개변수: 데이터, 델타 시간, 반복 횟수



[4. 충돌 및 제약조건 관리]
```cpp
void SetDisableCollisionsUpdateFunction(...)
void AddPBDConstraintFunction(...)
```

SetDisableCollisionsUpdateFunction:
    특정 객체 쌍 간의 충돌을 비활성화하는 함수 설정
    매개변수: 충돌 비활성화할 객체 쌍의 인덱스 집합


AddPBDConstraintFunction:
    Position Based Dynamics 제약조건 추가 함수
    물리 기반 애니메이션, 천 시뮬레이션 등에 사용
    매개변수: 데이터, 델타 시간


[5. 힘 적용 및 구현 접근]

```cpp
void AddForceFunction(...)
ImplType& GetImpl()
```
AddForceFunction:
    물리 객체에 힘을 적용하는 함수 추가
    중력, 바람, 폭발 등의 외력 시뮬레이션
    매개변수: 데이터, 델타 시간, 반복 횟수


GetImpl():
    실제 구현체에 대한 접근자
    템플릿 구현체의 세부 기능 접근에 사용

#### PhysScene_PhysX
Unreal Engine에서 사용하는 PhysScene 구현체임. 물리 엔진은 PhysX를 사용.

#### APEX
NVIDIA APEX는 PhysX 엔진을 확장한 물리 시뮬레이션 미들웨어

[주요 기능]
파괴 시뮬레이션
    건물이나 구조물의 사실적인 파괴 효과
    동적인 파편(debris) 생성

의류 시뮬레이션
    캐릭터의 옷감 물리
    실시간 의류 변형

입자 시스템
    연기, 불, 폭발 효과
    대규모 입자 시뮬레이션


#### CCD

CCD(Continuous Collision Detection, 연속 충돌 감지)
[핵심 개념]
CCD는 고속으로 움직이는 물체들의 충돌을 정확하게 감지하기 위한 물리 시뮬레이션 기술

[장점]
    정확한 충돌 감지
    빠른 물체의 충돌도 놓치지 않음
    얇은 물체 통과 현상 방지
    더 나은 게임플레이 경험
    물리 기반 게임에서 중요
    총알, 발사체 등의 정확한 처리
[단점]
    성능 오버헤드
    더 많은 계산이 필요
    모든 물체에 적용하면 성능 저하

#### CallbackFactory

1. ISimEventCallbackFactory를 상속받은 CallbackFactory 구현
2. 실제 콜백 클래스도 구현
3. 팩토리 등록 및 사용

```cpp
// 커스텀 시뮬레이션 이벤트 콜백 팩토리
class FMySimEventCallbackFactory : public ISimEventCallbackFactory
{
public:
    // Create 메서드 구현
    virtual physx::PxSimulationEventCallback* Create(FPhysScene_PhysX* PhysScene) override
    {
        // 새로운 콜백 객체 생성
        return new FMySimulationCallback(PhysScene);
    }

    // Destroy 메서드 구현
    virtual void Destroy(physx::PxSimulationEventCallback* Callback) override
    {
        // 콜백 객체 정리
        delete Callback;
    }
};

// 커스텀 시뮬레이션 콜백
class FMySimulationCallback : public physx::PxSimulationEventCallback
{
public:
    FMySimulationCallback(FPhysScene_PhysX* InPhysScene)
        : PhysScene(InPhysScene)
    {}

    // 충돌 이벤트 처리
    virtual void onContact(const physx::PxContactPairHeader& PairHeader,
                         const physx::PxContactPair* Pairs,
                         PxU32 NumPairs) override
    {
        // 충돌 처리 로직
        UE_LOG(LogPhysics, Log, TEXT("충돌 발생!"));
    }

    // 다른 필요한 이벤트 메서드들도 구현
    virtual void onConstraintBreak(...) override { }
    virtual void onWake(...) override { }
    virtual void onSleep(...) override { }
    // ...

private:
    FPhysScene_PhysX* PhysScene;
};


// 엔진 초기화 시점에서
void InitializePhysics()
{
    // 전역 팩토리 설정
    FPhysScene_PhysX::SimEventCallbackFactory = MakeShared<FMySimEventCallbackFactory>();
}

```


### Actor와 Aggregate의 차이점

Aggregate - 집합체

Aggregate(집합체)는 여러 개의 Actor들을 하나의 그룹으로 관리하는 PhysX의 개념

[차이점]

성능 최적화
    Aggregate: 여러 물체를 하나의 broad-phase 영역으로 처리
    Actor: 개별적으로 broad-phase 검사 수행

메모리 관리
    Aggregate: 그룹 단위로 메모리 관리 가능
    Actor: 개별적인 메모리 관리 필요


#### void FPhysScene_PhysX::InitPhysScene(const AWorldSettings* Settings)

메서드 안에서 PhysScene에 대한 플래그를 설정함.

PxSceneFlag

Active Actors/Transform 관련
    eENABLE_ACTIVE_ACTORS           // 활성화된 액터 알림 기능
    eENABLE_ACTIVETRANSFORMS       // 활성화된 트랜스폼 알림 기능

* 움직이는 물체들의 상태를 추적할 때 사용
* 성능에 영향을 줄 수 있으므로 필요한 경우에만 활성화

충돌 감지 관련
    eENABLE_CCD                    // 연속 충돌 감지(CCD) 활성화
    eDISABLE_CCD_RESWEEP          // CCD 재스윕 비활성화
    eENABLE_PCM                    // GJK 기반 거리 충돌 감지

* CCD는 고속 이동 물체의 터널링을 방지
* PCM은 더 정확한 충돌 감지를 제공

키네마틱 상호작용
    eENABLE_KINEMATIC_STATIC_PAIRS  // 키네마틱-정적 물체 간 상호작용
    eENABLE_KINEMATIC_PAIRS         // 키네마틱 물체들 간 상호작용

* 키네마틱 물체들의 충돌 필터링을 제어
* 기본적으로는 비활성화

[중요한 성능 관련 플래그]
캐시와 버퍼
    eDISABLE_CONTACT_CACHE  // 접촉 캐시 비활성화
    eDISABLE_CONTACT_REPORT_BUFFER_RESIZE  // 접촉 보고 버퍼 크기 조정 비활성화

* 메모리 사용량과 성능 사이의 균형을 조절

안정성 향상
    eENABLE_STABILIZATION   // 추가적인 안정화 패스 활성화
    eENABLE_AVERAGE_POINT   // 접촉 매니폴드의 평균점 활성화


PxSceneFlag::eENABLE_ACTIVE_ACTORS; 플래그를 켜야 onAdvance() 하이재킹 가능