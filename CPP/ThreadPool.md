```cpp
#include <chrono>
#include <condition_variable>
#include <cstdio>
#include <functional>
#include <mutex>
#include <queue>
#include <thread>
#include <vector>

namespace ThreadPool {
class ThreadPool {
 public:
  ThreadPool(size_t num_threads);
  ~ThreadPool();

  // job 을 추가한다.
  void EnqueueJob(std::function<void()> job);

 private:
  // 총 Worker 쓰레드의 개수.
  size_t num_threads_;
  // Worker 쓰레드를 보관하는 벡터.
  std::vector<std::thread> worker_threads_;
  // 할일들을 보관하는 job 큐.
  std::queue<std::function<void()>> jobs_;
  // 위의 job 큐를 위한 cv 와 m.
  std::condition_variable cv_job_q_;
  std::mutex m_job_q_;

  // 모든 쓰레드 종료
  bool stop_all;

  // Worker 쓰레드
  void WorkerThread();
};

ThreadPool::ThreadPool(size_t num_threads)
    : num_threads_(num_threads), stop_all(false) {
  worker_threads_.reserve(num_threads_);
  for (size_t i = 0; i < num_threads_; ++i) {
    worker_threads_.emplace_back([this]() { this->WorkerThread(); });
  }
}

void ThreadPool::WorkerThread() {
  while (true) {
    std::unique_lock<std::mutex> lock(m_job_q_);
    cv_job_q_.wait(lock, [this]() { return !this->jobs_.empty() || stop_all; });
    if (stop_all && this->jobs_.empty()) {
      return;
    }

    // 맨 앞의 job 을 뺀다.
    std::function<void()> job = std::move(jobs_.front());
    jobs_.pop();
    lock.unlock();

    // 해당 job 을 수행한다 :)
    job();
  }
}

ThreadPool::~ThreadPool() {
  stop_all = true;
  cv_job_q_.notify_all();

  for (auto& t : worker_threads_) {
    t.join();
  }
}

void ThreadPool::EnqueueJob(std::function<void()> job) {
  if (stop_all) {
    throw std::runtime_error("ThreadPool 사용 중지됨");
  }
  {
    std::lock_guard<std::mutex> lock(m_job_q_);
    jobs_.push(std::move(job));
  }
  cv_job_q_.notify_one();
}

}  // namespace ThreadPool

void work(int t, int id) {
  printf("%d start \n", id);
  std::this_thread::sleep_for(std::chrono::seconds(t));
  printf("%d end after %ds\n", id, t);
}

int main() {
  ThreadPool::ThreadPool pool(3);

  for (int i = 0; i < 10; i++) {
    pool.EnqueueJob([i]() { work(i % 3 + 1, i); });
  }
}
```

한계: 우리가 전달한 함수가 어떠한 값을 리턴할 때 입니다. 물론 그 함수에 포인터로 리턴값을 저장할 변수를 전달하면 되기는 합니다. 하지만, 기존의 `future` 처럼 그 값이 설정될 때 까지 기다리는 것은 불가능

따라서 더 나은 구조로는 `EnqueueJob` 함수가 임의의 형태의 함수를 받고, 그 함수의 리턴값을 보관하는 `future` 를 리턴

```cpp
#include <chrono>
#include <condition_variable>
#include <cstdio>
#include <functional>
#include <future>
#include <mutex>
#include <queue>
#include <thread>
#include <vector>

namespace ThreadPool {
class ThreadPool {
 public:
  ThreadPool(size_t num_threads);
  ~ThreadPool();

  // job 을 추가한다.
  template <class F, class... Args>
  std::future<typename std::result_of<F(Args...)>::type> EnqueueJob(
    F f, Args... args);

 private:
  // 총 Worker 쓰레드의 개수.
  size_t num_threads_;
  // Worker 쓰레드를 보관하는 벡터.
  std::vector<std::thread> worker_threads_;
  // 할일들을 보관하는 job 큐.
  std::queue<std::function<void()>> jobs_;
  // 위의 job 큐를 위한 cv 와 m.
  std::condition_variable cv_job_q_;
  std::mutex m_job_q_;

  // 모든 쓰레드 종료
  bool stop_all;

  // Worker 쓰레드
  void WorkerThread();
};

ThreadPool::ThreadPool(size_t num_threads)
    : num_threads_(num_threads), stop_all(false) {
  worker_threads_.reserve(num_threads_);
  for (size_t i = 0; i < num_threads_; ++i) {
    worker_threads_.emplace_back([this]() { this->WorkerThread(); });
  }
}

void ThreadPool::WorkerThread() {
  while (true) {
    std::unique_lock<std::mutex> lock(m_job_q_);
    cv_job_q_.wait(lock, [this]() { return !this->jobs_.empty() || stop_all; });
    if (stop_all && this->jobs_.empty()) {
      return;
    }

    // 맨 앞의 job 을 뺀다.
    std::function<void()> job = std::move(jobs_.front());
    jobs_.pop();
    lock.unlock();

    // 해당 job 을 수행한다 :)
    job();
  }
}

ThreadPool::~ThreadPool() {
  stop_all = true;
  cv_job_q_.notify_all();

  for (auto& t : worker_threads_) {
    t.join();
  }
}

template <class F, class... Args>
std::future<typename std::result_of<F(Args...)>::type> ThreadPool::EnqueueJob(
  F f, Args... args) {
  if (stop_all) {
    throw std::runtime_error("ThreadPool 사용 중지됨");
  }

  using return_type = typename std::result_of<F(Args...)>::type;
  auto job =
    std::make_shared<std::packaged_task<return_type()>>(std::bind(f, args...));
  std::future<return_type> job_result_future = job->get_future();
  {
    std::lock_guard<std::mutex> lock(m_job_q_);
    jobs_.push([job]() { (*job)(); });
  }
  cv_job_q_.notify_one();

  return job_result_future;
}

}  // namespace ThreadPool

int work(int t, int id) {
  printf("%d start \n", id);
  std::this_thread::sleep_for(std::chrono::seconds(t));
  printf("%d end after %ds\n", id, t);
  return t + id;
}

int main() {
  ThreadPool::ThreadPool pool(3);

  std::vector<std::future<int>> futures;
  for (int i = 0; i < 10; i++) {
    futures.emplace_back(pool.EnqueueJob(work, i % 3 + 1, i));
  }
  for (auto& f : futures) {
    printf("result : %d \n", f.get());
  }
}
```

