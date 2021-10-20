---
layout: post
title: "iOS 15 UINavigationBar appearance issue (진행중)"
data: 2021-10-20 21:03:00 +0900
categories: iOS
---

## 문제

Xcode 13으로 업데이트 한 후 스타벅스 프로젝트의 모든 `largeTitle` 화면에서 `navigationBar`의 그림자가 이상하게 동작하는 문제가 생겼다. 같은 Xcode 13으로 빌드하더라도 iOS 14 또는 더 낮은 버전에서 실행하면 기존과 같이 정상적으로 동작하지만 iOS 15 이상 디바이스에서 실행하면 그림자가 깨지는 문제가 발생했다.

|![when iOS 15.png](/assets/iOS/when_iOS_15.png) | ![when iOS 14.png](/assets/iOS/when_iOS_14.png)|
|:---:|:---:|
|Xcode 13에서 iOS 15에 구동 | Xcode 13에서 iOS 14에 구동|

## 연구

`UINavigationBar`는 `standardAppearance`, `scrollEdgeAppearance`, `compactAppearance` 등 여러가지 상황에 따라 표시되는 모습을 사전정의할 수 있게 되어 있는데, 위 사진의 빨간색으로 표시되는 형태를 `scrollEdgeAppearance`, 파란색으로 표시되는 형태를 `standardAppearance`라고 한다.

그리고 위 예시는 정확하지는 않지만 두 appearance를 수정해서 `standardAppearance`일 경우에만 그림자를 출력하도록 작성하고 실행한 모습이다.

```swift
class ViewController: UINavigationController {
	override func viewDidLoad() {
		super.viewDidLoad()
		
		navigationBar.prefersLargeTitles = true
		
		let standard = UINavigationBarAppearance()
		standard.configureWithOpaqueBackground()
		standard.backgroundColor = .systemBlue
		standard.shadowImage = UIImage(color: .black)
		standard.shadowColor = .clear
		navigationBar.standardAppearance = standard
		
		let scroll = UINavigationBarAppearance()
		scroll.configureWithOpaqueBackground()
		scroll.backgroundColor = .systemRed
		scroll.shadowImage = UIImage()
		scroll.shadowColor = .clear
		navigationBar.scrollEdgeAppearance = scroll
	}
}
```

혼자서 도큐먼트 같은걸 찾아보면서 알아봤지만 도저히 납득할만한 업데이트 내용도 없었다. iOS 15에서 관련된 부분이 변경된 거라고는 `scrollEdgeAppearance`가 기존에는 `largeTitle`일 경우에만 적용되었지만 이제는 모든 경우에 적용된다는 [문서](https://developer.apple.com/documentation/uikit/uinavigationbar/3198027-scrolledgeappearance?language=objc) 뿐이었는데 무슨 말인지는 정확히 이해하지는 못했지만 위 경우와는 상관 없는 것으로 보인다. 실제로 Xcode 12에서 iOS 15에 똑같은 코드를 돌려보면 또 정사적으로 동작을 잘 한다.

먼저 스택오버플로우에 [질문글](https://stackoverflow.com/questions/69572120/how-to-make-separated-shadow-appearance-for-uinavigationbar-in-xcode-13-ios-15)을 올려봤다.

딱히 답이 없어서 애플 개발자 포럼에도 [질문글](https://developer.apple.com/forums/thread/692339)을 올려봤다. 애플 개발자로 보이는 사람이 달아준 답글에는 애플 측 버그인 것으로 보이니 버그리포팅을 해 달라는 내용이 적혀있었고, 애플 피드백 사이트에 [리포트](https://feedbackassistant.apple.com/feedback/9705096)를 업로드 했다.

추후 진행상황이 업데이트 되면 내용을 추가해야겠다.