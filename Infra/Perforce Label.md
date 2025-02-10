
체리 픽을 위해서 changing list (정확히는 파일)에 label을 태그할 수 있다. label은 반드시 'super' 계정으로 만든다.

#### label tagging 로직
Development에 submit -> p4 trigger -> 'auto_label.sync.sh' 실행
* script path: /opt/perforce/triggers/auto_label_sync.sh
* p4 login -> 
* chaging_list number로 description 조회
* Regex pattern으로 `[yyyy.mm]` 패턴 찾기
* label_name 생성: REL-yyyy.mm
* label_name으로 조회 및 없는 경우 생성
* Regex를 이용해서 변경 파일들 조회
* p4 tag 실행

```bash
#/opt/perforce/triggers/auto_label_sync.sh

P4PASSWD=""

p4 login <<EOF
$P4PASSWD
EOF

changelist_number=$1


desc=$(p4 -ztag describe -s "$changelist_number" | grep "^... desc" | sed 's/^... desc //')

# [yyyy.mm] 형식을 추출하는 정규표현식 패턴
date_pattern=$(echo "$desc" | grep -o '\[[0-9]\{4\}\.[0-9]\{2\}\]' | sed 's/\[//;s/\]//')
label_name="REL-$date_pattern"

# Label이 있는지 체크
label_exists=$(p4 labels | grep "^$label_name")

# 만약 Label이 없다면, 만들어주기
if [ -z "$label_exists" ]; then
    p4 label -i << EOF
Label:  $label_name
Owner:  super
Description:  auto $label_name description
Options:  unlocked
View:
        //reltest/dev/...
EOF
fi

files=$(p4 -ztag describe -s "$changelist_number" | grep "^... depotFile[0-9]* " | sed 's/^... depotFile[0-9]* //' | sed "s|$|@$changelist_number|" | tr '\n' ' ')
p4 tag -l "$label_name" $files
```

#### 명령어
* p4 label - 모든 레이블 조회
* p4 files {label_name} - label과 파일 연관 관계 조회. 
* p4 tag -d -l {label_name} {depot_path} - label과 파일 연관 관계 끊기
* p4 label -d {label_name} - label 삭제
