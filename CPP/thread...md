프로세스끼리 메모리 공유하지 않음.

쓰레드: 프로그램 내에서 동시에 실행될 수 있는 작은 실행 단위임

쓰레드는 같은 프로세스 내에 있으면 같은 메모리 공유


#### c++에서 쓰레드 생성하는 방법

```cpp
#include <iostream>
#include <thread>
using std::thread;

void func1() {
  for (int i = 0; i < 10; i++) {
    std::cout << "쓰레드 1 작동중! \n";
  }
}

void func2() {
  for (int i = 0; i < 10; i++) {
    std::cout << "쓰레드 2 작동중! \n";
  }
}

void func3() {
  for (int i = 0; i < 10; i++) {
    std::cout << "쓰레드 3 작동중! \n";
  }
}
int main() {
  thread t1(func1);
  thread t2(func2);
  thread t3(func3);

  t1.join();
  t2.join();
  t3.join();
}
```

`join`은 쓰레드들이 실행을 종료하면 리턴하는 함수
`detatch`는 해당 쓰레드를 실행 시킨 후, 잊어버리는 것 이라 생각하시면 됩니다. 대신 쓰레드는 알아서 백그라운드에서 돌아감


### 쓰레드에 인자 전달하기


```cpp
#include <cstdio>
#include <iostream>
#include <thread>
#include <vector>
using std::thread;
using std::vector;

void worker(vector<int>::iterator start, vector<int>::iterator end,
            int* result) {
  int sum = 0;
  for (auto itr = start; itr < end; ++itr) {
    sum += *itr;
  }
  *result = sum;

  // 쓰레드의 id 를 구한다.
  thread::id this_id = std::this_thread::get_id();
  printf("쓰레드 %x 에서 %d 부터 %d 까지 계산한 결과 : %d \n", this_id, *start,
         *(end - 1), sum);
}

int main() {
  vector<int> data(10000);
  for (int i = 0; i < 10000; i++) {
    data[i] = i;
  }

  // 각 쓰레드에서 계산된 부분 합들을 저장하는 벡터
  vector<int> partial_sums(4);

  vector<thread> workers;
  for (int i = 0; i < 4; i++) {
    workers.push_back(thread(worker, data.begin() + i * 2500,
                             data.begin() + (i + 1) * 2500, &partial_sums[i]));
  }

  for (int i = 0; i < 4; i++) {
    workers[i].join();
  }

  int total = 0;
  for (int i = 0; i < 4; i++) {
    total += partial_sums[i];
  }
  std::cout << "전체 합 : " << total << std::endl;
}
```


worker에서 std::cout 대신에 printf를 사용한 이유

한 번 여러분이 컴퓨터라고 생각하고 위 `std::cout` 명령을 실행한다고 생각해보세요. 만약에 `std::cout << "쓰레드 "` 까지 딱 실행했는데 운영체제가 갑자기 다른 쓰레드를 실행시키면 어떨까요? 그렇다면 화면에는 쓰레드 만 딱 나오고 그 뒤로 다른 쓰레드의 메세지가 표시될 것입니다.

따라서 위와 같이 `std::cout` 의 `<<` 를 실행하는 과정 중간 중간에 계속 실행되는 쓰레드들이 바뀌면서 결과적으로 메세지가 뒤섞여서 나타나게 됩니다.

`std::cout` 의 경우 `std::cout << A;` 를 하게 된다면 A 의 내용이 출력되는 동안 중간에 다른 쓰레드가 내용을 출력할 수 없게 보장을 해줍니다 (그 사이에 컨텍스트 스위치가 되더라도 말이지요). 하지만 `std::cout << A << B;` 를 하게 되면 `A` 를 출력한 이후에 `B` 를 출력하기 전에 다른 쓰레드가 내용을 출력할 수 있습니다.

반면에 [printf](https://modoocode.com/35) 는 조금 다릅니다. [printf](https://modoocode.com/35) 는 `"..."` 안에 있는 문자열을 출력할 때, 컨텍스트 스위치가 되더라도 다른 쓰레드들이 그 사이에 메세지를 집어넣지 못하게 막습니다. (자세한 내용은 여기 [참고](https://stackoverflow.com/questions/23586682/how-to-use-printf-in-multiple-threads))

따라서, 방해받지 않고 전체 메세지를 제대로 출력할 수 있게 해줍니다.


### 메모리를 같이 접근한다면?

```cpp
#include <iostream>
#include <thread>
#include <vector>
using std::thread;
using std::vector;

void worker(int& counter) {
  for (int i = 0; i < 10000; i++) {
    counter += 1;
  }
}

int main() {
  int counter = 0;

  vector<thread> workers;
  for (int i = 0; i < 4; i++) {
    // 레퍼런스로 전달하려면 ref 함수로 감싸야 한다 (지난 강좌 bind 함수 참조)
    workers.push_back(thread(worker, std::ref(counter)));
  }

  for (int i = 0; i < 4; i++) {
    workers[i].join();
  }

  std::cout << "Counter 최종 값 : " << counter << std::endl;
}
```


흠 결과가 조금 이상하네요? 분명히 각 쓰레드에서 10000 씩 더했기 때문에 정상적인 상황이였다면 40000 이 출력되어야 했을 것입니다. 그런데, 모든 쓰레드들이 종료되고 최종적으로 `Counter` 에 써진 값은 10000 이 되었습니다 -> 레이스 컨디션