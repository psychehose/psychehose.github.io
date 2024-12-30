
한 쓰레드에서만 위 코드를 실행시키기
race condition을 방지하기 위함

```cpp
#include <iostream>
#include <mutex>  // mutex 를 사용하기 위해 필요
#include <thread>
#include <vector>

void worker(int& result, std::mutex& m) {
  for (int i = 0; i < 10000; i++) {
    m.lock();
    result += 1;
    m.unlock();
  }
}

int main() {
  int counter = 0;
  std::mutex m;  // 우리의 mutex 객체

  std::vector<std::thread> workers;
  for (int i = 0; i < 4; i++) {
    workers.push_back(std::thread(worker, std::ref(counter), std::ref(m)));
  }

  for (int i = 0; i < 4; i++) {
    workers[i].join();
  }

  std::cout << "Counter 최종 값 : " << counter << std::endl;
}
```


```cpp
m.lock();
result += 1;
m.unlock();
```

`m.lock()` 은 뮤텍스 `m` 을 내가 쓰게 달라! 라고 이야기 하는 것입니다. 이 때 중요한 사실은, 한 번에 한 쓰레드에서만 `m` 의 사용 권한을 갖는다는 것입니다. 그렇다면, 다른 쓰레드에서 `m.lock()` 을 하였다면 어떻게될까요? 이는 `m` 을 소유한 쓰레드가 `m.unlock()` 을 통해 `m` 을 반환할 때 까지 무한정 기다리게 됩니다.

m.unlock을 만약 하지 않는다면 다른 쓰레드에서 무한정 기다림 -> 데드락


위와 같은 문제를 해결하기 위해서는 취득한 뮤텍스는 사용이 끝나면 반드시 반환을 해야 합니다. 하지만 코드 길이가 길어지게 된다면 반환하는 것을 까먹을 수 있기 마련입니다.

곰곰히 생각해보면 이전에 비슷한 문제를 해결한 기억이 있습니다. `unique_ptr` 를 왜 도입을 하였는지 생각을 해보자면, 메모리를 할당 하였으면 사용 후에 반드시 해제를 해야 하므로, 아예 이 과정을 `unique_ptr` 의 소멸자에서 처리하도록 했었습니다.

뮤텍스도 마찬가지로 사용 후 해제 패턴을 따르기 때문에 동일하게 소멸자에서 처리


```cpp
#include <iostream>
#include <mutex>  // mutex 를 사용하기 위해 필요
#include <thread>
#include <vector>

void worker(int& result, std::mutex& m) {
  for (int i = 0; i < 10000; i++) {
    // lock 생성 시에 m.lock() 을 실행한다고 보면 된다.
    std::lock_guard<std::mutex> lock(m);
    result += 1;

    // scope 를 빠져 나가면 lock 이 소멸되면서
    // m 을 알아서 unlock 한다.
  }
}

int main() {
  int counter = 0;
  std::mutex m;  // 우리의 mutex 객체

  std::vector<std::thread> workers;
  for (int i = 0; i < 4; i++) {
    workers.push_back(std::thread(worker, std::ref(counter), std::ref(m)));
  }

  for (int i = 0; i < 4; i++) {
    workers[i].join();
  }

  std::cout << "Counter 최종 값 : " << counter << std::endl;
}
```


### 데드락 (deadlock) & Starvation (기아)


데드락을 방지하기 위해서 한 쓰레드에 우선권을 주면, 다른 쓰레드만 일하고 남은 쓰레드는 일하지 않는 경우가 생김 -> Starvation이 생길 가능성이 있음.

```cpp
#include <iostream>
#include <mutex>  // mutex 를 사용하기 위해 필요
#include <thread>

void worker1(std::mutex& m1, std::mutex& m2) {
  for (int i = 0; i < 10; i++) {
    m1.lock();
    m2.lock();
    std::cout << "Worker1 Hi! " << i << std::endl;

    m2.unlock();
    m1.unlock();
  }
}

void worker2(std::mutex& m1, std::mutex& m2) {
  for (int i = 0; i < 10; i++) {
    while (true) {
      m2.lock();

      // m1 이 이미 lock 되어 있다면 "야 차 빼" 를 수행하게 된다.
      if (!m1.try_lock()) {
        m2.unlock();
        continue;
      }

      std::cout << "Worker2 Hi! " << i << std::endl;
      m1.unlock();
      m2.unlock();
      break;
    }
  }
}

int main() {
  std::mutex m1, m2;  // 우리의 mutex 객체

  std::thread t1(worker1, std::ref(m1), std::ref(m2));
  std::thread t2(worker2, std::ref(m1), std::ref(m2));

  t1.join();
  t2.join();

  std::cout << "끝!" << std::endl;
}
```


