### Overview

언리얼 엔진에서 PSO(Pipeline State Object)는 그래픽 렌더링 파이프라인의 상태를 정의하는 객체. PSO는 주로 Direct3D 12 및 Vulkan 같은 저수준 그래픽 API에서 사용되며, 그래픽 파이프라인의 다양한 설정(셰이더, 블렌딩)을 캡슐화합니다. 이를 통해 렌더링을 효율적으로 관리할 수 있습니다.

### PSO 기능

- PSO는 그래픽 파이프라인 상태를 한 번에 정의하고 이를 캐시할 수 있습니다. 이는 매 프레임마다 상태를 변경하는 것보다 훨씬 효율적입니다.
    
- PSO는 다양한 파이프라인 상태를 하나의 객체로 관리할 수 있어 상태 변경 관리가 단순해집니다.
    
- Unreal Engine은 이러한 저수준 API들을 추상화하여 PSO를 통해 다양한 플랫폼에서 최적화된 렌더링을 제공합니다.
    

### PSO 데이터 수집 목적

PSO 데이터 수집 목적은 성능 최적화, 셰이더 컴파일 시간 단축, 일관된 성능 제공, 런타임 프레임 드롭 방지, 디버깅 및 최적화에 있다. PSO를 미리 컴파일하여 필요할 때 빠르게 사용할 수 있어 런타임 성능을 향상시킵니다. 사전 정의된 PSO를 사용하면 상태 설정 과정에서 발생할 수 있는 불필요한 연산을 피할 수 있습니다.

### PSO 데이터 수집 방법 In Golfzon M

1. PSO는 패키징 설정을 Development로 바꾸고 Share Material Shader Code 체크
    
    1. PSO 데이터 수집을 위해서는 디버깅 정보를 포함해서 패키징 해야하므로 Dev 빌드
        
    2. 여러 머터리얼이 공통으로 사용하는 셰이더 코드를 공유하도록 함 → 중복된 셰이더 제거 → 메모리 효율 증가  
        
2. 맵 포함 패키징
    
    1. 맵에서 사용하는 모든 셰이더와 랜더링 상태를 로드하고 기록하기 위해 맵 전체 포함
        

3. .shk 파일을 백업
    
    1. 셰이더 키 캐시(Shader Key Cache) 파일
        
    2. 파일은 셰이더 컴파일 과정에서 생성되며, 특정 셰이더의 키 정보를 저장합니다. 이 키 정보는 셰이더가 변경되었는지 여부를 확인하고, 캐싱된 버전을 사용할 수 있는지를 판단
        
    3. 셰이더 키 캐시를 통해 게임 실행 시 셰이더 로딩 시간을 줄일 수 있음  
        
4. 위에서 뽑은 맵을 이용해서 PSO 데이터 수집
    
    1. Android, iOS 각 플랫폼마다 게임을 플레이하면서 PSO 데이터를 수집  
        
5. 데이터 수집한 디바이스에서 .upipelinecache 확장자명을 가진 파일 추출
    
    1. PSO 데이터가 저장된 .upipelinecache  
        
6. 백업한 파일과 upipelinecache 확장자명을 가진 파일을 이용해서 csv 파일로 추출 (with 최승인 프로)
    
    1. .shk 키와 .upipelinecache 데이터를 이용해서 PSO 데이터를 텍스트 형식으로 저장해 분석/수정 가능하게 수정
       
7. 이를 {Project}/Build/{Platform}/PipelineCaches에 넣고, Shipping, Share Material Shader Code를 체크 해제하고 해당 맵을 포함한채로 패키징
    
    1. 해당 경로에 CSV 배치해두면 패키징 과정에서 해당 파일 참조하여 PSO 데이터 포함하게 된다.



### Detail 5, 6, 7

5. 모바일 디바이스를 PC에 연결하고 앱 Saved/CollectedPSOs 위치에 아래와 같은 파일들이 있다.

![[collectpso.png]]

6. 셰이더 키 캐시 파일(.shk)과 upipelinecache 파일을 한 폴더에 넣고 UE4Editor-Cmd를 이용해서 csv 파일을 추출한다.

![[pso_data.png]]

```
cd {your_engine folder}/Engine/Binaries/Win64


.\UE4Editor-Cmd.exe D:\Develop\WaveM\U2Client_Wave\U2Client.uproject -run=ShaderPipelineCacheTools expand C:\Users\psyche95\Desktop\WM_Backup\pso_2.1.1_backup\psodata\*.rec.upipelinecache C:\Users\psyche95\Desktop\WM_Backup\pso_2.1.1_backup\psodata\*.shk U2Client_SF_METAL.stablepc.csv 

```


### 안드로이드 PSO 데이터 수집시 주의점

#### Overview

안드로이드는 GPU에 따라 OpenGL, 또는 Vulkan을 사용할 수 있다. (둘 다 지원할 수도 있음) 그렇기 때문에 안드로이드는 PSO 수집을 두 번 해야 한다.

안드로이드 신버전은 앱에서 파일을 뽑는 것을 보안상 막아놓았기 때문에 구버전 (현재 가지고 있는 기기 s9, note9) 으로 진행한다.

노트9, s9의 GPU는 Mali_72이고, OpenGL, Vulkan을 지원한다.

따라서 Mali_72를 이용해서 OpenGL, Vulkan PSO 데이터를 수집하면 된다.
```

[Android_Mali_G72 DeviceProfile] 
DeviceType=Android  
BaseProfileName=Android_High; enable Vulkan on Android 9 and up, older versions crash on creating PSO with a compute shader that uses texel_buffer (eye adaptation)  
+CVars=r.Android.DisableVulkanSupport=0
+CVars=r.Vulkan.RobustBufferAccess=1  
+CVars=r.Vulkan.DescriptorSetLayoutMode=2  
+CVars=r.DefaultBackBufferPixelFormat=0  
+CVars=r.Vulkan.RayTracing.AllowCompaction=0  
+CVars=r.Vulkan.RayTracing.TLASPreferFastTraceTLAS=0
```

위의 코드가 들어가 있으면 Mali_72 GPU는 Vulkan API를 사용한다는 뜻이다. → 불칸용 PSO 데이터 수집

위의 코드가 들어가 있지 않으면 Mali_72 GPU는 OpenGL API를 사용한다는 뜻이다 → OpenGL PSO 데이터 수집

Saved/Cooked/Android_ASTC/{GameProject}/Metadata/PipelineCaches에 .shk 파일이 생김. 이를 백업해놓아야함.