`EnqueueJob` 함수의 경우 인자들의 복사본을 받는다는 것 하지만 이는 불필요한 복사를 야기하므로 [완벽한 전달](https://modoocode.com/228) 패턴을 사용

`EnqueueJob` 파라미터에서 우측값 레퍼런스로 변경, bind 함수에 forward로 인자 전달

```cpp
#include <condition_variable>
#include <cstdio>
#include <functional>
#include <future>
#include <mutex>
#include <queue>
#include <thread>
#include <vector>

class ThreadPool {

public:

ThreadPool(size_t num_threads);

~ThreadPool();

  

template <class F, class... Args>

std::future<typename std::invoke_result<F, Args...>::type> EnqueueJob(

F&& f, Args&&... args) {

if (stop_all) {

throw std::runtime_error("ThreadPool is stopped");

}

  

using return_type = typename std::invoke_result<F, Args...>::type;

auto job = std::make_shared<std::packaged_task<return_type()>>(

std::bind(std::forward<F>(f), std::forward<Args>(args)...));

  

std::future<return_type> job_result_future = job->get_future();

jobs_.push([job]() { (*job)(); });

cv_job_q_.notify_one();

  

return job_result_future;

}

  

private:

size_t num_threads_;

  

std::vector<std::thread> worker_threads_;

  

std::queue<std::function<void()>> jobs_;

  

std::condition_variable cv_job_q_;

std::mutex m_job_q_;

  

bool stop_all;

  

void WorkerThread();

};
```




ThreadPool은 Thread에 일을 시키는 거야. 

그러니까 Thread에 일을 시키기 위해서는 어떤 일을 할당할 지 함수를 매개변수로 넘겨 줘야해. c에서는 이걸 함수 포인터로 처리해 하지만 c++에서는 다른 방법이 있어. 

(1.이 부분을 좀 더 풀어서 설명하고 싶어. 지금과 같은 상황에서는 어떤 용어를 써야하지? )
그걸 템플릿 메서드를 이용해서 F f 로 받을거야.
우리가 넘겨줄 함수 f에는 매개변수가 들어갈수도 있어. 이 매개변수를 처리하기 위해 파라미터 팩을 이용할거야. class ... Args를 이용해서 함수 f의 매개변수의 값을 받는거지.

return type은 비동기 처리를 위해서 future로 설정했어. 
(2.이 부분이 비동기 처리를 위해서 future를 사용했는 지 확실하지 않아.)
그리고 invoke_result 함수를 통해서 함수 f의 리턴 값을 추론 할 수 있어.

invoke_result를 이용해서 함수 f의 리턴 값 추론 하고 비동기 지원을 위해 future에 담아서 return 하는거지. 

(3.만약 future가 아니라면 어떻게 될까?)

이제 함수 내용을 살펴보자

stop_all 플래그가 이미 true면 runtime_error를 던져주자.

리턴 타입이 너무 길어 그래서 using을 이용해서 return_type을 정의할게.

이제 우리는 함수 f, f에 들어갈 파라미터 args, 그리고 함수 f의 리턴값이 준비되었어.

이제 job을 만들거야. 이걸 shared_ptr로 만들자. 
(4. 왜 인지 보충 설명이 필요해) 

타입은 std::packaged_task<return_type()>이거야. 

(5.return_type을 packaged_task로 감싼 이유를 알려줘. 그냥 return_type을 사용하면 안되는건지..)

그리고 bind 함수를 통해서 f와, args를 결합할거야.

job_result_future = job->get_future();를 해서 패키지된 task에서 future를 추출했어.

(6. jobs_.push([job]() { (*job)(); });를 하는 이유가 뭘까? 이 부분은 문법적으로 이해가 잘 가지 않아.)

그런 다음에 job_result_future을 리턴할거야.

main에서 사용법을 보자. EnqueueJob을 하고 값을 futres에 넣을거야.
futures는 job_result_future를 원소로 가지고 있고 이걸 .get을 하면 그 함수 내용이 실행될거야. get을 하면 함수가 실행되니까.


1. C와 C++의 함수 처리 방식 차이에 대해 설명하자면: C에서는 함수 포인터를 사용해서 콜백 함수를 전달하는데, 이는 문법이 복잡하고 타입 안전성이 떨어집니다. 반면 C++에서는 `std::function`이라는 개념을 사용합니다. 이것을 "함수 객체" 또는 "콜러블(callable) 객체"라고 부릅니다.

`std::function`은 일반 함수, 멤버 함수, 람다 함수 등 모든 종류의 호출 가능한 객체를 저장할 수 있어서 더 유연하고 안전합니다.

2. `future`의 비동기 처리 역할: 네, 정확합니다. `std::future`는 비동기 작업의 결과를 나중에 가져올 수 있게 해주는 객체입니다. 쓰레드 풀에서 작업이 완료되면, 그 결과를 future를 통해 메인 쓰레드에서 받아올 수 있습니다.
   
3. future를 사용하지 않았다면: 만약 future를 사용하지 않았다면:

- 작업의 완료 여부를 알 수 없습니다
- 작업의 결과값을 받아올 방법이 없습니다
- 작업이 실패했을 때 예외 처리를 할 수 없습니다
- 비동기 작업의 동기화가 매우 어려워집니다

4. shared_ptr을 사용하는 이유: `shared_ptr`을 사용하는 이유는 메모리 관리 때문입니다. job은 여러 쓰레드에서 공유될 수 있고, 마지막 쓰레드가 작업을 완료했을 때 자동으로 메모리가 해제되어야 합니다. `shared_ptr`은 이런 참조 카운팅을 자동으로 처리해줍니다.
   
5. packaged_task를 사용하는 이유: `std::packaged_task`는 함수와 그 결과를 연결해주는 래퍼(wrapper)입니다. 이것을 사용하는 이유는:

- 함수의 실행을 지연시킬 수 있습니다
- 함수의 결과를 future를 통해 받을 수 있습니다
- 예외 처리를 안전하게 할 수 있습니다

그냥 return_type만 사용하면 이런 기능들을 구현할 수 없습니다.

6. jobs_.push(job { (*job)(); }); 설명: 이 부분은 람다 함수를 만들어 큐에 push하는 것입니다.

- `[job]`은 람다 함수가 캡처할 변수입니다 (job을 람다 안에서 사용하기 위함)
- `()`는 람다 함수의 매개변수입니다 (여기서는 없음)
- `{ (*job)(); }`는 람다 함수의 본문입니다

이렇게 하는 이유는 job(packaged_task)을 나중에 실행하기 위해서입니다. 실제 실행은 WorkerThread에서 이루어집니다.



왜 Mutex가 필요한지
```cpp
// mutex가 없다면 발생할 수 있는 문제 예시
std::queue<std::function<void()>> jobs_;  // 공유 자원

// 쓰레드 1이 작업을 추가하려고 함
jobs_.push(new_job);  // 이 순간에

// 쓰레드 2가 작업을 가져가려고 함
auto job = jobs_.front();  // 동시 접근! 데이터 손상 발생!
```


왜 꼭 packaged_task를 사용해야하는 지 - 함수 실행 지연 & 예외처리

packaged_task는 "작업의 패키징"이 주 목적이고, future는 "결과값을 받는 것"이 주 목적

```cpp
// 1. std::future만 사용할 경우
std::future<int> future = std::async(작업함수);
// 이 시점에서 작업이 이미 시작됨!

// 2. std::packaged_task 사용
std::packaged_task<int()> task(작업함수);
std::future<int> future = task.get_future();
// 이 시점에서는 작업이 시작되지 않음
// 나중에...
task();  // 이 시점에 작업 시작
```


[ThreadPool에서 필요한 이유]
작업 큐잉(Queueing)

```cpp
class ThreadPool {
    std::queue<std::function<void()>> jobs_;  // 작업 대기열
    
    // packaged_task를 사용하면:
    void addJob(std::packaged_task<int()> task) {
        jobs_.push([task = std::move(task)]() {
            task();  // 나중에 실행 가능
        });
    }
};
```


```
// ThreadPool이 하는 일:
1. 작업 받기
auto future = pool.EnqueueJob(work, args...);

2. 내부적으로는:
- packaged_task로 작업을 감싸서
- 큐에 저장했다가
- worker 쓰레드가 나중에 실행

// future만 사용하면:
- 작업이 즉시 시작되어 버림
- 큐잉이 불가능
- 쓰레드 풀의 의미가 없어짐
```


WorkerThread()에서 job(); 할 때 실제 실행

```cpp
// 1. main에서:
auto future = pool.EnqueueJob(work, 1, 2);

// 2. EnqueueJob 내부:
jobs_.push([task]() { (*task)(); });

// 3. WorkerThread에서:
job();  // 실제로 work(1, 2)가 실행되는 시점!
```

