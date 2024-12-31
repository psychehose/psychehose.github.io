
### 들어가기 전에

 Ounce를 개발할 때, 버튼을 누르면 밑에서 present modal 방식으로 올라오는 뷰를 만든 적이 있습니다. 다만 그때 시간이 촉박해서 애플이 기본적으로 제공하는 방식으로 구현했습니다.

잠깐 코드를 가져와보겠습니다.

```
let targetVC = ViewController()
targetVC.modalPresentationStyle = .popover
self.present(targetVC, animated: true, completion: nil)
```

이제는 좀 더 나아가서 드래그가 가능하고 이전의 뷰가 백그라운드 처리가 되어 있는 뷰를 만들어보겠습니다.

### 구현

 뷰의 전환은 ViewController → BottomCardViewController입니다.

먼저 구현에 필요한 extension을 추가하겠습니다. 화면 전환을 할 때 현재 화면(ViewController)의 스냅샷을 찍어서 화면 전환한 후 (BottomSheetViewController)의 배경화면으로 사용할 겁니다. 그리고 ViewController에서 사용할 화면전환 함수를 만들 것인데 저는 tapGestureRecognizer를 이용하도록 할게요.

```
// UIView+Snapshot.swift

extension UIView  {
    // render the view within the view's bounds, then capture it as image
  func asImage() -> UIImage {
    let renderer = UIGraphicsImageRenderer(bounds: bounds)
    return renderer.image(actions: { rendererContext in
        layer.render(in: rendererContext.cgContext)
    })
  }
}
```

```
//UIViewController.swift

let tapGesture = UITapGestureRecognizer(target: self,
					action: #selector(self.tapImageView(_:)))

@objc func tapImageView(_ sender:UITapGestureRecognizer) {
        let bottomCardViewController = BottomCardViewController()
        bottomCardViewController.modalPresentationStyle = .fullScreen
        bottomCardViewController.backgroundImage = view.asImage()
        
        self.present(bottomCardViewController, animated: false, completion: nil)
    }
```

여기까지 작성하면 ViewController에서 해야 할 일은 끝났습니다.

이제 BottomCardViewController를 만들겠습니다. BottomCardViewController는 다음과 같은 UIComponet와 Property들이 필요하다.

- 배경을 투명하게 만들어 주는 **dimmerView**: UIView
- **cardView**: UIView
- backgroundImage: UIImage - 스냅샷 이미지
- 스냅샷으로 넘겨받은 이미지를 담는 UIImageView
- **cardViewTopConstraints**: NSLayoutConstraint - 드래그를 할 때 이 값의 변화에 따라 실제로 뷰가 움직입니다.

처음에 화면 전환을 할 때, 아래에서부터 cardView가 올라와야 하기 때문에 viewDidLoad에 다음과 같이 레이아웃을 잡았습니다. cardView의 Top Constraint를 잡은 것을 보면 카드뷰의 Top이 우리가 화면에서 보는 뷰 바로 아래에 위치하는 것을 알 수 있습니다.

```
func setupViews() {
	if let safeAreaHeight = UIApplication.shared.windows.first?
            .safeAreaLayoutGuide.layoutFrame.size.height,
           let bottomPadding = UIApplication.shared.windows.first?.safeAreaInsets.bottom {
            cardViewTopConstraint = cardView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor,
                                                                  constant: safeAreaHeight + bottomPadding)
        }
        
        cardView.bottomAnchor.constraint(equalTo: view.bottomAnchor).isActive = true
        cardView.leadingAnchor.constraint(equalTo: view.leadingAnchor).isActive = true
        cardView.trailingAnchor.constraint(equalTo: view.trailingAnchor).isActive = true
        NSLayoutConstraint.activate([cardViewTopConstraint])
}
```

그러고 나서 viewDidAppear에서 **카드뷰를 올라오게 하는 함수**를 사용하면 되겠네요.

카드뷰를 올라오게 하는 함수 showCard()를 어떻게 구현하면 될까요?

- dimmerView의 alpha 값을 조절
- cardView의 topConstraint의 값을 조절

```
private func showCard() {
    self.view.layoutIfNeeded()
    if
      let safeAreaHeight = UIApplication
        .shared
        .windows.first?
        .safeAreaLayoutGuide
        .layoutFrame
        .size
        .height,
      let bottomPadding = UIApplication.shared.windows.first?.safeAreaInsets.bottom {
      cardViewTopConstraint.constant = (safeAreaHeight + bottomPadding) / 2
    }

    let showCard = UIViewPropertyAnimator(duration: 1, curve: .easeIn, animations: {
      self.view.layoutIfNeeded()
    })

    showCard.addAnimations({
      self.dimmerView.alpha = 0.7
    })
    showCard.startAnimation()
  }
```

