
### 도커 설치

```
brew install --cask docker
```

```zsh
docker -v # docker 설치 확인
docker search ubuntu # ubuntu 이미지 검색
```

### 초기설정

```zsh
docker pull ubuntu:22.04 #specific version
docker run -it ubuntu:22.04 /bin/bash
```

### 패키지 설치 및 설정

```zsh
# 패키지 목록 업데이트
apt update

# zsh 및 git, curl 등등 필요한 도구 설치
apt install -y zsh curl git build-essential vim

# Oh My Zsh 설치 (선택사항)
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

### 설치한 zsh 플러그인 

git - manual로 설치했음
```
zsh-autosuggestions
zsh-syntax-highlighting
```

### 컨테이너 아이디 확인 (다른 터미널 세션에서)
```shell
docker ps
docker ps -a #  꺼져있는 컨테이너도 조회됨.
```

### 컨테이너 커밋

```shell
docker commit [컨테이너ID] my-ubuntu-zsh:1.0 # {name}:{version}
```

### 컨테이너 실행

docker는 exit를 입력하면 종료됨. 삭제 되는게 아닌 컨테이너가 종료됨. 
* run: 만들면서 실행
* start: 종료된 컨테이너 실행

```shell
$ docker start -i {name} # 이름으로
$ docker start -i {container_id} #id로
```

### 컨테이너 이름 변경

처음 run할 때 --name 지정 안하면 랜덤으로 지정됨 그래서 바꿔야함

```
docker rename {old} {new}
```

### 컨테이너 삭제

```
docker rm {docker_container_name}
```

