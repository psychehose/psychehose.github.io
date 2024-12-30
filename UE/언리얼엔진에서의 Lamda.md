
### 람다의 문제점

참조로 변수를 캡쳐하는데 람다가 실행되기 전에 해당 '객체'가 파괴되면 널 포인터 참조가 발생함. 이 현상이 언리얼엔진의 Deferred Execution(지연 실행), 비동기 작업, 생명주기 같은 것들과 맞물려서 위험할 수 있음.


### 람다의 문제점을 완화하는 방법


1. **Prefer Capturing by Value**
    
    가능하다면 참조 캡쳐보다 값 캡쳐를 사용할 것. 람다는 값을 복사해서 사용할 것이므로 독립적인 생명주기를 가지게 됨

2. **Use TWeakObjectPtr**

   Unreal 객체에 대한 포인터를 캡처할 때 raw 포인터 대신 TWeakObjectPtr를 사용하는 것. TWeakObjectPtr는 객체가 파괴되는 경우를 안전하게 처리하여 객체에 액세스하기 전에 객체가 여전히 유효한지 확인
   

3. **Check for Validity**
    
    캡쳐된 포인터나 레퍼런스에 접근하기 전에 nullptr 또는 TWeakObjectPtr를 체크한다.

4. **Using a Weak Lambda**

    Weak Lamda를 사용하면 람다 내부의 객체를 안전하게 참조할 수 있음. 소유 객체가 유효하지 않다면 대리자를 통해 람다를 호출하지 않음.

    ```cpp
    if (const UWorld* World = OwnerComp.GetWorld())
	{
		const float DeltaTime = NumTicksExecuting * FAITestHelpers::TickInterval;
		World->GetTimerManager().SetTimer(TaskMemory->TimerHandle,
			FTimerDelegate::CreateWeakLambda(this, [&OwnerComp, TaskMemory, this]() // <---- This is the declaration of a weak lambda
			{
				TaskMemory->TimerHandle.Invalidate();

				ensure(!TaskMemory->bIsAborting);
				LogExecution(OwnerComp, LogIndexExecuteFinish);
				FinishLatentTask(OwnerComp, LogResult);
			}), DeltaTime, false);
	}
    ```

