# OpenGL 깊이 버퍼와 렌더링 순서

## 1. 깊이 버퍼(Z-buffer) 기본 개념

깊이 버퍼는 3D 그래픽스에서 어떤 물체가 다른 물체를 가리는지 결정하는 핵심 메커니즘입니다.

- **작동 원리**: 각 픽셀마다 카메라로부터의 거리(Z값)를 저장
- **범위**: 일반적으로 [0.0, 1.0] (0.0은 near plane, 1.0은 far plane)
- **정밀도**: 일반적으로 16비트, 24비트 또는 32비트 (24비트가 가장 일반적)
- **분포**: 비선형적 분포 - 가까운 거리에서 정밀도가 높고, 먼 거리에서는 정밀도가 낮음

## 2. 깊이 버퍼 설정 방법

### 기본 설정
```cpp
// 깊이 테스트 활성화
glEnable(GL_DEPTH_TEST);

// 깊이 테스트 비교 함수 설정 (기본값은 GL_LESS)
glDepthFunc(GL_LESS);  // 새 픽셀의 깊이 값이 작을 때(더 가까울 때) 통과
```

### 깊이 버퍼 초기화
```cpp
// 렌더링 루프에서 매 프레임마다 색상 버퍼와 깊이 버퍼를 함께 초기화
glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
```

### 깊이 버퍼 쓰기 제어
```cpp
// 깊이 버퍼 쓰기 활성화 (기본값)
glDepthMask(GL_TRUE);

// 깊이 버퍼 쓰기 비활성화 (투명 오브젝트 그릴 때 유용)
glDepthMask(GL_FALSE);
```

### 깊이 비교 함수 옵션
- `GL_LESS`: 새 값이 작을 때 통과 (기본값)
- `GL_LEQUAL`: 새 값이 작거나 같을 때 통과
- `GL_GREATER`: 새 값이 클 때 통과
- `GL_GEQUAL`: 새 값이 크거나 같을 때 통과
- `GL_EQUAL`: 새 값이 같을 때만 통과
- `GL_NOTEQUAL`: 새 값이 다를 때 통과
- `GL_ALWAYS`: 항상 통과
- `GL_NEVER`: 절대 통과하지 않음

## 3. 깊이 버퍼 정밀도와 Z-fighting

Z-fighting은 두 물체가 거의 같은 깊이에 있을 때 깜빡이는 현상입니다.

### 원인
- 깊이 버퍼의 제한된 정밀도
- near plane과 far plane 사이의 거리가 너무 큰 경우
- 특히 먼 거리에서 정밀도가 낮아지는 비선형 분포로 인해 발생

### 해결 방법
- **near/far plane 비율 줄이기**: 시야 범위를 필요한 만큼만 설정
  ```cpp
  // 좋은 예시: near와 far 사이 비율을 합리적으로 유지
  glm::mat4 projection = glm::perspective(glm::radians(45.0f), aspect_ratio, 0.1f, 100.0f);
  
  // 피해야 할 예시: 비율이 너무 큼
  // glm::mat4 projection = glm::perspective(glm::radians(45.0f), aspect_ratio, 0.001f, 10000.0f);
  ```
- **높은 정밀도의 깊이 버퍼 사용**: 24비트 또는 32비트 깊이 버퍼 사용
- **물체 배치 조정**: 서로 다른 물체가 완전히 같은 깊이에 있지 않도록 약간 간격 두기

## 4. 투명 오브젝트 렌더링

투명 오브젝트는 일반적인 깊이 테스트만으로는 올바르게 표시되지 않습니다.

### 올바른 방법
1. **불투명 오브젝트 먼저 그리기**: 깊이 테스트와 깊이 쓰기를 활성화한 상태로 그림
2. **깊이 버퍼 쓰기 비활성화**: 투명 오브젝트를 그리기 전에 설정
   ```cpp
   glDepthMask(GL_FALSE);
   ```
3. **투명 오브젝트 정렬**: 먼 것부터 가까운 것 순으로 정렬
   ```cpp
   std::sort(transparent_objects.begin(), transparent_objects.end(), 
             [camera_pos](const Object& a, const Object& b) {
                 float dist_a = glm::distance(camera_pos, a.getPosition());
                 float dist_b = glm::distance(camera_pos, b.getPosition());
                 return dist_a > dist_b; // 내림차순 정렬 (먼 것부터)
             });
   ```
4. **블렌딩 활성화**: 투명도를 위한 블렌딩 설정
   ```cpp
   glEnable(GL_BLEND);
   glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);
   ```
5. **정렬된 순서로 투명 오브젝트 그리기**
6. **원래 상태로 복원**:
   ```cpp
   glDepthMask(GL_TRUE);
   glDisable(GL_BLEND);
   ```

## 5. 실제 렌더링 루프 예시

```cpp
void renderScene() {
    // 버퍼 초기화
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
    
    // 뷰와 투영 행렬 설정
    glm::mat4 view = camera.getViewMatrix();
    glm::mat4 projection = glm::perspective(glm::radians(45.0f), 
                                          windowWidth / windowHeight, 
                                          0.1f, 100.0f);
    
    // 셰이더에 행렬 전달
    shader.use();
    shader.setMat4("view", view);
    shader.setMat4("projection", projection);
    
    // 불투명 오브젝트 렌더링
    glEnable(GL_DEPTH_TEST);
    glDepthFunc(GL_LESS);
    glDepthMask(GL_TRUE);
    
    for (const auto& obj : opaqueObjects) {
        // 모델 행렬 설정
        glm::mat4 model = obj.getModelMatrix();
        shader.setMat4("model", model);
        
        // 오브젝트 그리기
        obj.draw();
    }
    
    // 투명 오브젝트 렌더링
    glEnable(GL_BLEND);
    glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);
    glDepthMask(GL_FALSE);  // 깊이 버퍼 쓰기 비활성화
    
    // 투명 오브젝트는 거리 기준으로 정렬 (먼 것부터)
    sortTransparentObjects();
    
    for (const auto& obj : transparentObjects) {
        // 모델 행렬 설정
        glm::mat4 model = obj.getModelMatrix();
        shader.setMat4("model", model);
        
        // 투명 오브젝트 그리기
        obj.draw();
    }
    
    // 원래 상태로 복원
    glDepthMask(GL_TRUE);
    glDisable(GL_BLEND);
}
```

## 6. 참고 사항

- 깊이 버퍼는 불투명 오브젝트의 경우 그리는 순서에 관계없이 올바른 결과를 제공
- 투명 오브젝트는 특별한 처리가 필요하며, 정렬이 중요함
- Z-fighting을 피하기 위해 near/far 비율을 적절히 유지해야 함
- 깊이 테스트와 깊이 버퍼 쓰기는 별도의 기능이므로 각각 설정 필요

---
#opengl #graphics #depth-buffer #transparency #rendering
