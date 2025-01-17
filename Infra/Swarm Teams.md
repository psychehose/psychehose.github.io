
Swarm에 있는 모듈을 만들어야함.


- **module.config.php**

    - Swarm이 모듈을 인식하도록 하는 설정 파일
    - PHP 배열 형태로 이벤트 리스너 바인딩, 서비스 등록 등을 설정합니다.
      
- **Module.php** (이름은 꼭 ‘Module’일 필요는 없지만 일반적으로 많이 사용)
    - Swarm에서 제공하는 다양한 Hook(Listener)나 Config 설정을 등록하고, 실제 기능(웹훅 전송 로직 등)을 구현하는 파일입니다.