cardViewTopConstraint.constant = (safeAreaHeight + bottomPadding) / 2를 통해서  카드뷰의 위치를 중간쯤으로 위치시켰습니다. 이렇게 딱 끝나면 좋을 것 같은데 끝이 아닙니다. topConstraint의 값은 변했지만 UI 변화는 실시간으로 반영되지 않습니다. 저는 애니메이션을 주고 싶기 때문에 카드 뷰의 위치를 실시간으로 업데이트해주고 싶어요. 뷰의 위치를 업데이트하는 방법은 layoutIfNeeded()를 이용하는 것입니다. 저는 애니메이션을 실행시키는 여러 방법 중 하나인 UIViewPropertyAnimator를 사용해서 카드뷰 위치를 업데이트하겠습니다. 그리고 화면 Dim 처리 역시 UIViewPropertyAnimator에게 넘겨주겠습니다.

이제는 바텀시트를 닫아주는 함수를 만들게요. showCard()를 이해했다면, 쉽게 알 수 있어요. 거의 똑같은 함수라서요. 차이점은 카드뷰가 내려갔을 때 BottomSheetViewControll를 dismiss 해주면 됩니다.

```
private func hideAndDismiss() {
  self.view.layoutIfNeeded()
  if
    let window = UIApplication.shared.windows.first {
    let safeAreaHeight = window.safeAreaLayoutGuide.layoutFrame.size.height
    let bottomPadding = window.safeAreaInsets.bottom
    cardViewTopConstraint.constant = (safeAreaHeight + bottomPadding)
  }
  let hideAndDismiss = UIViewPropertyAnimator(duration: 1, curve: .easeIn, animations: {
    self.view.layoutIfNeeded()
  })

  hideAndDismiss.addAnimations({
    self.dimmerView.alpha = 0.0
  })
  hideAndDismiss.addCompletion({ position in
    if position == .end {
      if(self.presentingViewController != nil) {
        self.dismiss(animated: false, completion: nil)
      }
    }
  })
  hideAndDismiss.startAnimation()
}
```

여기까지 완료 했으면, 다음과 같이 작동합니다. gif 변환 과정에서 바텀시트가 올라오는 모습이 좀 어색해졌네요.

