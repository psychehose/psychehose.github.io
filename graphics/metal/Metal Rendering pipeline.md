
## Metal Rendering pipeline

1. 초기화 
	- Device 생성
	- CommandQueue 생성

2.  자원 생성
	* Vertex Buffer 생성
	* Texture Descriptor 생성

3. 파이프라인 상태 객체 생성
	* 셰이더 함수 로드
	* 렌더 파이프라인 상태 생성 (RenderPipelineState)

4. 명령 인코딩
	* CommandBuffer 생성
	* RednerPassDescriptor 생성
	* 렌더 인코더 생성
	* 파이프라인 상태 설정
	* 자원 설정
	* 그리기 명령
	* 인코딩 종료

5. 실행 및 표시
	* 명령 버퍼 커밋


## Metal Rendering pipeline flow

1. 앱 초기화 (단 한번 수행)
	* MTLDevice 생성
	* MTLCommandQueue 생성
	* 셰이더 컴파일 및 MTLRenderPipelineState 생성
	* 정적 자원 (버퍼, 텍스쳐) 생성

2. 프레임 마다 수행
	* MTLCommandBuffer 생성
	* MTLRenderPassDescriptor 설정
	* MTLRenderCommandEncoder 생성
	* 렌더링 상태 및 자원 설정
	* 그리기 명령 인코딩
	* 인코딩 종료
	* 명령 버퍼 커밋 및 실행

