

Objective - C는 아직도 시장에서 꽤 큰 점유율을 가지고 있음
Swift와 C++의 interoperability 가능 (WWDC23 - Mix Swift and C++) 신기능이라 아직 레퍼가 부족?


Swift 코드베이스에 Objc 쓰기 -> Objective-C Bridging Header
Objc 코드베이스에 Swift 쓰기 -> public과 open 키워드가 있는 객체들 자동으로 헤더로 생성

framework를 개발할 때 공개해야할 API를 잘 설정해야함.



### Importing Objective - C code into Swift internally for a framework

Swift 코드 베이스에서 Objc를 사용할 것. 프레임워크 내부에서 ObjC API를 보고 사용할 수 있지만 이 프레임워크를 사용하는 외부는 Objc 코드를 볼 수 있음


엄브렐라 헤더에 Objc 해더를 임포트하고 Build phrase에서 public으로 바꾸기

그런 다음에 xcframework로 뽑기


```bash

## Framework 아카이브 - iphoneOS
xcodebuild archive -project ObjcIntoSwiftFramework.xcodeproj \
-scheme ObjcIntoSwiftFramework \
-sdk iphoneos \
-destination "generic/platform=iOS" \
-archivePath archives/ObjcIntoSwiftFramework-iOS.xcarchive

## Framework 아카이브 - iphone simulator
xcodebuild archive -project ObjcIntoSwiftFramework.xcodeproj \
-scheme ObjcIntoSwiftFramework \
-sdk iphonesimulator \
-destination "generic/platform=iOS Simulator" \
-archivePath archives/ObjcIntoSwiftFramework-iOS_Simulator.xcarchive


## xcframework 추출
xcodebuild -create-xcframework \
-archive archives/ObjcIntoSwiftFramework-iOS.xcarchive -framework ObjcIntoSwiftFramework.framework \
-archive archives/ObjcIntoSwiftFramework-iOS_Simulator.xcarchive -framework ObjcIntoSwiftFramework.framework \
-output xcframeworks/ObjcIntoSwiftFramework.xcframework
```


xcframework을 확인 해보면 헤더에 모든 헤더가 노출되어 있음

![[allpublic.png]]


이제 해야할 것은 xcframework에서 internal objective - c 헤더를 제거하는 것 

이를 위해서엄블레라 헤더에서 

Internal 주석 이하 모두 삭제xcframework 경로를 파라미터로 받는 스크립트를 실행해서 처리함



```bash
#! /bin/sh -e

#

# removeInternalHeaders.sh

#

  

## 1

XCFRAMEWORK_DIR=$1

INTERNAL_MARK="__INTERNAL__"

  

## 2

function removeInternalHeadersInUmbrellaHeader {

  local framework_name="$(basename $1 .framework)"

  local headers_dir="$1/Headers"

  local umbrella_header_file="$headers_dir/$framework_name.h"

  local internal_mark_found=false

  local internal_headers=()

  ## 2.1

  while read -r line; do

    if $internal_mark_found; then

      if [[ $line == "#import"* ]]; then

        local filename=$(sed 's/.*\"\(.*\)\".*/\1/' <<< $line)

        internal_headers[${#internal_headers[@]}]=$filename

      fi

    elif [[ $line == *$INTERNAL_MARK* ]]; then

        internal_mark_found=true

    fi

  done < $umbrella_header_file

  

  ## 2.2

  echo "${#internal_headers[@]} files will be removed"

  for filename in ${internal_headers[@]}; do

    local file="$headers_dir/$filename"

    if [ -f "$file" ]; then

      rm $file

      echo "Removed file: $file"

    else

      echo "Tried to remove file but it does not exist: $file"

    fi    

  done

  

  ## 2.3

  sed -i "" '/'$INTERNAL_MARK'/,$d' $umbrella_header_file 

}

  

## 3

for directory in ${XCFRAMEWORK_DIR}/**/*.framework; do

  [ -d "$directory" ] || continue

  removeInternalHeadersInUmbrellaHeader $directory

done


```


```bash
# $HOME = 내 Users Path
$ ./removeInternalHeaders.sh $HOME/Labs/ObjcIntoSwiftFramework/xcframeworks/ObjcIntoSwiftFramework.xcframework
```

결과

![[removedinternal.png]]





### Importing Swift code into Objective - C internally for a framework


https://www.fleksy.com/blog/developing-an-ios-framework-in-unison-with-swift-objective-c/