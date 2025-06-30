

```
shadowMapSize - 그림자 맵 텍스처의 해상도(예: 1024x1024, 2048x2048)
 높을수록 → 더 정밀한 그림자, 더 부드러운 경계
 낮을수록 → 거친 그림자, 계단 현상 발생
 PCF에서 텍셀 크기 계산에 사용
```


```
shadowBias - "Shadow Acne" (그림자 여드름) 현상 방지
 부동소수점 정밀도 한계로 인한 자기 그림자 현상을 방지
 깊이 비교 시 작은 오프셋을 추가하여 오차 보정
```

#### Depth-Only Pass의 의미

- 목적: 광원 시점에서 본 장면의 깊이 정보만 저장
- 결과물: Shadow Map (깊이 텍스처)

Depth-Only Pass가 필요한 이유는

1. 2-Pass 렌더링
	* 1단계: 광원 시점에서 깊이만 렌더링 (현재 함수)
	* 2단계: 카메라 시점에서 색상 + 그림자 렌더링

2. 효율성: 색상 계산 없이 깊이만 저장하여 성능 최적화

3. **그림자 판단 기준**: 나중에 "이 픽셀이 광원에서 보이는가?"를 판단할 데이터 생성

#### Blinn-Phong 정점 셰이더 (그림자 적용)

```metal
out.shadowPosition = shadowUniforms.lightViewProjectionMatrix * worldPosition;
```

1. **좌표계 변환 체인**:
    - 로컬 좌표 → 월드 좌표 → 광원 뷰 좌표 → 광원 클립 좌표
      
2. **핵심 연산**:
    - 각 정점을 광원의 시점에서 본 좌표로 변환
    - 나중에 "이 점이 그림자 안인지 밖인지" 판단할 데이터 준비
      
3. **왜 정점 셰이더에서?**:
    - GPU 병렬 처리 효율성
    - 프래그먼트 셰이더에서 보간된 값 사용



####  PCF (Percentage Closer Filtering)

* 문제상황: 단순 그림자 매핑은 계단 현상 발생
* 해결: 주변 여러 샘플을 확인해서 부드러운 그림자 생성
* 그래서 x = -1 ... 1 , y = -1 ... 1 을 사용해서 총 9개 샘플 사용 (3 x 3  커널)
* 
```swift
float sampleShadowMapPCF(depth2d<float> shadowMap, sampler shadowSampler, float2 shadowCoord, float currentDepth, float bias, float shadowMapSize) {
  float shadow = 0.0;
  float2 texelSize = 1.0 / shadowMapSize;
  
  // 3x3 PCF
  for (int x = -1; x <= 1; ++x) {
    for (int y = -1; y <= 1; ++y) {
      float2 offset = float2(x, y) * texelSize;
      float pcfDepth = shadowMap.sample(shadowSampler, shadowCoord + offset);
      shadow += (currentDepth - bias) > pcfDepth ? 1.0 : 0.0;
    }
  }
  
  return shadow / 9.0; // 9개 샘플의 평균
}
```


####  Blinn-Phong 프래그먼트 셰이더

* NDC 좌표 변환
```swift
float3 projCoords = in.shadowPosition.xyz / in.shadowPosition.w;  
projCoords = projCoords * 0.5 + 0.5;
```

* 그림자 적용
```swift
float3 finalColor = ambient + (1.0 - shadow) * (diffuse + specular);
```

- Ambient**: 항상 유지 (간접광 시뮬레이션)
- **Diffuse/Specular**: 그림자 영향으로 감소
-  `(1.0 - shadow)`:  shadow가 1이면 완전 어둡게, 0이면 원래 밝기

#### 전체 그림자 렌더링 파이프라인
*  단계별 과정
	1. Shadow Map 생성: 광원 시점에서 깊이만 렌더링
	2. 메인 렌더링: 카메라 시점에서 색상 + 조명 계산
	3. 그림자 테스트: 각 픽셀이 광원에서 보이는지 확인
	4. PCF 적용: 부드러운 그림자 경계 생성
	5. 최종 색상 합성: 조명 + 그림자 결합

