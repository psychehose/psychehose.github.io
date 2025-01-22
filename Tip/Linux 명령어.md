

#### 읽기, 쓰기 권한을 다른사용자 (others)에게 부여

```bash
sudo chmod o+x # 쓰기 권한 cd 
sudo chmod o+r # 읽기 권한 ls
```


#### 리눅스 전체 삭제 및 파일 내용 지우기

```vi
$ gg # 첫번째 줄로 이동
$ dG # 현재부터 끝까지 지움
```

```vi
$ dd #한줄삭제
$ 5dd #현재위치부터 5줄 삭제
```


#### 로그 보는 방법

```bash
$ tail -n {몇라인 볼 지 = 수} {log_path}
```

#### linux 프로세스 확인

```bash
$ ps -ef f | grep p4d
$ ps aux | grep p4d
```

- `ps -ef`: 전체 형식(FULL format)으로 표시, 프로세스 간의 부모-자식 관계를 트리 구조(`f` 옵션)로 보여줌
- `ps aux`: BSD 스타일 출력, CPU/메모리 사용량 등 더 자세한 리소스 정보 표시


#### linux kill 명령어

* kill -1 (SIGHUP): 프로세스 재시작
* kill -2 (SIGINT): Ctrl+C와 동일, 정상 종료 요청
* kill -9 (SIGKILL): 강제 종료, 즉시 종료
* kill -15 (SIGTERM): 기본값, 정상 종료 요청
* kill -18 (SIGCONT): 중지된 프로세스 재개
* kill -19 (SIGSTOP): 프로세스 일시 중지
* kill -20 (SIGTSTP): Ctrl+Z와 동일, 일시 중지

권장 사용 순서:
1. kill -15 (SIGTERM) 시도
2. kill -2 (SIGINT) 시도
3. 마지막 수단으로 kill -9 (SIGKILL) 사용