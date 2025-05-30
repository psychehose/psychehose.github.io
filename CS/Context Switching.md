컨텍스트 스위칭은 CPU가 하나의 프로세스 또는 스레드에서 다른 프로세스 또는 스레드로 제어를 전환하는 과정이다.이 과정에서 현재 실행 중인 프로세스의 상태(컨텍스트)를 저장하고 다음에 실행할 프로세스의 상태를 불러온다.

#### 컨텍스트 스위칭 발생 원인

1. 시분할(Time Sharing): 여러 프로세스가 CPU 시간을 공유하기 위해 운영체제가 주기적으로 스위칭
2. 인터럽트(Interrupt) 발생: 하드웨어 인터럽트, 소프트웨어 인터럽트 등이 발생했을 때
3. I/O 요청: 프로세스가 입출력 작업을 요청했을 때(디스크 읽기/쓰기 등)
4. 동기화 이벤트: 세마포어 획득 대기, 뮤텍스 락 대기 등


#### 컨텍스트 스위칭 과정
1. 현재 실행 중인 프로세스의 상태를 PCB에 저장
2. 해당 프로세스의 PCB를 프로세스 테이블에 업데이트
3. 다음 실행할 프로세스의 PCB를 프로세스 테이블에서 로드
4. 해당 PCB의 정보를 기반으로 CPU 레지스터, 메모리 맵 등을 복원


#### 오버헤드

1. 컨텍스트 스위칭을 하면 오버헤드가 발생할 수 밖에 없다. 프로세스가 전환되면 기존의 컨텍스트를 PCB에 저장하고 프로세스를 업데이트 하고 전환된 PCB를 로드해야하기 때문이다. (CPU 시간 소비)
2. 프로세스는 자신만의 메모리 공간을 사용하는데 이전 프로세스가 사용했던 CPU 캐시는 새 프로세스에게 유용하지 않다. 즉 캐시 미스가 증가한다.


오버헤드를 줄이기 위해서 컨텍스트 스위칭을 최적화 하는 방법이 여러가지가 있는데 **스레드 사용과 배치 처리**가 있다.

스레드를 사용하면 프로세스를 전환하는 것보다 오버헤드가 적다.

1. 스레드는 프로세스보다 적은 상태 정보를 저장한다.
2.  같은 프로세스 내 스레드들은 메모리 공간을 공유하므로 전체 주소 공간(코 데 힙) 전환 이 필요 없다. (스택 영역은 독립적이라 변경 필요)
3. 스레드간 전환 시 캐시 재사용 가능성 높아짐


배치 처리는 요청을 묶어서 한번에 처리해 컨텍스트 스위칭 횟수 자체를 줄이는 방법이다.

1. 시스템 콜: 여러 개의 시스템 호출을 하나로 묶어 커널 모드로의 전환 횟수 감소
2. I/O 작업 최적화: 여러 I/O 작업을 한 번에 수행하여 블로킹으로 인한 컨텍스트 스위칭 감소
3. 자원 할당: 자원 할당/해제 과정에서 발생하는 컨텍스트 스위칭 최소화



