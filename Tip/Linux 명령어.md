

읽기, 쓰기 권한을 다른사용자 (others)에게 부여

```bash
sudo chmod o+x # 쓰기 권한 cd 
sudo chmod o+r # 읽기 권한 ls
```


리눅스 전체 삭제 및 파일 내용 지우기

```vi
$ gg # 첫번째 줄로 이동
$ dG # 현재부터 끝까지 지움
```

```vi
$ dd #한줄삭제
$ 5dd #현재위치부터 5줄 삭제
```


로그 보는 방법

```bash
$ tail -n {몇라인 볼 지 = 수} {log_path}
```