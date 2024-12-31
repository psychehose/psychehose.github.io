![](https://blog.kakaocdn.net/dn/5mg6k/btr3ft0Suio/SnPjEjZKMZu9EObJNZkfjK/img.png)

### What is PhotoKit?

 PhotoKit은 애플에서 자체적으로 제공하는 프레임워크로 사진 앱에서 관리하는 Asset(photo, Video)에 대해서 접근을 가능하게 해 줍니다. PhoKit을 이용하면 사진 앱에 있는 앨범에 접근하여 Asset들을 가져올 수 있습니다.

### PHObject

PhotoKit의 객체는 PHObject 객체를 기본적으로 상속받습니다. PHObject는 Asset, Collection의 추상적인 superclass입니다.

따라서 이 객체를 다룰 때, 직접적으로 생성하거나 사용해서는 안됩니다. 대신에, 이 객체의 subclass인 PHAssectCollection, PHCollectionList, PHObjectPlaceHolder를 이용해서 개발을 해야 합니다.

### PHAsset

사진 앱에 있는 이미지, 비디오, 라이브 포토의 "A representation"입니다. 그러니까 사진, 비디오, 라이브 포토 하나하나가 PHAsset 인 셈입니다. PHAsset은 다음과 같은 특징을 가지고 있습니다.

- Asset은 class method인 fectchAssets()을 통해 개발을 시작할 수 있다.
- Asset은 메타데이터만을 가지고 있다.
- Asset은 immutable이다. (불변성을 가진다.)

### PHCollection

Photos Asset Collections과 Collection Lists의 superclass입니다.

이것 역시 PHObject처럼 직접적으로 create 하거나 인스턴스로 작업을 할 수 없습니다. 대신에 이것의 subclass인 PHAssetCollection 또는 PHCollectionList로 작업을 할 수 있습니다.

### PHAssetCollection

PHCollection을 상속받습니다. PHAssetCollection은 쉽게 말하면 앨범입니다. (Photos asset grouping)

Photo 프레임워크 안에 있는 Collection Object들은 member object를 직접 참조할 수 없고, 어떠한 object들도 collection object들을 직접 참조할 수 없습니다. Asset Collection의 member를 가지고 오려면, PHAsset class method를 이용해서 fetch 해야 합니다. 이것 또한 Assets과 Collection List와 마찬가지로 immutable 하기 때문에 create, rename... 기타 등등 작업을 하기 위해서 PHAssetCollectionChangeRequest Object를 사용해서 Photo Library를 업데이트해야 합니다.

### PHCollectionList

여러 개의 Photo Asset Collection을 리스트화한 것입니다. ( 예를 들면 기억에 나는 순간들, 연별 폴더 등 같은 것들입니다.)

위의 PHAssetCollection처럼 멤버들을 직접 참조할 수 없고 직접 참조될 수 없습니다. member를 가지고 오려면 PHCollection class method를 이용해서 fetch 해야 합니다. 또한 역시 immutable 하기 때문에, PHCollectionListChangeRequest를 이용해서 업데이트해야 합니다.

### PHFetchResult

이 메서드를 사용하면 PHAsset, PHCollection에서 An ordered list of asset and collection을 가져올 수 있습니다.

### PHFetchOptions

Fetch를 어떻게 할 것인가에 대한 option class이다.

> 요약 및 정리  
>   
> PHObject, PHCollection  
> PHAsset(사진, 비디오, 라이브포토), PHAssetCollection(앨범), PHCollectionList(앨범목록)   
>   
> * superclass(PHObject, PHCollection)를 직접 create 하거나, Instance를 사용하는 것은 불가능  
> * 사진, 비디오, 라이브 포토를 다루기 위해서는 하위 클래스들의 fetchMethod를 이용해서 retrive 해야만 함.