worker 1에 우선권을 줬음.

```cpp
#include <iostream>
#include <mutex>  // mutex 를 사용하기 위해 필요
#include <thread>

void worker1(std::mutex& m1, std::mutex& m2) {
  for (int i = 0; i < 10; i++) {
    m1.lock();
    m2.lock();
    std::cout << "Worker1 Hi! " << i << std::endl;

    m2.unlock();
    m1.unlock();
  }
}

void worker2(std::mutex& m1, std::mutex& m2) {
  for (int i = 0; i < 10; i++) {
    while (true) {
      m2.lock();

      // m1 이 이미 lock 되어 있다면 "야 차 빼" 를 수행하게 된다.
      if (!m1.try_lock()) {
        m2.unlock();
        continue;
      }

      std::cout << "Worker2 Hi! " << i << std::endl;
      m1.unlock();
      m2.unlock();
      break;
    }
  }
}

int main() {
  std::mutex m1, m2;  // 우리의 mutex 객체

  std::thread t1(worker1, std::ref(m1), std::ref(m2));
  std::thread t2(worker2, std::ref(m1), std::ref(m2));

  t1.join();
  t2.join();

  std::cout << "끝!" << std::endl;
}
```

데드락을 해결하는건 매우매우 복잡하고 완벽하지 않음. 그래서, 데드락 상황이 발생하지 않게 잘 설계하는 것이 중요


데드락 상황을 피하기 위해 다음과 같은 가이드라인을 제시

#### 중첩된 Lock 을 사용하는 것을 피해라

