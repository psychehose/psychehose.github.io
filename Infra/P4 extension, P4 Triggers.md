
trigger와 extension을 사용하면 Helix core에서 발생하는 동작과 바인딩해서 유저가 작생한 로직을 실행할 수 있음.
### P4 Trigger

- 특정 Perforce 명령어가 실행될 때 자동으로 실행되는 스크립트
- 주로 간단한 검증이나 알림을 위해 사용
- shell, python, perl 등 사용
- form - out을 이용해서 디폴트 description 변경

### P4 Extension

- Perforce 서버의 기능을 확장하는 플러그인 형태의 프로그램
- lua로 작성되며 Perforce API를 직접 사용할 수 있음
- 복잡한 로직에 적합함
- swarm을 extension으로 사용했음.


둘은 동시 사용 가능


#### p4 trigger 적용법


예시는 form-out 이벤트가 발생할 때 default description를 변경하는  것이다.

* p4 trigger와 바인딩할 파일(shell, python 등등 가능) 을 p4 서버 내에 만든다.
```bash
#!/bin/bash

formfile="$1"

sed -i '/\<enter description\>/c\r [기능출시예정일자][이름][커밋내용]' "$formfile"
```

* p4 서버 내에서 trigger와 바인딩 한다.

```bash
$ p4 triggers
# Triggers 아래에
# $ {Trigger 이름} {Trigger 이벤트 종류} "{file path}"

Triggers:
	form_template form-out change "/opt/perforce/triggers/form_template.sh %formfile%"

```

앞으로는 form-out change가 이벤트가 발생하면 자동으로 해당 스크립트가 실행된다.

#### p4 extension 적용법


Extname을 가진 스켈레톤 폴더를 만든다.
```bash
$ cd $P4ROOT/server.extension.dir
$ p4 extension --sample Extname
```

`manifest.json`, `main.lua` 을 수정한다.

`manifest.json`에 들어 가야할 필수 요소들이 있기 때문에 문서를 참고

[Server Extension JSON manifest fields][https://help.perforce.com/helix-core/server-apps/extensions/current/Content/Extensions/extensionjson.html]

`main.lua` 파일에 우리가 원하는 로직을 작성 해야한다. 어떤 Event Callback을 받아서 처리할건지는 문서를 참고

```json
{
    "manifest_version": 1,
    "api_version": 20191,
    "script_runtime": {
        "language": "lua",
        "version": "5.3"
    },
    "key": "여기에_새로_생성한_UUID_입력",
    "name": "My Extension name",
    "namespace": "MyCompany",
    "version": "1.0",
	"version_name": "",
    "description": "My descrip",
    "compatible_products": [ "p4d" ],
    "default_locale": "en",
	"supported_local" : ["en"],
    "developer": {
        "name": "Your Company",
        "url": "https://your-company.com/"
    },
    "license": "MIT",
    "license_body": "Copyright 2024, Your Company Name"
}

```


[Server extension callbacks][https://help.perforce.com/helix-core/server-apps/extensions/current/Content/Extensions/extensioncallbacks.html#Server_extension_callbacks]

![[compared_ext_trigger.png]]


InstanceConfigEvent()에서 어떤 이벤트 callback을 정의하고, 그에 해당하는 함수를 작성 해야한다. 

```lua
function GlobalConfigFields()
    return {}
end

function GlobalConfigEvents()
    return {}
end

function InstanceConfigFields()
    return {}
end

function InstanceConfigEvents()
    return { ["form-out"] = "change", ["form-in"] = "change" }
end

function FormOut()
    return true
end

function FormIn()
	return true
end
```



```bash
# change-desc-template 폴더 안에 main.lua,manifest.json 존재
sudo -u perforce p4 extension --package change-desc-template

# sign없이 install
sudo -u perforce p4 extension --install change-desc-template.p4-extension -y --allow-unsigned

# Extuser를 super 권한이 있는 user로 변경, 전역설정
sudo -u perforce p4 extension --configure Perforce::Change-Description-Template # Extuser를 super 권한이 있는 user로 변경, 전역설정

# super 권한이 있는 user로 변경, 인스턴스 설정
sudo -u perforce p4 extension \  
  --configure Perforce::Change-Description-Template \  
  --name "ChangeDescInstance" \  
  -y
```



```bash
# 현재 돌고 있는 익스텐션 확인
sudo -u perforce p4 extension --list --type=extensions

# To list all instance configurations:
sudo -u perforce p4 extension --list --type configs #configs (all), global, instance

# extension 모두 지우기
sudo -u perforce p4 extension --delete Perforce::Change-Description-Template -y

# extension name 기반 instance 지우기
sudo -u perforce p4 extension --delete Perforce::Change-Description-Template --name instance name -y

```