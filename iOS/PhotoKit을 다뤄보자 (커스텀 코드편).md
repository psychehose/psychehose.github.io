
이번에는 직접 Photo 프레임워크를 이용해서 사용자의 앨범에 접근을 해서 collectionView에 사진을 불러오고 카메라에 접근을 해서 촬영을 하고 촬영한 이미지를 불러오겠습니다. 컬렉션 뷰의 첫 번째 아이템(카메라 이미지)을 선택해서 카메라로 넘어가서 촬영하거나, 그 외 아이템을 클릭하면 해당하는 이미지를 헤더에 띄우는 것입니다. PhoKit의 사용법을 알면 다음과 같은 화면을 구현할 수 있게 됩니다. 인스타그램 업로드 화면 같은 뷰인데, 이미지를 띄우는 것 자체는 정말 간단합니다. 스크롤을 다룰 경우 좀 복잡해지는 것 같습니다. 

![](https://blog.kakaocdn.net/dn/duKNv8/btr3IYRGSGx/uL1BCd1x4FYtj26p5jHsy0/img.gif)

**중요! 구현 하기에 앞서 앱이 유저의 카메라와, 앨범에 접근하려면 권한이 필요합니다. Info.plist로 이동해서 Key들을 추가해 주세요.**

![](https://blog.kakaocdn.net/dn/RdKg1/btr3ky1Qbev/AJH4esSzjKHQ6Po2hfAjmk/img.png)

검정 버튼이 있는 화면을 TempViewController라고 작명할게요. 화면 전환을 했을 때, 사진들이 Load 되어 있는 컬렉션 뷰가 나타나게 하기 위해서 저는 이전 화면인 TempViewController에서 미리 Fetch를 하겠습니다. 저는 viewDidLoad에서 fetchAsset()을 했는데, 버튼을 클릭했을 때 비동기처리를 하는 것, TestCollectionViewController의 생성자로 PHAsset를 주입하는 것이 더 좋은 코드였겠죠? Sample 앱이라 그냥 넘어가도록 하겠습니다.

 제가 구현한 fetchAsset() 함수 내부를 살펴보면, PHFetchOptions()의 Instance를 생성하고 옵션 값을 여러 가지 설정하고 PHAsset의 class function인 fetchAssets() 사용하는 걸 알 수 있습니다. 이론 편에서 확인한 PHObject 같은 것들은 저희는 코드 내에서 직접 작성하는 경우가 거의 없다고 할 수 있습니다. 그 하위 클래스인 PHAsset만을 사용하는 것이죠. 특이한 점은 Instance를 생성하지 않는 것입니다. 사실 아이폰 사진 앱 기능을 생각해 보면 더 효율적이라는 생각이 들긴 드네요. 안드로이드 개발을 잠깐 찾아봤을 때, 흩어져있는 사진들을 모으는 게 좀 힘들었다고 본 것 같습니다. 아이폰 같은 경우는 모든 사진이 앨범에 모이기 때문에 애플 측에서 이런 방식으로 구현한 게 아닐까 싶습니다. 

```
import Photos
import UIKit

class TempVC: UIViewController {

	let btn: 생략
	var allPhotos: PHFetchResult<PHAsset>!

    override func viewDidLoad() {
        super.viewDidLoad()

        //...//

        fetchAsset()
    }

    private func fetchAsset() {
        let allPhotoOptions = PHFetchOptions()
        allPhotoOptions.sortDescriptors = [NSSortDescriptor(key: "creationDate", ascending: true)]
        allPhotos = PHAsset.fetchAssets(with: allPhotoOptions)

    }
    private func tapButton() {
        var navigationController: UINavigationController
        let targetVC = TestCollectionViewController()

        // 여기가 핵심. targetVC에 있는 변수인 fetchResult에 넘겨준다.
        
        targetVC.fetchResult = allPhotos

        navigationController = UINavigationController(rootViewController: targetVC)
        navigationController.modalPresentationStyle = .fullScreen

        present(navigationController, animated: true)
    }

}
```

다음으로는, PHFetchResult을 넘겨받아서 이미지를 collectionView에 띄어주는 ViewController를 구현하겠습니다. collectionView생성 및 화면 배치는 다들 익숙하니까 생략하도록 할게요.

 이전 챕터에서 언급했듯이, fetchResult의 결과는 메타데이터(정보)입니다. 실제 이미지를 받아 와야 하므로 이것을 기반으로 Image를 요청해야 합니다. 그러기 위해서 PHImageManager라는 것이 필요합니다. PHImageManger에 구현되어 있는 method requestImage를 통해 이미지를 받아올 수 있습니다. 

```
class TestCollectionViewController: UIViewController {
	// UIComponent

	var pictureCV = UICollectionView().then { }

	// Variable & properties

	var fetchResult: PHFetchResutl<PHAsset>!
    var photoImage: UIImage? // 여기에 카메라로 찍은 사진 저장

	fileprivate let imageManager = PHImageManager()
	fileprivate var thumbnailSize: CGSize!

	//MARK: - View Life Cycle

    override func viewDidLoad() {
        super.viewDidLoad()
        //...생략 //
    }

    override func viewWillAppear(_ animated: Bool) {
        let scale = UIScreen.main.scale
        let cellSize = pictureCV.size
        thumbnailSize = CGSize(width: cellSize.width * scale,
        					   height: cellSize.height * scale)

    }
}
extension TestCollectionViewController: UICollectionViewDataSource {
	func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
        return fetchResult.count + 1
        // 카메라 이미지를 넣어야 하기 때문에,
    }
    func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
    guard let pictureCell =
                collectionView.dequeueReusableCell(withReuseIdentifier: "PictureCell",
                                                   for: indexPath) as? PictureCell else
        { return UICollectionViewCell() }
        if indexPath.item == 0 {
        	pictureCell.thumbnailImage = UIImage(systemName: "camera")
            return pictureCell
        } else {
        	let asset = fetchResult.object(at: (indexPath.item)-1 )

            pictureCell.representedAssetIdentifier = asset.localIdentifier
            imageManager.requestImage(for: asset, targetSize: thumbnailSize, contentMode: .aspectFill, options: nil, resultHandler: { image, _ in
                if pictureCell.representedAssetIdentifier == asset.localIdentifier {
                    pictureCell.thumbnailImage = image
                }
            })

            return pictureCell
        }
    }
}
```

  여기에서 가장 중요한 부분은 당연하게 cellForItemAt 부분입니다. 이미지를 불러오기 위해서는 다음과 같은 작업이 필요합니다.   
이전 화면에서 넘겨받은 fetchResult의 메타데이터를 가지고 imageManager를 통해서 요청하면 CallBack 형식으로 UIImage에 접근할 수 있습니다.

그리고 코드를 확인해 보면 Asset의 localIdentifier(메타데이터)가 보이는데, 이 프로퍼티는 Unique 합니다. 이 부분을 잘 이용하면, Cell 이벤트를 잘 처리할 수 있을 것 같습니다.

마지막으로 카메라 이미지를 선택했을 때 이미지에 접근하는 법입니다. 애플에서 제공하는 UIImagePickerComtroller를 사용하게 되면, UIImagePickerControllerDelegate를 통해서 클로저로 간단하게 Image에 접근할 수 있습니다. 커스텀할 필요가 없다면 사용하는 것이 좋을 것 같습니다.

```
class TestCollectionViewCotroller: UIViewController {

///...//
	func didTapCamera() {
		let camera = UIImagePickerController()
        	camera.sourceType = .camera
        	camera.cameraDevice = .rear
        	camera.cameraCaptureMode = .photo
        	camera.delegate = self
        	present(camera, animated: true, completion: nil)
            }

}

extension TestCollectionViewCotroller: UIImagePickerControllerDelegate & UINavigationControllerDelegate {

    func imagePickerController(_ picker: UIImagePickerController, didFinishPickingMediaWithInfo info: [UIImagePickerController.InfoKey : Any]) {
        // 미디어 종류 확인
        let mediaType = info[UIImagePickerController.InfoKey.mediaType] as! NSString
        // 미디어가 사진이면
        if mediaType.isEqual(to: kUTTypeImage as NSString as String){
            // 사진을 가져옴
            let captureImage = info[UIImagePickerController.InfoKey.originalImage] as! UIImage

            // 사진을 포토 라이브러리에 저장. 안하려면 없애주면 된다.
            UIImageWriteToSavedPhotosAlbum(captureImage, self, nil, nil)
            photoImage = captureImage

        }
        // 현재의 뷰(이미지 피커) 제거
        self.dismiss(animated: true, completion: {
            self.didTapReset()
            let targetVC = tempCV()
            targetVC.photoImage = self.photoImage
            self.navigationController?.pushViewController(targetVC, animated: true)

        })
    }

    func imagePickerControllerDidCancel(_ picker: UIImagePickerController) {
        self.dismiss(animated: true, completion: nil)
    }

}
```