---
layout: post
title: "iOS 15 UINavigationBar appearance issue"
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

며칠동안 기다려봐도 이렇다 할 답변이 없어서 다시 하나하나 짚어가면서 문제를 찾아보기로 했다. 일단 도큐먼트에 이 문제와 관련해서 iOS 15에 업데이트 된 내용으로 보이는 건 이렇다.

> When running on apps that use iOS 14 or earlier, this property applies to navigation bars with large titles. In iOS 15, this property applies to all navigation bars.
> 

기존에는 `scrollEdgeAppearance`에 설정한 파라미터들이 네비게이션 바가 스크롤 되어서 `largeTitle`로 보일 때만 적용되다가 이제는 모든 경우를 덮어쓴다는 내용 같은데 지금 문제는 `standard`가 `scrollEdge`를 덮어쓰고 있어서 이건 문제가 안되는 것 같다. 다음으로 의심가는 부분은 `UINavigationBarAppearance`의 `shadowImage`에 관한 아래 내용이다.

> However, if this property doesn't contain a template image, the bar displays the image without applying the shadow color.
> 

template 이미지 라는 것을 사용하라고 하는데 이걸 원래는 그냥 영어를 해석해서 기본(template) 이미지 즉, 아무 설정도 안했을 때 나오는 실선 이미지라고 생각했었다. 근데 알아보니 `UIImage`에 `renderingMode`라는 파라미터가 있고, 이 옵션 중에 `.alwaysTemplate`라는 옵션이 있다고 한다.

```swift
class ViewController: UINavigationController {
	override func viewDidLoad() {
		super.viewDidLoad()
		
		navigationBar.prefersLargeTitles = true
		
		let standard = UINavigationBarAppearance()
		standard.configureWithOpaqueBackground()
		standard.backgroundColor = .systemBlue
		standard.shadowImage = UIImage(shadowColor: .black)?.withRenderingMode(.alwaysTemplate)
		navigationBar.standardAppearance = standard
		
		let scroll = UINavigationBarAppearance()
		scroll.configureWithOpaqueBackground()
		scroll.backgroundColor = .systemRed
		scroll.shadowColor = .clear
		navigationBar.scrollEdgeAppearance = scroll
	}
}
```

명시적으로 이 옵션을 주고 다시 실행해보니 잘 된다. 위 코드를 기반으로 문제를 추측해보자면 기존에는 `scrollEdgeAppearance`랑 `standardAppearance`가 독립적으로 작동했다가 iOS 15부터는 `scrollEdgeAppearance`의 설정이 `standardAppearance`를 덮어쓰기 때문인 것 같다. 예를 들어 `scrollEdgeAppearance`에만 `shadowImage`를 추가해도 `standardAppearance`에 그림자가 보인다.

![standard_nil_scoll_template.png](/assets/iOS/standard_nil_scoll_template.png)

- scrollEdgeAppearance.shadowImage = template 이미지
- standardAppearance.shadowImage = nil

`shadowImage`가 두 appearance 모두 `nil`일 경우 그림자를 그리지 않지만 iOS 15의 경우 어느 것 하나라도 `template`이 아닐 때 그 이미지를 가지고 두 appearance에 모두 그림자를 그린다. 

![standard_nil_scroll_origin.png](/assets/iOS/standard_nil_scroll_origin.png)

- scrollEdgeAppearance.shadowImage = 이미지
- standardAppearance.shadowImage = nil

그래서 현재로서는 `scrollEdgeAppearance`에는 그림자가 있고, `standardAppearance`는 그림자가 없는 형태로 만들 수가 없을 것으로 보인다.

![standard_clear_scroll_template.png](/assets/iOS/standard_clear_scroll_template.png)

iOS 14의 경우 각 appearance가 자신의 `shadowImage`, `shadowColor`에만 영향을 받기 때문에 위처럼 만들 수도 있지만 iOS 15의 경우 이번에 업데이트된 내용 때문에 이렇게 만들 수는 없게된 것 같다.

iOS 14의 `standardAppearance`

- 내 `shadowImage`가 `nil`인가? → 기본 `template` 이미지를 사용한다.
- 내 `shadowImage`가 `nil`이 아니고, `template`인가? → `shadowColor` 색으로 그림자를 그린다.
- 내 `shadowImage`가 `nil`이 아니고, `origin`인가? → 이미지로 그림자를 그린다.

iOS 15의 `standardAppearance`

- 내 `shadowImage`가 `nil`인가? → 기본 `template` 이미지를 사용한다.
- 내 `shadowImage`가 `nil`이 아니고, `template`인가? → `shadowColor` 색으로 그림자를 그린다.
- 내 `shadowImage`가 `nil`이 아니고, `origin`인가? → 이미지로 그림자를 그린다.
- `scrollEdgeAppearance`의 `shadowImage`가 `nil`이 아니고 `origin`인가? → `scrollEdgeAppearance`의 `shadowImage`로 그림자를 그린다. (이게 문제)

그래도 처음에 하려고 했던 `standardAppearance`에만 그림자가 있도록은 만들 수 있다.

![standard_template_scroll_clear.png](/assets/iOS/standard_template_scroll_clear.png)

- scrollEdgeAppearance.shadowColor = .clear
- standardAppearance.shadowImage = template 이미지

위 코드에서 그림자로 사용한 이미지들은 [이 방법](/ios/2021/10/30/샘플-UIImage-만들기.html)을 사용해서 코드로 간단하게 만들어볼 수 있다.
