
## 정의

Invalidation Box는 언리얼 엔진 4의 UMG(언리얼 모션 그래픽) UI 시스템에서 사용하는 위젯입니다. UI의 렌더링과 성능을 최적화하는 데 사용됩니다. Invalidation Box의 주요 기능은 자식 위젯의 렌더링을 캐시하는 것입니다. Invalidation Box 내부의 자식 위젯이 변경될 때, Invalidation Box는 전체 UI가 아닌 변경된 부분만 다시 렌더링하여 성능을 향상시킵니다.


## 역할

- **성능 최적화**: 자식 위젯의 렌더링을 캐시함으로써 드로우 콜 및 업데이트 수를 줄여 성능을 향상시킵니다. 특히 복잡한 UI에서 효과적입니다.
    
- **효율적인 리드로잉**: 변경이 발생할 때 영향을 받은 부분만 다시 그리므로 불필요한 계산과 렌더링 과정을 최소화합니다.
    
- **CPU 사용량 감소**: 드로우 콜이 줄어들고 업데이트 빈도가 낮아져 CPU 사용량이 크게 감소하여 게임이나 애플리케이션이 더 부드럽게 실행됩니다.


### Invalidation Box를 사용해야 할 때:

1. **복잡한 UI**: 많은 요소가 자주 업데이트되는 UI의 경우, Invalidation Box로 감싸면 리렌더링 범위를 제한하여 성능 향상을 얻을 수 있습니다.
    
2. **정적인 콘텐츠**: 자주 변경되지 않는 UI 섹션에 Invalidation Box를 사용하면 초기 렌더링을 캐시하여 중복 렌더링 사이클을 방지할 수 있습니다.
    
3. **동적인 요소**: 동적인 요소에서도 콘텐츠의 작은 부분만 변경되는 경우, Invalidation Box가 필요한 부분만 업데이트하여 성능을 최적화할 수 있습니다.


### Invalidation Box 사용 방법:

1. **UI에 추가**: UMG 위젯 팔레트에서 Invalidation Box를 드래그하여 UI 계층 구조에 추가합니다.
    
2. **위젯 래핑**: 최적화하려는 위젯을 Invalidation Box 내부에 배치합니다.
    
3. **설정 조정**: 필요에 따라 Invalidation Box 설정을 조정합니다. 기본적으로 자식 위젯을 올바르게 무효화하고 캐시하지만, 특정 성능 요구 사항에 따라 세부 조정이 가능합니다.
    
4. **테스트 및 프로파일링**: 언리얼 엔진의 프로파일링 도구를 사용하여 UI에서 성능 향상을 테스트하고 확인합니다. 필요에 따라 조정합니다.