![](https://blog.kakaocdn.net/dn/b19Hpt/btr3XzquX1H/gL1rnkKKReXJ3e2dikVml1/img.gif)

여기까지 구현한거면 큰 뼈대는 다 구현한 것이니 사실 다 만든 것이나 다름없다고 생각해요. 이제는 드래그가 가능하게 만들 겁니다.

단순히 핸들 뷰를 만들고 이것을 드래그할 때 cardViewTopConstraint가 변하는 것을 구현하면 됩니다.

어떤 것이든 상태가 있는 경우에는 대부분 Enum을 통해서 관리하는 것이 편하다고 생각해요. 그래서 cardView의 위치 또한 Enum으로 관리하도록 할게요.

```
enum CardViewState {
        case expanded // Safe Area Top에서 30pt 떨어진 상태
        case normal // (Safe Area Height + Safe Area Bottom Inset) / 2
 }
```

expanded를 어떻게 처리할까요? 이미 구현했던 showCard를 재사용하면 될 것 같습니다. 그러기 위해서 showCard를 리팩토링 하겠습니다. showCard의 매개변수는 void인데 재사용을 위해서 atState: CardViewState를 매개변수로 추가하겠습니다. 그리고나서 if문이나 switch문을 통해서 atState의 값에 따라 분기처리 하면 됩니다.

```
private func showCard(atState: CardViewState = .normal) {
        self.view.layoutIfNeeded()
        
        if let safeAreaHeight = UIApplication.shared.windows.first?
            .safeAreaLayoutGuide.layoutFrame.size.height,
           let bottomPadding = UIApplication.shared.windows.first?.safeAreaInsets.bottom {
            
            if atState == .expanded {
                cardViewTopConstraint.constant = 30.0
            } else {
                cardViewTopConstraint.constant = (safeAreaHeight + bottomPadding) / 2
            }
            cardPanStartingTopConstant = cardViewTopConstraint.constant
        }
        

        let showCard = UIViewPropertyAnimator(duration: 0.3, curve: .easeIn, animations: {
            self.view.layoutIfNeeded()
          })

        showCard.addAnimations({
            self.dimmerView.alpha = 0.7
          })
        showCard.startAnimation()

    }
```

Card View를 드래그하기 위해서는, UIPanGestureRecognizer를 등록해야 합니다.

GestureRecognizer는 사용자가 뷰에 행동을 취하는 것을 트랙킹 할 수 있도록 도와줍니다. 예를 들면 사용자가 뷰를 길게 터치하는 것, 뷰를 드래그하는 것들을 알 수 있고 그에 맞게 우리가 하고 싶은 행동(스크롤을 한다던가, 다른 뷰를 보여준다던가)을 취할 수 있게 됩니다. 

```
func panViewGesture() {
        let viewPan = UIPanGestureRecognizer(target: self, action: #selector(viewPanned(_: )))
        
        // iOS는 기본적으로 touch를 감지 (recode) 하기 전에 약간의 딜레이를 준다. 즉시 반응해야 하기 때문에 false
        viewPan.delaysTouchesBegan = false
        viewPan.delaysTouchesEnded = false
        cardView.addGestureRecognizer(viewPan)
        handleView.addGestureRecognizer(viewPan)
    }
    
    @objc func viewPanned(_ panRecognizer: UIPanGestureRecognizer) {
    	// 구현하면 될 것
    }
```

이제 viewPanned 함수를 구현해보도록 하겠습니다. UIPanGestureRecognize의 state라는 property를 이용하면 돼요. 이 state에 따라서 cardViewTopConstaraint를 변화시켜 주면 되겠죠?

코드 구현에 앞서 각 state를 어떻게 구현해야 할 지에 대해서 생각해 봅시다.

.began

- 드래그를 처음 시작했을 때의 topConstraint 값을 저장

.changed

- 드래그를 하고 있는 상태 (cardViewTopConstarint가 바뀌는 중)
- topConstraint가 expanded mode(30) 보다 작아서는 안됨
- 드래그를 따라서, dimmer View의 alpha 값 조절

.ended

- 드래그가 끝난 후 액션 처리

그리고 추가적으로 만약 사용자가 아래로 드래그를 빠르게 했을 때 (Snap) 바텀시트를 닫으면 사용자 관점에서 엄청 편할 거예요. 얼마나 빠르게 뷰를 드래그했는지 velocity라는 변수를 통해서 알 수 있습니다. 그래서 이것을 감지한다면 바텀시트를 닫아주면 돼요.

```
@objc func viewPanned(_ panRecognizer: UIPanGestureRecognizer) {
  let translation = panRecognizer.translation(in: view)
  let velocity = panRecognizer.velocity(in: view)

  switch panRecognizer.state {
  case .began:
    cardPanStartingTopConstant = cardViewTopConstraint.constant

  case .changed:
    if cardPanStartingTopConstant + translation.y > 30.0 {
      cardViewTopConstraint.constant = cardPanStartingTopConstant + translation.y
    }

  case .ended:
    if velocity.y > 1500.0 {
      hideAndDismiss()
      return
    }


    if
      let safeAreaHeight = UIApplication
        .shared.windows
        .first?
        .safeAreaLayoutGuide
        .layoutFrame
        .size.height,
       let bottomPadding = UIApplication.
        shared.windows.
        first?.
        safeAreaInsets.
        bottom {

      if cardViewTopConstraint.constant < (safeAreaHeight + bottomPadding) * 0.25 {
        showCard(atState: .expanded)
      } else if cardViewTopConstraint.constant < safeAreaHeight - 70 {
        showCard(atState: .normal)
      } else {
        hideAndDismiss()
      }
    }
  default:
    break
  }
}
```

여기까지 따라오셨으면, 다음과 같은 결과물을 얻을 수 있습니다.

![](https://blog.kakaocdn.net/dn/K8K6s/btr3ICC7PYS/9aMTgKNES93blUUkqi2tC0/img.gif)

### 전체코드 및 레퍼런스

```
//
//  BottomCardViewController.swift
//  DraggableBottomCard
//
//  Created by psychehose on 2021/06/19.
//

import UIKit

class BottomCardViewController: UIViewController {
    
    enum CardViewState {
        case expanded
        case normal
    }
    var cardViewState: CardViewState = .normal
    
    private var imageView: UIImageView = {
        let imageView = UIImageView()
        imageView.translatesAutoresizingMaskIntoConstraints = false
        return imageView
    }()

    var dimmerView: UIView = {
        let dimmerView = UIView()
        dimmerView.alpha = 0.0
        dimmerView.backgroundColor = .gray
        dimmerView.translatesAutoresizingMaskIntoConstraints = false
        return dimmerView
    }()
    
    var cardView: UIView = {
        let cardView = UIView()
        cardView.translatesAutoresizingMaskIntoConstraints = false
        cardView.backgroundColor = .white
        cardView.clipsToBounds = true
        cardView.layer.cornerRadius = 10.0
        cardView.layer.maskedCorners = [.layerMinXMinYCorner, .layerMaxXMinYCorner]
        return cardView
    }()
    
    var handleView: UIView = {
        let handleView = UIView()
        handleView.layer.cornerRadius = 3.0
        handleView.clipsToBounds = true
        handleView.translatesAutoresizingMaskIntoConstraints = false
        handleView.backgroundColor = .gray
        return handleView
    }()
    var cardViewTopConstraint: NSLayoutConstraint!
    var cardPanStartingTopConstant: CGFloat = 30.0

    var backgroundImage: UIImage?



    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .white
        imageView.image = backgroundImage
        configureLayout()
        tapBackgroundImageGesture()
        panViewGesture()
    }
    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
          showCard()
    }

    
}
    // MARK: - Gesture

extension BottomCardViewController {
    func tapBackgroundImageGesture() {
        let tapGesture = UITapGestureRecognizer(target: self, action: #selector(tapImageView(_: )))
        imageView.addGestureRecognizer(tapGesture)
        imageView.isUserInteractionEnabled = true
    }

    @objc func tapImageView(_ sender: UITapGestureRecognizer) {
        hideAndDismiss()
    }

    func panViewGesture() {
        let viewPan = UIPanGestureRecognizer(target: self, action: #selector(viewPanned(_: )))
        
        // iOS는 기본적으로 touch를 감지 (recode) 하기 전에 약간의 딜레이를 준다. 즉시 반응해야 하기 때문에 false
        viewPan.delaysTouchesBegan = false
        viewPan.delaysTouchesEnded = false
        cardView.addGestureRecognizer(viewPan)
        handleView.addGestureRecognizer(viewPan)
    }

    @objc func viewPanned(_ panRecognizer: UIPanGestureRecognizer) {
        let translation = panRecognizer.translation(in: view)
        let velocity = panRecognizer.velocity(in: view)
        
        switch panRecognizer.state {
        case .began:
            cardPanStartingTopConstant = cardViewTopConstraint.constant
        case .changed:
            if cardPanStartingTopConstant + translation.y > 30.0 {
                cardViewTopConstraint.constant = cardPanStartingTopConstant + translation.y
            }
            
            self.dimmerView.alpha =
                dimmerAlphaWithTopConstant(value: cardViewTopConstraint.constant)
            
            
        case .ended:
            if velocity.y > 1500.0 {
                hideAndDismiss()
                return
            }
            
            
            if let safeAreaHeight = UIApplication.shared.windows.first?
                .safeAreaLayoutGuide.layoutFrame.size.height,
               let bottomPadding = UIApplication.shared.windows.first?.safeAreaInsets.bottom {
                
                if cardViewTopConstraint.constant < (safeAreaHeight + bottomPadding) * 0.25 {
                    showCard(atState: .expanded)
                    
                } else if cardViewTopConstraint.constant < safeAreaHeight - 70 {
                    showCard(atState: .normal)
                } else {
                    hideAndDismiss()
                }
            }
            
        default:
            break
        }
        
    }
}


    // MARK: - UILayout

extension BottomCardViewController {

    func configureLayout() {
        view.addSubview(imageView)
        imageView.addSubview(dimmerView)
        view.addSubview(cardView)
        view.addSubview(handleView)
        
        imageView.topAnchor.constraint(equalTo: view.topAnchor).isActive = true
        imageView.bottomAnchor.constraint(equalTo: view.bottomAnchor).isActive = true
        imageView.leadingAnchor.constraint(equalTo: view.leadingAnchor).isActive = true
        imageView.trailingAnchor.constraint(equalTo: view.trailingAnchor).isActive = true
        
        dimmerView.topAnchor.constraint(equalTo: imageView.topAnchor).isActive = true
        dimmerView.bottomAnchor.constraint(equalTo: imageView.bottomAnchor).isActive = true
        dimmerView.leadingAnchor.constraint(equalTo: imageView.leadingAnchor).isActive = true
        dimmerView.trailingAnchor.constraint(equalTo: imageView.trailingAnchor).isActive = true
        
        
        if let safeAreaHeight = UIApplication.shared.windows.first?
            .safeAreaLayoutGuide.layoutFrame.size.height,
           let bottomPadding = UIApplication.shared.windows.first?.safeAreaInsets.bottom {
            cardViewTopConstraint = cardView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor,
                                                                  constant: safeAreaHeight + bottomPadding)
        }
        
        cardView.bottomAnchor.constraint(equalTo: view.bottomAnchor).isActive = true
        cardView.leadingAnchor.constraint(equalTo: view.leadingAnchor).isActive = true
        cardView.trailingAnchor.constraint(equalTo: view.trailingAnchor).isActive = true
        NSLayoutConstraint.activate([cardViewTopConstraint])
        
        handleView.centerXAnchor.constraint(equalTo: cardView.centerXAnchor).isActive = true
        handleView.widthAnchor.constraint(equalToConstant: 60).isActive = true
        handleView.heightAnchor.constraint(equalToConstant: 6).isActive = true
        handleView.bottomAnchor.constraint(equalTo: cardView.topAnchor, constant: -10).isActive = true
    }
    
}

extension BottomCardViewController {
    private func showCard(atState: CardViewState = .normal) {
        self.view.layoutIfNeeded()
        
        if let safeAreaHeight = UIApplication.shared.windows.first?
            .safeAreaLayoutGuide.layoutFrame.size.height,
           let bottomPadding = UIApplication.shared.windows.first?.safeAreaInsets.bottom {
            
            if atState == .expanded {
                cardViewTopConstraint.constant = 30.0
            } else {
                cardViewTopConstraint.constant = (safeAreaHeight + bottomPadding) / 2
            }
            cardPanStartingTopConstant = cardViewTopConstraint.constant
        }
        

        let showCard = UIViewPropertyAnimator(duration: 0.3, curve: .easeIn, animations: {
            self.view.layoutIfNeeded()
          })

        showCard.addAnimations({
            self.dimmerView.alpha = 0.7
          })
        showCard.startAnimation()

    }
    private func hideAndDismiss() {
        self.view.layoutIfNeeded()
        if let safeAreaHeight = UIApplication.shared.windows.first?
            .safeAreaLayoutGuide.layoutFrame.size.height,
           let bottomPadding = UIApplication.shared.windows.first?.safeAreaInsets.bottom {
            cardViewTopConstraint.constant = (safeAreaHeight + bottomPadding)
        }
        let hideAndDismiss = UIViewPropertyAnimator(duration: 0.3, curve: .easeIn, animations: {
            self.view.layoutIfNeeded()
          })
        
        hideAndDismiss.addAnimations({
            self.dimmerView.alpha = 0.0
          })
        hideAndDismiss.addCompletion({ position in
            if position == .end {
              if(self.presentingViewController != nil) {
                self.dismiss(animated: false, completion: nil)
              }
            }
          })
        hideAndDismiss.startAnimation()
    }
}

extension BottomCardViewController {
    private func dimmerAlphaWithTopConstant(value: CGFloat) -> CGFloat {
        let fullDimAlpha: CGFloat = 0.7
        
        guard let safeAreaHeight = UIApplication.shared.windows.first?
                .safeAreaLayoutGuide.layoutFrame.size.height,
              let bottomPadding = UIApplication.shared.windows.first?.safeAreaInsets.bottom else {
            return fullDimAlpha
        }
        
        let fullDimPosition = (safeAreaHeight + bottomPadding) / 2.0
        
        let noDimPosition = safeAreaHeight + bottomPadding
        
        if value < fullDimPosition {
            return fullDimAlpha
        }
        if value > noDimPosition {
            return 0.0
        }
        return fullDimAlpha * 1 - ((value - fullDimPosition) / fullDimPosition)
    }
}
```

[https://fluffy.es/facebook-draggable-bottom-card-modal-1/](https://fluffy.es/facebook-draggable-bottom-card-modal-2/)

 [Replicating Facebook's Draggable Bottom Card using Auto Layout - Part 2/2

In Part 1, we have managed to implement the show and hide card animation when user tap on button or the dimmer view. In this part, we are going to implement the card dragging animation. This post assume that you already knew about Auto Layout and Delegate.

fluffy.es](https://fluffy.es/facebook-draggable-bottom-card-modal-2/)

[https://fluffy.es/facebook-draggable-bottom-card-modal-2/](https://fluffy.es/facebook-draggable-bottom-card-modal-2/)

 [Replicating Facebook's Draggable Bottom Card using Auto Layout - Part 2/2

In Part 1, we have managed to implement the show and hide card animation when user tap on button or the dimmer view. In this part, we are going to implement the card dragging animation. This post assume that you already knew about Auto Layout and Delegate.

fluffy.es](https://fluffy.es/facebook-draggable-bottom-card-modal-2/)