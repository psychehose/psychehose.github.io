

```cpp
enum class ETriggerEvent : uint8
{
	// No significant trigger state changes occurred and there are no active device inputs
	None		= (0x0)		UMETA(Hidden),
	// Triggering occurred after one or more processing ticks
	Triggered	= (1 << 0),	// ETriggerState (None -> Triggered, Ongoing -> Triggered, Triggered -> Triggered)
	
	// An event has occurred that has begun Trigger evaluation. Note: Triggered may also occur this frame, but this event will always be fired first.
	Started		= (1 << 1),	// ETriggerState (None -> Ongoing, None -> Triggered)

	// Triggering is still being processed. For example, an action with a "Press and Hold" trigger
	// will be "Ongoing" while the user is holding down the key but the time threshold has not been met yet. 
	Ongoing		= (1 << 2),	// ETriggerState (Ongoing -> Ongoing)

	// Triggering has been canceled. For example,  the user has let go of a key before the "Press and Hold" time threshold.
	// The action has started to be evaluated, but never completed. 
	Canceled	= (1 << 3),	// ETriggerState (Ongoing -> None)

	// The trigger state has transitioned from Triggered to None this frame, i.e. Triggering has finished.
	// Note: Using this event restricts you to one set of triggers for Started/Completed events. You may prefer two actions, each with its own trigger rules.
	// Completed will not fire if any trigger reports Ongoing on the same frame, but both should fire. e.g. Tick 2 of Hold (= Ongoing) + Pressed (= None) combo will raise Ongoing event only.
	Completed	= (1 << 4),	// ETriggerState (Triggered -> None)
};
```


ETriggerEvent는 Enum으로 6가지 타입이 있다.

1. None
2. Triggered,
3. Started
4. Ongoing
5. Canceled
6. Completed


이 타입들은 Action에서 Triggers를 어떻게 설정한지에 따라 달라진다.

Action Triggers에 Pressed를 설정했을 때

ETriggerEvent는 1틱씩

None -> Start -> Triggered -> Completed 순으로 발생한다.

Action Triggers에 Pressed, Release로 설정할 시에는

None -> Start-> Trigger -> Ongoing -> Trigger -> Completed


Action Triggers에 Hold And Release로 설정하고 Hold Time Threshold를 0.5초로 설정

이벤트를 주면

None -> Start -> Ongoing -> Triggered -> Completed

 Hold Time Threshold를 5초로 설정하고 스페이스를 2초 후에 눌렀다 떼면

None -> Start -> Ongoing -> Canceled


정리하면 기본상태는 None이고, 이벤트가 취해질 때 맨 처음 Start로 상태가 한틱 변경된다. 그리고 이벤트 요구사항이 충족 되기 전까지는 Ongoing을 유지하고 충족 되면 Triggered로, 충족되지 않는다면 Canceled로 상태가 변경된다. 그리고 다시 None으로 변경됨.

