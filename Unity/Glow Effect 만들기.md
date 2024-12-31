게임을 만들어보고 싶어서, 요즘 유니티를 공부하고 있어요.

게임을 하다 보면 반짝거리는 물체들이 많이 있는데, 이 반짝거리는 효과를 어떻게 만드는지 공부해 보고 정리해 봤습니다. 

Glow Effect를 만들기 위해서 4가지가 성립해야 합니다.

1. Shader - Emission에서 Color가 검은색이 아니고, Intensity > 0 이어야 합니다.
2. Post Processing Bloom (Intensity와 Threshold 세팅)
3. Render Pipeline Asset에서 HDR 체크
4. Camera에서 Rendering - Post Processing 체크

#### 1. Shader - Emission에서 Color가 검은색이 아니고, Intensity > 0

먼저 3D Object 구를 만든 다음에 이 구에 적용한 Material을 만듭시다.

![](https://blog.kakaocdn.net/dn/ba0EHV/btsnF4MPmE9/aTlmEzy2uGUFIFEi65BrL1/img.png)![](https://blog.kakaocdn.net/dn/digjIe/btsnTEeToHv/SEV8Xa8MbugddHdz7EcIT0/img.png)

만든 Material을 구에 드래그 앤 드랍 해줍시다. 구 오브젝트를 선택하고 오른쪽 Inspector에서 Surface Inputs - Emission을 체크하고 색깔(검은색 X)을 지정하고 intensity를 0보다 높게 설정하면 1번 과정은 끝입니다. 그러면 구에 색깔이 있을 거예요.

intensity를 엄청 높여도 Render Pipeline Asset이 HDR 옵션이 해제되어 있다면 Glow Effect는 나타나지 않을 거예요. 만약 나타난다면 HDR 옵션이 체크되어 있는 겁니다.

![](https://blog.kakaocdn.net/dn/rYlDe/btsnTDfYB5N/KTQwg2cdfH6vEaKk9GTnR0/img.png)

#### 2. Post Processing Bloom (Intensity와 Threshold 세팅)

이제 Post Processing 옵션을 가지고 있는 Object를 만들 거예요. Create Empty 하고 이름을 변경합시다. 그런 다음에 아래처럼 Add Componet를 눌러서 Volume을 추가합시다.

![](https://blog.kakaocdn.net/dn/cAcJUG/btsnSXFAZ5U/Uirq8dXFSglHSBkO8vArnK/img.png)![](https://blog.kakaocdn.net/dn/ME19w/btsnFK8UrKr/uTAOWh6GYcHtVmySaBjv61/img.png)

![](https://blog.kakaocdn.net/dn/EykhQ/btsnEXAV6i8/qMLIiOvafOxKXXFoe67ye1/img.png)

Volume에서 Profile 칸이 아마 비워져 있을 텐데, New를 클릭해서 새로 생성한 뒤에, Add Override를 눌러서 Post Processing - Bloom을 클릭합니다. 그리고 Threshold를 설정하고 Intensity를 설정하면 됩니다. 이때 Threshold는 최솟값이에요. Intensity보다 작아야 작동합니다. Intensity는 Glow Effect의 강함입니다. 저는 Intensity를 1로 설정하겠습니다. 아직도 역시 효과가 없을 거예요!

#### Render Pipeline Asset에서 HDR 체크

이제 URP에서 HDR 옵션을 체크해 주면 Glow Effect가 적용됩니다.

![](https://blog.kakaocdn.net/dn/yWAiA/btsnF6Ya6uR/KmEDRHvcznviMUYwbWb2fK/img.png)

#### Camera에서 Rendering - Post Processing 체크

근데 한 가지 문제점이 있는데 씬 뷰에서는 적용되는데 게임 뷰에서 적용이 안될 수도 있습니다. (아래사진 왼쪽). Camera를 클릭해서 Rendering - Post Processing을 체크하면 게임 뷰에서도 적용(아래사진 오른쪽)됩니다.

![](https://blog.kakaocdn.net/dn/bFAcJS/btsnLvJE7ic/KLNSIRJlE4hBxymlgALJVK/img.png)![](https://blog.kakaocdn.net/dn/bg7pfi/btsnHRsEeVi/k0CzMQSMCyd2nV873aYKqK/img.png)

#### 마무리

단순 Glow Effect 뿐만 아니라 더 다양한 Effect를 적용하고 싶을 때 (예를 들면 캐릭터를 클릭했을 때 외곽선 하이라이팅 하고 싶은 경우)는 Unity에서 제공하는 Shader Graph를 사용하면 된다고 하네요. 다음에 기회가 되면 다뤄보도록 하겠습니다~

#### Ref.

 [https://www.youtube.com/watch?v=bkPe1hxOmbI](https://www.youtube.com/watch?v=bkPe1hxOmbI)