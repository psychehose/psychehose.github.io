
#### Ephemeral Wifi

* Android 10부터 도입된 WifiNetworkSpecifier를 이용한 Wifi는 보통 Ephemeral임.
* 안드로이드 OS는 네트워크로 인터넷이 안될 것 같다고 판단하면 우선순위 낮춤 -> 이 네트워크를 기본 라우트로 삼지 않게 됨

#### 기본 라우트가 아니면 TCP 연결이 실패하는 이유

* 기본 라우트가 아니면 OS는 패킷을 LTE나 기존 Wi-fi 쪽으로 보내려고 시도
* 즉 에페메럴 Wifi로는 도달 불가능 -> TCP 연결 실패

`bindProcessToNetwork(network)`를 통해서 앱 전체 프로세스가 내보내는 트래픽은 네트워크로 보내도록 설정 -> TCP 연결 성공

소켓 단위 바인딩 (bindSocket)을 이용하면 다른 통신은 기존 네트워크를 그대로 쓰면서, 특정 소켓은에페메럴 Wi-fi를 이용할 수 있음

