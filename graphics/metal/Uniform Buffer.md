
### Uniform Buffer

- CPU와 GPU 간의 효율적인 데이터 공유
- 동적으로 업데이트되는 데이터 처리에 적합
- 메모리 정렬과 크기 관리 중요
- 적절한 에러 처리 필요

```swift
device.makeBuffer(
    length: MemoryLayout<Uniforms>.size,  // 버퍼 크기
    options: .storageModeShared           // 메모리 저장 모드
)
```

```swift
// MTLDevice의 메서드
func makeBuffer(
    length: Int,        // 버퍼 크기 (바이트)
    options: MTLResourceOptions  // 메모리 관리 옵션
) -> MTLBuffer?  // 생성된 Metal 버퍼 반환
```

```
// 가능한 옵션들:
.storageModeShared    // CPU와 GPU가 모두 접근 가능
.storageModePrivate   // GPU만 접근 가능
.storageModeManaged   // CPU와 GPU 각각 별도 복사본 관리

// .storageModeShared 사용 시
CPU (Swift) ⟷ 공유 메모리 ⟷ GPU (Metal)
- 양방향 직접 접근 가능
- 동기화 오버헤드 최소화
- 작은 크기의 자주 업데이트되는 데이터에 적합

```

```swift

// Uniform 버퍼 생성
let uniformBuffer = device.makeBuffer(
    length: MemoryLayout<Uniforms>.size,
    options: .storageModeShared
)

// 데이터 업데이트
let uniforms = Uniforms(
    modelMatrix: modelMatrix,
    viewMatrix: camera.viewMatrix,
    projectionMatrix: camera.projectionMatrix
)
memcpy(uniformBuffer.contents(), &uniforms, MemoryLayout<Uniforms>.size)
```
