## Media

MediaPlayer, MediaSource, MediaTexture, UImage, UMaterialInstanceDynamic 관계

* MediaSource
	* 역할: 미디어 데이터의 원천을 정의
	* UFileMediaSource, UStreamMediaSource 등 부모 클래스
	* 관계
		* MediaPlayer와 직접 연결
	    * 다른 컴포넌트들과는 직접적인 관계가 없음.

* MediaPlayer
	* 역할: 미디어를 재생하고 제어합니다.
	* 관계
		* MediaSource로부터 데이터를 받아 디코딩함.
		* MediaTexture에 디코딩된 비디오 프레임을 제공.
		* 전체 시스템의 중심 역할

* MediaTexture
	* 역할: MediaPlayer가 제공하는 비디오 프레임을 텍스처로 변환
	* 관계
		* MediaPlayer로부터 비디오 프레임을 받음
		* UMaterialInstanceDynamic의 텍스처 파라미터로 사용됨


* UMaterialInstanceDynamic
	* 역할: 동적으로 변경 가능한 머티리얼 인스턴스를 제공
	* 관계
		* MediaTexture를 텍스처 파라미터로 사용
		* UImage의 브러시로 사용됨

* UImage
	* 역할: UI에 이미지나 비디오를 표시
	* 관계
		* UMaterialInstanceDynamic을 브러시로 사용하여 비디오를 표시

