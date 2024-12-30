
![[metal_rendering_pipeline.png]]

MTLDevice
- GPU에 대한 추상화된 인터페이스
- 리소스(버퍼, 텍스처 등) 생성을 담당
- 커맨드 큐 생성
- GPU 메모리 할당 관리
- 일반적으로 `MTLCreateSystemDefaultDevice()`로 생성

MTLCommandQueue
- GPU에 전송할 명령어들의 큐
- 커맨드 버퍼들을 순차적으로 관리
- 렌더링 명령을 GPU에 전달하는 파이프라인
- `device.makeCommandQueue()`로 생성

MTLRenderPipelineState
- 그래픽스 렌더링 파이프라인의 상태를 캡슐화
- 셰이더 프로그램, 버텍스 레이아웃, 블렌딩 모드 등 포함
- 렌더링 설정을 고정하여 성능 최적화

MTLBuffer
- GPU 메모리에 할당된 데이터 버퍼
- 버텍스 데이터, 변환 행렬 등을 저장
- CPU-GPU 간 데이터 전송에 사용

MTLDepthStencilState
- 깊이(Z) 테스트와 스텐실 테스트 설정을 관리
- 3D 렌더링에서 물체의 앞뒤 관계 처리

Camera
- 가상 카메라의 속성과 행렬을 관리하는 커스텀 클래스
- View와 Projection 행렬 계산

렌더링과정
1. `MTLDevice`로 필요한 리소스들 생성
2. `MTLBuffer`에 데이터 저장
3. `MTLRenderPipelineState`로 렌더링 파이프라인 설정
4. `MTLCommandQueue`를 통해 커맨드 버퍼 생성
5. 커맨드 버퍼에 렌더링 명령 인코딩
6. 커맨드 버퍼 커밋하여 GPU 실행


### 깊이 테스트를 하는 이유

```swift
// 1. 깊이 버퍼 포맷 설정 
// 32비트 부동소수점 형식의 깊이 버퍼 사용
metalView.depthStencilPixelFormat = .depth32Float


// 2. 깊이 테스트 설정
let depthDescriptor = MTLDepthStencilDescriptor()
// 새로운 픽셀의 깊이 값이 기존 깊이 값보다 작거나 같을 때만 렌더링
depthDescriptor.depthCompareFunction = .lessEqual  // 깊이 비교 함수
// 렌더링된 픽셀의 깊이값을 깊이 버퍼에 저장
// 왜 있을까? - 다음 렌더링 할 물체와의 깊이 비교
depthDescriptor.isDepthWriteEnabled = true         // 깊이 값 쓰기 활성화
```

이유: 3D 객체들의 앞뒤 관계를 올바르게 표현하기 위해서

### Camera 클래스


```swift
import Foundation
import MetalKit
import simd

class Camera {
    var position: SIMD3<Float>
    var target: SIMD3<Float>
    var up: SIMD3<Float> = [0, 1, 0]
    var aspect: Float = 1.0
    var fov: Float = Float.pi / 3
    var near: Float = 0.1
    var far: Float = 100
    
    init(position: SIMD3<Float>, target: SIMD3<Float>) {
        self.position = position
        self.target = target
    }
    
    var viewMatrix: float4x4 {
        // 시선 방향 계산
        let direction = normalize(target - position)
        let right = normalize(cross(direction, up))
        let newUp = normalize(cross(right, direction))
        
        let translation = float4x4(columns: (
            SIMD4<Float>(1, 0, 0, 0),
            SIMD4<Float>(0, 1, 0, 0),
            SIMD4<Float>(0, 0, 1, 0),
            SIMD4<Float>(-position.x, -position.y, -position.z, 1)
        ))
        
        let rotation = float4x4(columns: (
            SIMD4<Float>(right.x, newUp.x, -direction.x, 0),
            SIMD4<Float>(right.y, newUp.y, -direction.y, 0),
            SIMD4<Float>(right.z, newUp.z, -direction.z, 0),
            SIMD4<Float>(0, 0, 0, 1)
        ))
        
        return rotation * translation
    }
    
    var projectionMatrix: float4x4 {
        let y = 1 / tan(fov * 0.5)
        let x = y / aspect
        let z = far / (far - near)
        let w = -z * near
        
        return float4x4(columns: (
            SIMD4<Float>(x, 0, 0, 0),
            SIMD4<Float>(0, y, 0, 0),
            SIMD4<Float>(0, 0, z, 1),
            SIMD4<Float>(0, 0, w, 0)
        ))
    }
}
```

```
// SIMD3<Float>
- Single Instruction Multiple Data 벡터 타입
- 3D 공간의 x, y, z 좌표를 표현
- 고성능 벡터 연산을 위해 사용

// float4x4
- 4x4 부동소수점 행렬
- 3D 변환(이동, 회전, 크기 조절)을 표현
- 뷰와 투영 변환에 사용

// viewMatrix
- 월드 공간을 카메라 공간으로 변환
- 카메라의 위치와 방향을 기준으로 모든 객체의 위치 계산
- 구성: 회전 행렬 × 이동 행렬
- Translation: 물체들을 카메라 위치만큼 이동
- Rotation: 카메라 방향으로 회전

// projectionMatrix
- 3D 공간을 2D 화면으로 투영
- 원근감 표현 (가까운 물체는 크게, 먼 물체는 작게)
- fov, aspect, near, far 평면으로 계산
- 원근 투영을 위한 4x4 행렬
- fov와 aspect ratio로 시야 영역 정의
- near/far로 보이는 범위 제한

fov (시야각):
- 작은 값: 망원 렌즈 효과 (확대)
- 큰 값: 광각 렌즈 효과 (축소)

near/far 평면:
- near: 너무 가까운 물체 잘림
- far: 너무 먼 물체 잘림

aspect ratio:
- 화면 비율에 맞춰 이미지 왜곡 방지

```


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


