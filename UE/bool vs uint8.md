Unreal Engine의 리플렉션 시스템에서 `bool` 대신 `uint8`을 사용하는 것은 주로 효율성과 일관성을 위한 선택입니다. 여기에는 몇 가지 상세한 이유가 있습니다:

1. **메모리 효율성**: C++에서 `bool` 타입은 일반적으로 1바이트를 차지합니다. 하지만, 이는 표준에 의해 보장되는 것은 아닙니다. 따라서, 다른 컴파일러나 플랫폼에서 `bool`의 크기가 달라질 수 있습니다. `uint8`은 명확하게 1바이트 크기를 갖기 때문에 플랫폼 간 일관성을 보장합니다. 이러한 일관성은 특히 크로스-플랫폼 게임 개발에 있어 중요합니다.
    
2. **리플렉션과 직렬화**: Unreal Engine의 리플렉션 시스템은 객체의 프로퍼티를 자동으로 식별하고 처리하는 데 사용됩니다. 이 시스템은 `uint8`과 같은 명확하게 정의된 크기를 가진 데이터 타입을 더 잘 처리할 수 있습니다. 이는 직렬화(데이터를 네트워크를 통해 전송하거나 디스크에 저장하는 과정)에서 중요합니다. `bool`이 다른 크기를 가질 수 있다면, 직렬화 과정에서 데이터의 정확한 해석이 어려울 수 있습니다.
    
3. **비트 필드 사용**: Unreal Engine은 종종 비트 필드를 사용하여 메모리 사용을 최적화합니다. `uint8`은 이러한 목적으로 사용하기에 적합한 타입입니다. 비트 필드에서 각 비트는 별도의 플래그나 설정을 나타낼 수 있으며, 이는 `bool`보다 메모리를 더 효율적으로 사용하는 방법입니다.
    
4. **컴파일러 최적화**: 일부 컴파일러는 `bool` 타입을 특별히 최적화하지 않을 수 있습니다. `uint8`은 보다 일반적인 데이터 타입이므로, 컴파일러에 따라 `bool`보다 더 효율적으로 처리될 수 있습니다.
    
5. **확장성**: `bool`은 단순히 참/거짓만을 나타내지만, `uint8`을 사용하면 추가적인 정보를 동일한 메모리 공간에 저장할 수 있습니다. 예를 들어, 여러 상태나 모드를 나타내는데 `uint8`이 더 유용할 수 있습니다.

### Bool type의 크기가 달라질 수 있음?


네, C++ 표준에서 `bool` 타입의 정확한 크기는 명시하지 않고 있습니다. 표준은 단지 `bool`이 최소 1바이트를 차지해야 한다고 규정할 뿐입니다. 이는 `bool`이 다른 컴파일러나 아키텍처에서 다른 크기를 가질 수 있음을 의미합니다.

예를 들어, 대부분의 현대 컴파일러와 플랫폼에서 `bool`은 1바이트를 차지합니다. 이는 `bool`을 효율적으로 처리하기 위한 최소 크기입니다. 그러나 특정 시스템이나 컴파일러 최적화 설정에 따라 `bool`이 더 큰 메모리 공간을 차지할 수도 있습니다. 예를 들면:

1. **일부 오래된 또는 특수한 시스템**: 이러한 시스템에서 `bool`은 1바이트보다 큰 크기를 가질 수 있습니다. 예를 들어, 일부 구형 컴퓨터 아키텍처에서는 1바이트보다 큰 단위로 메모리를 관리하는 경우가 있었습니다.
    
2. **최적화 목적**: 컴파일러는 때때로 메모리 접근 속도를 개선하기 위해 `bool`의 크기를 늘릴 수 있습니다. 예를 들어, 특정 프로세서 아키텍처에서는 더 큰 단위의 메모리 접근이 더 효율적일 수 있어, `bool`을 그 크기에 맞춰 조정할 수 있습니다.
    
3. **특정 컴파일러 설정**: 개발자가 특정 컴파일러 옵션을 사용해 `bool`의 크기를 조절할 수도 있습니다. 이런 경우는 드물지만, 성능 최적화나 특정 하드웨어 요구 사항에 의해 발생할 수 있습니다.
    

이러한 상황에서 `bool`의 크기가 1바이트가 아닐 수 있으므로, 크로스-플랫폼 애플리케이션 개발 시 `bool`의 크기에 대해 가정하지 않는 것이 중요합니다. 대신, 명시적인 크기를 갖는 타입(예: `uint8_t`)을 사용하는 것이 더 안전합니다.