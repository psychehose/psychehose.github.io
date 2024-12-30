
![[ebtnode_result_type.png]]


FindPatrolPos 노드에서 **Succeeded**를 리턴 했을 때
Wait 노드 -> FindPatrolPos  -> Move To 노드


FindPatrolPos 노드에서 강제로 **Failed**를 리턴 했을 때

다음 노드로 진행하지 않고 루트로 가서 처음부터 다시 시작합니다.
Wait 노드 -> FindPatrolPos (return Failed) -> Root 노드 -> Wait

![[ebtnode_result_failed.png]]

FindPatrolPos 노드에서 강제로 **InProgress**를 리턴 했을 때

틱마다 FindPatrolPos 노드를 실행합니다. Inprogress를 사용할 경우 계속 그 노드가 실행되기 때문에 탈출 조건을 명시 해야 합니다.

```cpp
if (어떤 조건이 true) 
	FinishLatentTask(OwnerComp, EBTNodeResult::Succeeded);
 
```

![[ebtnode_result_inprogress.png]]




FindPatrolPos 노드에서 강제로 **Aborted**를 리턴 했을 때
트리 노드의 실행이 완료되기 전에 중단될 때 사용됩니다.
작업이 중단되면 동작 트리 시스템은 새 작업으로의 전환을 적절하게 처리해야 합니다. 
그렇지 않다면 Inprogress와 비슷하게 계속 트리 노드를 실행합니다.  이것 역시Inprogress와 유사하게 **FinishLatentAbort**를 이용해서 처리하면 됩니다.



![[ebtnode_result_aborted.png]]


