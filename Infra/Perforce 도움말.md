
## 스트림 개념 이해

스트림 타입과 Depth: 스트림의 깊이(Depth)는 코드가 얼마나 안정적이어야 하는지를 나타냅니다. 상류에서 하류로 갈수록 코드는 더 안정적이고 검증된 상태가 됩니다.

1. Mainline (깊이: 1)
    - 가장 기본이 되는 스트림입니다
    - 프로젝트의 "진실의 원천(source of truth)"입니다
    - 모든 안정적인 코드가 최종적으로 모이는 곳입니다
      
2. Development (깊이: 2)
    - 실제 개발이 이루어지는 스트림입니다
    - Mainline에서 분기되어 새로운 기능을 개발합니다
    - 여러 개발자들의 작업이 통합되는 곳입니다
      
3. Release (깊이: 2)
    - 제품 출시를 위한 스트림입니다
    - Mainline에서 분기되어 특정 버전의 안정화를 담당합니다
    - 버그 수정만 허용되며, 새로운 기능 개발은 제한됩니다
      
4. Task/Feature (깊이: 3)
    - 가장 구체적인 작업이 이루어지는 스트림입니다
    - Development 스트림에서 분기됩니다
    - 개별 기능 개발이나 버그 수정을 위해 사용됩니다


## 명령어
### base revision 체크하는 방법 
```
# Open command Windows here 선택 후
p4 changes -m1 "./...#have"
```

### ignore 적용

```
p4 set P4IGNORE=.p4ignore 
```

### depot 조회
```
p4 depots # 현재 존재하는 depot 확인
```

### stream depot 생성

```
p4 depot -t stream {streamdepotname}
```

### stream 설정

```
p4 stream -t mainline //mystream/main #mainline 설정
p4 stream -t development -P //mystream/main //mystream/dev #development 설정

```

* `-t development`: 스트림의 타입을 'development'로 지정. 이는 이 스트림이 개발 작업을 위한 것.
- `-P //mystream/main`: 부모 스트림을 지정. //mystream/main은 부모 스트림의 전체 경로.
- `//mystream/dev`: 생성하려는 새 스트림의 전체 경로

### stream 확인

```
p4 streams -a //{Depot_name}/...
```

### stream 삭제

```
#development 스트림 강제 삭제
p4 stream -F -d //{Depot_name}/develop

#develop 스트림의 파일 삭제
p4 stream --obliterate -y //{Depot_name}/develop
```

### local depot 삭제

```
p4 obliterate -y //{Depot_name}/...
p4 depot -f -d {Depot_name}
```
### 쓰기 권한으로 변경

```
p4 edit -t +w -c {changing_list_number} ...
```

### 유저 생성

```
p4 user -f {username} # 아이디 생성
p4 passwd {username} # 비밀번호 설정
```

### 모든 유저 조회

```
p4 users
```

### 유저 삭제

```
# 1. 해당 사용자의 클라이언트 워크스페이스 삭제
p4 client -d username-workspace

# 2. 그 다음 사용자 삭제
p4 user -d username

# 3. 삭제가 제대로 되었는지 확인
p4 users | grep username
```



