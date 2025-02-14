

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


#### vim 라인 주석 / 해제

1. v 눌러서 visual mode 진입
2. 주석할 라인 선택
3. norm i# (normal mode에서 i -> edit 모드 -> # 삽입) 입력 
---
1. v 눌러서 visual mode 진입
2. 주석 해제할 라인 선택
3. norm 1x (맨 앞에서 한글자 지우기) 입력

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
4. kill -15 (SIGTERM) 시도
5. kill -2 (SIGINT) 시도
6. 마지막 수단으로 kill -9 (SIGKILL) 사용


#### sed

1. sed 's/^... test'//' 
	* 시작부터 ... test" 문자열을 찾음 -> 빈문자열로 치환(//)
	* ^ = 라인의 시작
	  
2. sed `"s|$|@$changelist_number|"`
	- `s|pattern|replacement|`: 여기서는 구분자로 `/` 대신 `|` 사용 (경로에 `/`가 있어서)
	- `$`: 라인의 끝을 의미
	- `@$changelist_number`: 라인 끝에 "@"와 changelist 번호를 추가


3. sed 's/.$//'
	- `.`: 아무 문자 하나를 의미
	- `$`: 문자열의 끝을 의미
	- 따라서 `.$`는 문자열의 마지막 문자 하나를 의미
	- `s/pattern//`: pattern과 매칭되는 부분을 빈 문자열로 치환 (= 삭제)

4. sed 's/..$//'
	* 맨 마지막 문자 2개를 삭제


#### tee

터미널과 출력과 파일에 입력을 동시에 할 수 있는 명령어.

```shell
$ echo test | tee tee-test-file.txt
test
$ cat tee-test-file.txt
test
```