모든 쓰레드들이 최대 1 개의 [Lock](https://modoocode.com/lock) 만을 소유한다면 (일반적인 경우에) 데드락 상황이 발생하는 것을 피할 수 있습니다. 또한 대부분의 디자인에서는 1 개의 [Lock](https://modoocode.com/lock) 으로도 충분합니다. 만일 여러개의 [Lock](https://modoocode.com/lock) 을 필요로 한다면 정말 필요로 하는지 를 되물어보는 것이 좋습니다.

#### Lock 을 소유하고 있을 때 유저 코드를 호출하는 것을 피해라

사실 이 가이드라인 역시 위에서 말한 내용과 자연스럽게 따라오는 것이긴 한데, 유저 코드에서 [Lock](https://modoocode.com/lock) 을 소유할 수 도 있기에 중첩된 [Lock](https://modoocode.com/lock) 을 얻는 것을 피하려면 [Lock](https://modoocode.com/lock) 소유시 유저 코드를 호출하는 것을 지양해야 합니다.

#### Lock 들을 언제나 정해진 순서로 획득해라

만일 여러개의 [Lock](https://modoocode.com/lock) 들을 획득해야 할 상황이 온다면, 반드시 이 Lock 들을 정해진 순서로 획득해야 합니다. 우리가 앞선 예제에서 데드락이 발생했던 이유 역시, `worker1` 에서는 `m1, m2` 순으로 [lock](https://modoocode.com/lock) 을 하였지만 `worker2` 에서는 `m2, m1` 순으로 [lock](https://modoocode.com/lock) 을 하였기 때문이지요. 만일 `worker2` 에서 역시 `m1, m2` 순으로 [lock](https://modoocode.com/lock) 을 하였다면 데드락은 발생하지 않았을 것입니다.



### 생산자(Producer) 와 소비자(Consumer) 패턴

```cpp

#include <chrono>  // std::chrono::miliseconds
#include <iostream>
#include <mutex>
#include <queue>
#include <string>
#include <thread>
#include <vector>

void producer(std::queue<std::string>* downloaded_pages, std::mutex* m,
              int index) {
  for (int i = 0; i < 5; i++) {
    // 웹사이트를 다운로드 하는데 걸리는 시간이라 생각하면 된다.
    // 각 쓰레드 별로 다운로드 하는데 걸리는 시간이 다르다.
    std::this_thread::sleep_for(std::chrono::milliseconds(100 * index));
    std::string content = "웹사이트 : " + std::to_string(i) + " from thread(" +
                          std::to_string(index) + ")\n";

    // data 는 쓰레드 사이에서 공유되므로 critical section 에 넣어야 한다.
    m->lock();
    downloaded_pages->push(content);
    m->unlock();
  }
}

void consumer(std::queue<std::string>* downloaded_pages, std::mutex* m,
              int* num_processed) {
  // 전체 처리하는 페이지 개수가 5 * 5 = 25 개.
  while (*num_processed < 25) {
    m->lock();
    // 만일 현재 다운로드한 페이지가 없다면 다시 대기.
    if (downloaded_pages->empty()) {
      m->unlock();  // (Quiz) 여기서 unlock 을 안한다면 어떻게 될까요?

      // 10 밀리초 뒤에 다시 확인한다.
      std::this_thread::sleep_for(std::chrono::milliseconds(10));
      continue;
    }

    // 맨 앞의 페이지를 읽고 대기 목록에서 제거한다.
    std::string content = downloaded_pages->front();
    downloaded_pages->pop();

    (*num_processed)++;
    m->unlock();

    // content 를 처리한다.
    std::cout << content;
    std::this_thread::sleep_for(std::chrono::milliseconds(80));
  }
}

int main() {
  // 현재 다운로드한 페이지들 리스트로, 아직 처리되지 않은 것들이다.
  std::queue<std::string> downloaded_pages;
  std::mutex m;

  std::vector<std::thread> producers;
  for (int i = 0; i < 5; i++) {
    producers.push_back(std::thread(producer, &downloaded_pages, &m, i + 1));
  }

  int num_processed = 0;
  std::vector<std::thread> consumers;
  for (int i = 0; i < 3; i++) {
    consumers.push_back(
        std::thread(consumer, &downloaded_pages, &m, &num_processed));
  }

  for (int i = 0; i < 5; i++) {
    producers[i].join();
  }
  for (int i = 0; i < 3; i++) {
    consumers[i].join();
  }
}
```


먼저 `producer` 쓰레드에서는 웹사이트에서 페이지를 계속 다운로드 하는 역할을 하게 됩니다. 이 때, 다운로드한 페이지들을 `downloaded_pages` 라는 큐에 저장
![[cpp_13.png]]

그리고 다운 받은 웹사이트 내용이 `content` 라고 생각해봅시다.

그렇다면, 이제 다운 받은 페이지를 작업 큐에 집어 넣어야 합니다. 이 때 주의할 점으로, `producer` 쓰레드가 1 개가 아니라 5 개나 있다는 점입니다. 따라서 `downloaded_pages` 에 접근하는 쓰레드들 사이에 race condition 이 발생할 수 있습니다.

이를 방지 하기 위해서 뮤텍스 `m` 으로 해당 코드를 감싸서 문제가 발생하지 않게 해줍니다.

먼저 `consumer` 쓰레드의 입장에서는 언제 일이 올지 알 수 없습니다. 따라서 `downloaded_pages` 가 비어있지 않을 때 까지 계속 `while` 루프를 돌아야겠지요. 한 가지 문제는 컴퓨터 CPU 의 속도에 비해 웹사이트 정보가 큐에 추가되는 속도는 매우 느리다는 점입니다.

우리의 `producer` 의 경우 대충 100ms 마다 웹사이트 정보를 큐에 추가하게 되는데, 이 시간 동안 `downloaded_pages->empty()` 이 문장을 수십 만 번 호출할 수 있을 것입니다. 이는 상당한 CPU 자원의 낭비가 아닐 수 없지요. 그래서 empty인 경우 10 밀리 세컨드 후에 다시 실행 (continue;) 

정상 작동함..근데 consumer가 10밀리 세컨드마다 downloaded_pages에 할일이 있는 지 확인하고 있으면 하고 없으면 기다리는 형태라서 비효율임 매 번 언제 올지 모르는 데이터를 확인하기 위해 지속적으로 `mutex` 를 [lock](https://modoocode.com/lock) 하고, 큐를 확인해야 하기 때문

어떻게 개선할까?

consumer는 그냥 재워놓고 producer에서 할 일이 온다면 그때 consumer를 깨우자.


### condition_variable

위와 같은 상황에서 쓰레드들을 10 밀리초 마다 재웠다 깨웠다 할 수 밖에 없었던 이유는 어떠 어떠한 조건을 만족할 때 까지 자라! 라는 명령을 내릴 수 없었기 때문입니다.

위 경우 `downloaded_pages` 가 `empty()` 가 참이 아닐 때 까지 자라 라는 명령을 내리고 싶었겠지요.

```cpp
#include <chrono>              // std::chrono::miliseconds
#include <condition_variable>  // std::condition_variable
#include <iostream>
#include <mutex>
#include <queue>
#include <string>
#include <thread>
#include <vector>

void producer(std::queue<std::string>* downloaded_pages, std::mutex* m,
              int index, std::condition_variable* cv) {
  for (int i = 0; i < 5; i++) {
    // 웹사이트를 다운로드 하는데 걸리는 시간이라 생각하면 된다.
    // 각 쓰레드 별로 다운로드 하는데 걸리는 시간이 다르다.
    std::this_thread::sleep_for(std::chrono::milliseconds(100 * index));
    std::string content = "웹사이트 : " + std::to_string(i) + " from thread(" +
                          std::to_string(index) + ")\n";

    // data 는 쓰레드 사이에서 공유되므로 critical section 에 넣어야 한다.
    m->lock();
    downloaded_pages->push(content);
    m->unlock();

    // consumer 에게 content 가 준비되었음을 알린다.
    cv->notify_one();
  }
}

void consumer(std::queue<std::string>* downloaded_pages, std::mutex* m,
              int* num_processed, std::condition_variable* cv) {
  while (*num_processed < 25) {
    std::unique_lock<std::mutex> lk(*m);

    cv->wait(
        lk, [&] { return !downloaded_pages->empty() || *num_processed == 25; });

    if (*num_processed == 25) {
      lk.unlock();
      return;
    }

    // 맨 앞의 페이지를 읽고 대기 목록에서 제거한다.
    std::string content = downloaded_pages->front();
    downloaded_pages->pop();

    (*num_processed)++;
    lk.unlock();

    // content 를 처리한다.
    std::cout << content;
    std::this_thread::sleep_for(std::chrono::milliseconds(80));
  }
}

int main() {
  // 현재 다운로드한 페이지들 리스트로, 아직 처리되지 않은 것들이다.
  std::queue<std::string> downloaded_pages;
  std::mutex m;
  std::condition_variable cv;

  std::vector<std::thread> producers;
  for (int i = 0; i < 5; i++) {
    producers.push_back(
        std::thread(producer, &downloaded_pages, &m, i + 1, &cv));
  }

  int num_processed = 0;
  std::vector<std::thread> consumers;
  for (int i = 0; i < 3; i++) {
    consumers.push_back(
        std::thread(consumer, &downloaded_pages, &m, &num_processed, &cv));
  }

  for (int i = 0; i < 5; i++) {
    producers[i].join();
  }

  // 나머지 자고 있는 쓰레드들을 모두 깨운다.
  cv.notify_all();

  for (int i = 0; i < 3; i++) {
    consumers[i].join();
  }
}
```



```cpp
std::unique_lock<std::mutex> lk(*m);

cv->wait(lk,
         [&] { return !downloaded_pages->empty() || *num_processed == 25; });
```

`condition_variable` 의 [wait](https://modoocode.com/wait-fwait) 함수에 어떤 조건이 참이 될 때 까지 기다릴지 해당 조건을 인자로 


여러 쓰레드에서 같은 객체의 값을 수정한다면 Race Condition 이 발생합니다. 이를 해결하기 위해서는 여러가지 방법이 있지만, 한 가지 방법으로 뮤텍스를 사용하는 방법이 있습니다. 뮤텍스는 한 번에 한 쓰레드에서만 획득할 수 있습니다. 획득한 뮤텍스는 반드시 반환해야 합니다. `lock_guard` 나 `unique_lock` 등을 이용하면 뮤텍스의 획득-반환을 손쉽게 처리할 수 있습니다. 뮤텍스를 사용할 때 데드락이 발생하지 않도록 주의해야 합니다. 데드락을 디버깅하는 것은 매우 어렵습니다. `condition_variable` 을 사용하면 생산자-소비자 패턴을 쉽게 구현할 수 있습니다.