### 메시 생성 

구를 그릴 때 지구본처럼 위도 경도를 이용해서 그림.

```
// 구면 좌표계에서 데카르트 좌표계로 변환하는 공식:
x = r * cos(θ) * cos(φ)
y = r * sin(θ)
z = r * cos(θ) * sin(φ)

여기서:
r = radius (골프공 반지름)
θ (theta) = 위도 (-π/2 ~ π/2)
φ (phi) = 경도 (0 ~ 2π)

// 1. 위도(latitude) 계산
let lat = Float.pi * (-0.5 + Float(i) / Float(segments))
/*
i = 0 일 때: -π/2 (-90°) 
i = segments/2 일 때: 0° (적도)
i = segments 일 때: π/2 (90°)
*/

// 2. y 좌표 계산
let y = radius * sin(lat)
/*
lat = -π/2 일 때: y = -radius (아래)
lat = 0 일 때: y = 0 (중간)
lat = π/2 일 때: y = radius (위)
*/

// 3. 현재 위도에서의 원의 반지름
let conLat = radius * cos(lat)
/*
lat = -π/2 일 때: conLat = 0 (점)
lat = 0 일 때: conLat = radius (가장 큰 원)
lat = π/2 일 때: conLat = 0 (점)
*/


// 경도에 따른 x,z 좌표 계산
let lng = 2 * Float.pi * Float(j) / Float(segments)
let x = conLat * cos(lng)
let z = conLat * sin(lng)

/*
j = 0 일 때: (x,z) = (conLat, 0)     [0°]
j = segments/4 일 때: (x,z) = (0, conLat)   [90°]
j = segments/2 일 때: (x,z) = (-conLat, 0)  [180°]
j = 3*segments/4 일 때: (x,z) = (0, -conLat) [270°]
j = segments 일 때: (x,z) = (conLat, 0)     [360°]
*/

```

```cpp
import MetalKit

class GolfBallRenderer: NSObject, MTKViewDelegate {    
    // MARK: - Geometry Creation
    static func createGeometry(device: MTLDevice) -> (MTLBuffer?, MTLBuffer?) {
        // 골프공 메시 데이터 생성 (UV 구)
        var ballVertices: [Float] = []
        let segments = 32
        let radius: Float = 0.0213 // 실제 골프공 크기 (미터)
        
        for i in 0...segments {
            let lat = Float.pi * (-0.5 + Float(i) / Float(segments))
            let y = radius * sin(lat)
            let cosLat = radius * cos(lat)
            
            for j in 0...segments {
                let lng = 2 * Float.pi * Float(j) / Float(segments)
                let x = cosLat * cos(lng)
                let z = cosLat * sin(lng)
                
                // 위치
                ballVertices.append(x)
                ballVertices.append(y)
                ballVertices.append(z)
                
                // 법선 벡터
                ballVertices.append(x/radius)
                ballVertices.append(y/radius)
                ballVertices.append(z/radius)
            }
        }
        // 지면 메시 데이터 생성
        let groundVertices: [Float] = [
            -50, 0, -50,  0, 1, 0,
             50, 0, -50,  0, 1, 0,
             50, 0,  50,  0, 1, 0,
            -50, 0, -50,  0, 1, 0,
             50, 0,  50,  0, 1, 0,
            -50, 0,  50,  0, 1, 0
        ]
        guard let ballBuffer = device.makeBuffer(bytes: ballVertices,
                                               length: ballVertices.count * MemoryLayout<Float>.stride,
                                               options: []),
              let groundBuffer = device.makeBuffer(bytes: groundVertices,
                                                 length: groundVertices.count * MemoryLayout<Float>.stride,
                                                 options: []) else {
            return (nil, nil)
        }
        
        return (ballBuffer, groundBuffer)
    }
}
```


### 렌더링 파이프라인


```swift
let library = device.makeDefaultLibrary() 
let vertexFunction = library.makeFunction(name: "vertexShader") 
let fragmentFunction = library.makeFunction(name: "fragmentShader")
```

- 컴파일된 셰이더 코드를 로드
- 버텍스와 프래그먼트 셰이더 함수 참조 획득

```swift
let pipelineDescriptor = MTLRenderPipelineDescriptor()
pipelineDescriptor.vertexFunction = vertexFunction
pipelineDescriptor.fragmentFunction = fragmentFunction
pipelineDescriptor.colorAttachments[0].pixelFormat = metalView.colorPixelFormat
```

- 렌더링 파이프라인 구성 설정
- 셰이더 함수 연결
- 색상 출력 포맷 지정