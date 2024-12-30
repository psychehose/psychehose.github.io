
### perforce helix core

1. 저장소 설정

```bash
sudo tee /etc/yum.repos.d/perforce.repo << 'EOF'
[perforce]
name=Perforce
baseurl=https://package.perforce.com/yum/rhel/9/x86_64
enabled=1
gpgcheck=1
gpgkey=https://package.perforce.com/perforce.pubkey
EOF

```

2. 공개키 등록

```bash
sudo rpm --import https://package.perforce.com/perforce.pubkey
```

3. 저장소 확인

```bash
# 저장소가 제대로 등록되었는지 확인
dnf repolist | grep perforce
# 패키지 검색이 되는지 확인
dnf search helix
```

```bash
# 패키지목록 초기화 하고 다시 받기
sudo dnf clean all
sudo dnf makecache
sudo dnf update
```


4. Helix Core 서버 설치 (p4d) 

```bash
sudo dnf install helix-p4d
```

5. p4d 초기 설정

```bash
sudo /opt/perforce/sbin/configure-helix-p4d.sh
export P4CHARSET=utf8 # 텍스트 인코딩
```

master가 config할 때 정한 이름이였던 거 같음

* P4 설정 파일 경로: `/etc/perforce/p4dctl.conf.d/master.conf`

* P4 루트 디렉토리: `/opt/perforce/servers/master/root`

* 서버중지 - `sudo systemctl stop p4d` 또는 `sudo -u perforce p4dctl stop master`

* 서버 재시작 - `sudo systemctl restart p4d` 또는 `sudo -u perforce p4dctl restart master` 

* 부팅 시 자동 시작 설정: `sudo systemctl enable p4d`

6. aws인 경우 인바운드 추가해서 1666 포트 추가로 열기


### Swarm

1. Swram 필수 패키지 설치
```bash
sudo dnf install httpd php php-xml php-mbstring php-json php-gd php-curl
```

2. Apache 웹 서버 활성화
```bash
sudo systemctl enable httpd
sudo systemctl start httpd
```

3. Swarm 설치
```bash
sudo dnf install helix-swarm
```

4. Swarm 초기설정
```bash
sudo /opt/perforce/swarm/bin/configure-swarm.sh
```

