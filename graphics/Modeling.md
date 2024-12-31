수학을 배운 경험이 있다면 x^2  + y^2 = 1 형태의 식을 본 적이 있을 것이다. 이러한 형태를 음함수라고 하는데 GPU는 음함수를 잘 처리하지 못한다. 그러면 어떻게 하느냐? 평면의 점을 샘플링해서 Polygon Mesh로 만든다. (샘플링을 한다는 것은 정점과, 법선 벡터를 잘 뽑는 것을 의미한다.)

어떤 물체가 있고 이를 잘 샘플링해서 폴리곤 메쉬로 만들었다. 이렇게 만들어진 메쉬를 컴퓨터(?)는 어떻게 저장을 하는 지 알아보자.

![](https://blog.kakaocdn.net/dn/YrGkD/btstmtNqVn8/aYD2GrUzEEMsCpUlJ2Ws71/img.png)

t1, t2, t3라는 삼각형 메쉬가 존재한다. 이를 메모리에 저장을 하는 방법은 간단하다. 좌표를 그냥 배열에 때려넣으면 된다. 때려 넣고 나니 문제점이 있는 것 같다.  vertex array를 보면 중복되는 것이 많다는 것을 알 수 있다. 낭비가 심하다. 그래서 위와 같이 저장하지 않는다. 문제를 해결하기 위해 인덱스를 추가해 보자.

![](https://blog.kakaocdn.net/dn/bB6wFv/btstk41D2xI/WHrJSexTiosKNTGHhMQe40/img.png)

각 정점들에 index를 줘서 해결하면 더 빠르게 처리할 수 있다.

#### Export  Data

3ds Max와 같은 모델링 프로그램을 이용해서 export를 하면 .obj 파일을 얻을 수 있다. 간단한 구를 모델링해서 export를 해서 열면 어떤 데이터가 들어 있을까?

구는 26개의 정점과 48개의 삼각형으로 이뤄져 있다.

![](https://blog.kakaocdn.net/dn/d3DpsC/btstkTy9Ufk/3TFup6zSfeHakZy5345wc1/img.png)

v는 Vertex를 의미하고, 숫자는 순서대로 x, y, z이다.

vn는 Vertex Noraml을 의미하고 숫자는 순서대로 x, y, z이다.

f는 face를 의미하고 v // vn 을 의미한다. (구는 v와 vn이 1:1로 대응하지만, 직육면체와 같은 입체에서는 vn이 중복될 수 있다.)

#### Import

![](https://blog.kakaocdn.net/dn/1Vyde/btsth6yLCPq/GbVkDxpfn73pFLdUo44HZ1/img.png)

.obj를 Import 하게 되면 메모리는 위와 같이 저장된다.

#### Ref.
http://www.kocw.net/home/search/kemView.do?kemId=1349173