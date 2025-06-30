### 1. 조명 모델의 기본 구성 

두 모델 모두 **3가지 조명 성분**을 합쳐서 최종 색상을 계산
최종 색상 = Ambient + Diffuse + Specular

#### **Ambient Light (환경광)**
- 모든 방향에서 균등하게 오는 빛
- 그림자 부분도 완전히 검은색이 되지 않게 해줌
  
```
ambient = ambientColor × ambientStrength
```

#### **Diffuse Light (확산광)**
- 표면에서 모든 방향으로 균등하게 산란되는 빛
- 램버트 코사인 법칙 적용

```
diffuse = lightColor × max(0, dot(normal, lightDirection))
```

#### **Specular Light (정반사광)**
- 특정 방향으로만 반사되는 빛 (하이라이트)
- **여기서 Phong과 Blinn-Phong의 차이가 발생함

### 2. Phong 셰이딩 모델 (1975)

#### **Specular 계산 방식**
1. 완전 반사 벡터(R) 계산: R = reflect(-L, N)  
2. 반사벡터와 시선벡터 내적: dot(R, V)  
3. 최종: specular = pow(max(0, dot(R, V)), shininess)

#### **특징**
- **장점**: 물리적으로 직관적 (실제 반사 방향 계산)
- **단점**: reflect() 함수 연산이 비쌈 (벡터 계산 복잡)
- **시각적**: 더 날카롭고 집중된 하이라이트

---

### 3. Blinn-Phong 셰이딩 모델 (1977)

#### **Specular 계산 방식**
1. 하프 벡터(H) 계산: H = normalize(L + V)  
2. 하프벡터와 노말벡터 내적: dot(N, H)  
3. 최종: specular = pow(max(0, dot(N, H)), shininess)

#### **특징**
- **장점**: 계산이 더 빠름 (normalize 한 번만)
- **단점**: 물리적 정확도는 Phong보다 약간 떨어짐
- **시각적**: 좀 더 부드럽고 넓은 하이라이트


### 4. 핵심 차이점

| 구분           | Phong           | Blinn-Phong     |
| ------------ | --------------- | --------------- |
| **계산 복잡도**   | 높음 (reflect 연산) | 낮음 (normalize만) |
| **성능**       | 느림              | 빠름              |
| **하이라이트 모양** | 날카로움            | 부드러움            |
| **물리적 정확도**  | 높음              | 약간 낮음           |
| **사용도**      | 고품질 렌더링         | 실시간 게임/앱        |
