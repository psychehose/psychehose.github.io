이번에는 유니티를 이용해 WASD로 움직이는 캐릭터를 만들어봅시다.

Unity Project 3D URP로 생성해요.

![](https://blog.kakaocdn.net/dn/LQB5d/btsn6JuOaJp/TvEG2Tak3NZ7KqEo55x7f0/img.png)

공허한 화면을 뒤로한 채 Hierarchy에서 오른쪽 클릭 후 3D Object -> Plane을 눌러서 생성합시다. 이름을 Floor로 해줄게요

그런 다음에, Inspector 창에서, Scale을 5, 5, 5로 Position과 Rotation을 0, 0, 0으로 설정합니다.

![](https://blog.kakaocdn.net/dn/bgttrZ/btsn6GSp3Nd/jRc4YxJMqApLlhmuq8I6O0/img.png)

하이어러키 창에서 메인 카메라를 클릭한 다음 인스펙터창에서 Position, Rotation, Scale을 아래처럼 설정합니다.

![](https://blog.kakaocdn.net/dn/AjHWa/btsn9Tizkle/7XvsKtRoy95S7IjBmF8cd1/img.png)

하이어러키 창에서 Create Empty를 만들고 이름을 Player로 합시다. 그리고 만든 Player 하위에 3D Object - Capsule을 생성합니다. 이렇게 하는 이유는 뷰 로직과 동작 로직을 분리하기 위해서예요.

Player 오브젝트의 Position과 Rotation은 0,0,0 Scale은 1,1,1으로 설정하고 PlayerVisual(하위에 만든 캡슐)은 Postion (0,1,0) Rotation은 0,0,0, Scale은 1,1,1로 설정해 주세요.

![](https://blog.kakaocdn.net/dn/RqMrq/btsn8pI3tFB/SnaoJYXV9i7IINUlK9jrO1/img.png)![](https://blog.kakaocdn.net/dn/cBgb5p/btsn7trHo5k/ibnoMMkaaZStsV2b9Jpaek/img.png)

Project 창에서 Asset 폴더 아래에 Script 폴더를 만들고 그곳에 C# Script를 만들고 파일을 Player 오브젝트에 드래그 앤 드롭합니다.

그런 다음에 Asset - Open C# Project를 클릭합니다.

![](https://blog.kakaocdn.net/dn/TaZwr/btsn8pI31yr/M7jSrnckR44FD7Tkg4yU70/img.png)

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Player : MonoBehaviour { 

    [SerializeField] private float moveSpeed = 7f;
    private void Update() {
        Vector2 inputVector = new Vector2(0, 0);

        if (Input.GetKey(KeyCode.W)) {
            inputVector.y += 1;
        }
        if (Input.GetKey(KeyCode.A)) {
            inputVector.x -= 1;
        }
        if (Input.GetKey(KeyCode.S)) {
            inputVector.y -= 1;
        }
        if (Input.GetKey(KeyCode.D)) {
            inputVector.x += 1;
        }

       inputVector = inputVector.normalized;

       Vector3 moveDir = new Vector3(inputVector.x, 0, inputVector.y);

       transform.position += moveDir * moveSpeed * Time.deltaTime;

    }
}
```

좌표 시스템이 평소 학교에서 배웠던 거랑 일치하지 않을 수 있어요. 그리고 게임 엔진마다 축이 조금씩 상이하더라고요.  제가 공부할 때는 Right Handed를 많이 사용했는데, 유니티는 Left Handed라고 합니다. 아래 그림 참고 하시면 될 것 같아요.

![](https://blog.kakaocdn.net/dn/TIRvR/btsn8lfDOKD/RL0AI3d1kyNgGDFcfsrZSk/img.png)

코드 자체는 별 게 없는 것 같아요. 조금 특징적인 것에 대해서만 알면 될 것 같습니다.

```
inputVector = inputVector.normalized;
```

왜 inputVector를 정규화를 할까요? 왜냐하면, Vector는 속력이 아닌 속도를 나타내기 때문입니다. 만약 정규화를 하지 않았을 때, 대각선으로 이동한다면 inputVector의 값은 (1,1)이 될 거예요. 그러면 벡터의 길이 (스칼라) 값은 1.414...이 되기 때문에 대각선으로 이동할 때 더 빠른 속도로 이동하게 됩니다. 즉 동일한 속도가 보장이 되지 않게 됩니다. 이러한 이유로 정규화를 통해서 방향만을 얻고, 이동 속도는 변수로 곱해서 처리하는 것이 일반적인 것 같습니다.

```
transform.position += moveDir * moveSpeed * Time.deltaTime;
```

위 코드에서 속력과 방향에 Time.deltaTime를 곱하고 있습니다. 왜 그럴까요? 이것을 알기 위해서는 Frame(프레임)과 FPS에 대해 먼저 알아야 합니다.

동영상은 정지된 여러 장의 사진으로 구성되어 있고, 우리는 이 사진이 빠르게 변하는 것을 움직이는 것으로 보게 됩니다. 여기에서 이 정지된 사진을 프레임이라고 합니다. FPS는 Frame Per seconds의 준말로 1초당 몇 장의 사진이 변하는가입니다. 60 FPS면 1초당 연속하는 60장의 사진을 보는 것이라고 이해하면 될 것 같아요. 그래서 FPS는 성능에 비례합니다. (고사양 컴퓨터면 더 높고, 더 낮은 사양이면 더 낮습니다.)

유니티에서 Update() 메서드는 1 프레임을 주기로 호출됩니다. 만약 위의 코드에서 Time.deltaTime이 없다면, 어떻게 될까요? 고사양의 컴퓨터를 가진 사람은 더 빨리 이동할 것입니다. 저사양의 컴퓨터를 가진 사람은 더 느리게 이동할 테고요. 이러한 차이를 보정하기 위해서 바로 Time.deltaTime을 곱하는 것입니다. 

유니티에서 Time.deltaTime의 정의는 **The interval in seconds from the last frame to the current one입니다.** 즉 현재 프레임과 이전 프레임의 시간 간격을 의미하게 되는 거예요. 따라서 고사양의 컴퓨터는 deltaTime의 값은 작을 테고, 저사양의 컴퓨터의 값은 더 높게 됩니다. 

고사양 컴퓨터는 60 fps가 나오고, 저사양의 게임은 30fps가 나온다고 가정하면, 아래와 같이 1 프레임당 시간 간격, 즉 deltaTime을 구할 수 있습니다.

30fps = 1 / 30 = 0.033333333

60fps = 1 / 60 = 0.01666666

이때 Update 코드를 생각해봅시다. Update 코드는 1 프레임에 한번 호출된다고 했으니까, 

저사양 컴퓨터: (moveDir * moveSpeed * 0.3333333) * 30 만큼 이동 (30프레임)

고사양 컴퓨터: (moveDir * moveSpeed * 0.016666666) * 60 만큼 이동 (60프레임)의 값은 같게 됩니다.

그래서, Time.deltaTime로 컴퓨터 사양에 따른 이동을 보정할 수 있게 됩니다.

#### 마무리

캐릭터 이동을 구현할 때, GetKey를 이용해 구현하는 것보다, Input System이라는 package를 이용해서 구현하는 것이 확장하기 더 좋아요. 절대 Input System Package를 이용하세요!

#### Github

https://github.com/psychehose/Ex_Collision_Detection

#### Ref

[https://m.blog.naver.com/destiny9720/221411002215?view=img_1](https://m.blog.naver.com/destiny9720/221411002215?view=img_1) 

[https://ko.khanacademy.org/computing/computer-programming/programming-natural-simulations/programming-vectors/a/vector-magnitude-normalization](https://ko.khanacademy.org/computing/computer-programming/programming-natural-simulations/programming-vectors/a/vector-magnitude-normalization)

[https://windeva.tistory.com/840](https://windeva.tistory.com/840)