---
layout: post
title: "Stackable"
data: 2021-06-21 21:24:00 +0900
categories: iOS
---

SwiftUI가 발표되면서 가장 먼저 눈에 들어오는 기능은 `VStack`, `HStack`같은 view였다. 그 전까지 UIKit를 사용해서 화면을 구성할 때는 frame을 지정하고 `NSLayoutConstraint`라는 좌표 규칙 같은걸 지정해야 view를 화면에 그릴 수 있었는데 정말 직관적인 단 몇 줄의 코드로 자동으로 autolayout이 되는 화면을 구성할 수 있었다. SwiftUI를 직접 사용하면 좋겠지만 아직 익숙하지 않아서 UIKit로 비슷하게 만들어보자.

## Stackable

먼저 protocol을 사용해서 쌓을 수 있는(Stackable)한 view가 가질 기능들을 정의한다. 필요한 기능들을 생각해보자. 가장 먼저 떠오르는 건 마지막에 추가한 view를 저장할 변수가 필요할 것이고, view를 추가할 수 있는 함수가 필요하다. 두가지만 추가해도 좋지만 정렬이 있으면 좋을 것 같다는 생각이 든다. `UIView`에는 `ContentMode`라는 자료형이 있는데, 정확히 원하는 정렬만 제공하지는 않지만 `center`, `left`, `bottom` 사용하기에 큰 문제는 없는 자료형이 있기 때문에 그대로 사용하기로 했다.

```swift
protocol Stackable: UIView {
	var alignment: UIView.ContentMode { get set }
	
	var stackTop: UIView? { get }
		
	func push(_ stack: UIView, spacing: CGFloat)
}
```

`push()`는 단순히 view만 파라미터로 받지 않고 `spacing`이라는 값을 받아서, 자동으로 스택에 추가하지만 사용자가 원하는 만큼 띄울 수 있는 자유도를 제공한다.

바로 이 protocol을 채택하는 구체 클래스를 만들어 보도록 하자. 휴대폰 화면은 세로(vertical)이기 때문에 `VStack`을 먼저 만들어 보자.

```swift
class VStack: UIView, Stackable {
	var alignment: UIView.ContentMode = .center
	
	var stackTop: UIView?

	func push(_ stack: UIView, spacing: CGFloat) {

	}
}
```

`push()`에 레이아웃을 추가하는 코드를 작성해야 하는데 작성하기 전에 로직을 한번 생각 해보자. `alignment`에 따라 왼쪽으로 치우치게 추가해야 할 수도 있고, 크기가 고정되지 않았을 때 양 끝으로 가득 채우도록 해야 한다. if문도 몇개 필요할 것 같고 특히 `NSLayoutConstraint`를 설정하는 코드가 그 조건문 안에 많이 들어가게 될 것 같다.

간단하게 윗면을 붙이는 코드만 하더라도 벌써 3줄이 넘어간다.

```swift
stack.translatesAutoresizingMaskIntoConstraint = false
addSubview(stack)
NSLayoutConstraint.activate([
	stack.topAnchor.constraint(equalTo: stackTop.bottomAnchor, constant: spacing)
])
```

최근 **Clean Code**라는 책을 읽고 있는데 이 책에서 좋은 코드는 위에서부터 순서대로 읽었을 때 동작을 유추할 수 있는 코드라고 한다. 따라서 가독성이 떨어지는 layout 코드들은 모두 걷어내고 최대한 읽기 좋고 짧게 작성해봤다.

```swift
func push(_ stack: UIView, spacing: CGFloat) {
	attachToStackTop(stack, spacing: spacing)
	
	if stack.frame.width > 0 {
		horizontalLayoutToFit(stack)
	} else {
		horizontalLayoutToFill(stack)
	}
}
```

코드 추상화 단계도 맞추기 위해서 모든 layout 코드를 함수로 뺐고, 간결하게 작성해보았다. 세로(Vertical) stack인 만큼 윗면을 먼저 붙이고, 가로(Horizontal) layout을 설정한다. 각 함수 내부에서 필요한 작업을 수행한다.

```swift
private func attachToStackTop(_ stack: UIView, spacing: CGFloat) {
	var verticalConstraint: [NSLayoutConstraint] = []
	stack.translatesAutoresizingMaskIntoConstraints = false
	addSubview(stack)
	verticalConstraint.append(stack.topAnchor.constraint(equalTo: stackTop?.bottomAnchor ?? topAnchor, constant: spacing))
	
	if stack.frame.height > 0 {
		verticalConstraint.append(stack.heightAnchor.constraint(equalToConstant: stack.frame.height))
	}
	
	// 자체크기 확장
	if let bottom = constraints.first(where: { $0.firstAttribute == .bottom }) {
		NSLayoutConstraint.deactivate([bottom])
	}
	verticalConstraint.append(bottomAnchor.constraint(equalTo: stack.bottomAnchor))
	
	NSLayoutConstraint.activate(verticalConstraint)
	
	stackTop = stack
}
```

생각보다 코드가 길어보이기는 하지만 더이상 분리하면 더 복잡해질 수도 있을 것 같아서 멈췄다. 특히 자체 크기 확장 layout을 하는 코드는 엄밀히 말하면 `attachToStackTop()`이라는 이름이랑 맞지 않는 코드인 것 같아서 마음에 걸리긴 한다.

`horizontalLayoutToFit()`은 넓이가 0이 아닌 즉, 자신이 넓이를 알고 있는 경우 `VStack`의 정렬에는 맞추지만 넓이는 그대로 유지할 수 있도록 layout하는 코드를 작성한다.

```swift
private func horizontalLayoutToFit(_ stack: UIView) {
	var horizontalConstraints: [NSLayoutConstraint] = []
	horizontalConstraint.append(stack.widthAnchor.constraint(equalToConstant: stack.frame.width))
	
	switch contentMode {
	case .center:
		horizontalConstraints.append(stack.centerXAnchor.constraint(equalTo: centerXAnchor))
	case .left:
		horizontalConstraints.append(stack.leadingAnchor.constraint(equalTo: leadingAnchor))
	case .right:
		horizontalConstraints.append(stack.trailingAnchor.constraint(equalTo: trailingAnchor))
	
	default:
	break
	}
	
	NSLayoutConstraint.activate(horizontalConstraints)
}
```

바로 아래에서 설명할 `horizontalLayoutToFill()`도 굳이 함수로 분리했어야 하나 하는 생각이 들 정도로 별로 특별한 작업을 안한다.

```swift
private func horizontalLayoutToFill(_ stack: UIView) {
	NSLayoutConstraint.activate([
		stack.leadingAnchor.constraint(equalTo: leadingAnchor),
		stack.trailingAnchor.constraint(equalTo: trailingAnchor)
	])
}
```

하지만 이 함수들을 분리하지 않고 `push()`에 그대로 작성했다면 추상화 단계가 뒤섞이게 되어 결과적으로 `push()`의 가독성을 떨어뜨리게 된다. 위에서 보여준 `push()`와 아래 `push()`를 비교해보자.

```swift
func push(_ stack: UIView, spacing: CGFloat) {
	attachToStackTop(stack, spacing: spacing)
	
	var horizontalConstraints: [NSLayoutConstraint] = []
	if stack.frame.width > 0 {
		horizontalConstraint.append(stack.widthAnchor.constraint(equalToConstant: stack.frame.width))
		
		switch contentMode {
		case .center:
			horizontalConstraints.append(stack.centerXAnchor.constraint(equalTo: centerXAnchor))
		case .left:
			horizontalConstraints.append(stack.leadingAnchor.constraint(equalTo: leadingAnchor))
		case .right:
			horizontalConstraints.append(stack.trailingAnchor.constraint(equalTo: trailingAnchor))
		
		default:
		break
		}
		NSLayoutConstraint.activate(horizontalConstraints)
	} else {
		NSLayoutConstraint.activate([
			stack.leadingAnchor.constraint(equalTo: leadingAnchor),
			stack.trailingAnchor.constraint(equalTo: trailingAnchor)
		])
	}
}
```

결과적으로 로직은 똑같고, 오히려 함수를 호출하지 않기 때문에 더 효율적이라고 할 수도 있지만 `push()`라는 함수가 하는 일을 알아내기 위해서는 굳이 layout 코드를 모두 읽고 이해하는 과정이 필요하다. 

### Use

`VStack`은 자동으로 자신의 크기를 계산할 수 있기 때문에 조금의 제약(Constraint)만으로도 쉽게 화면을 구성할 수 있다.

```swift
let stack = VStack()
stack.translatesAutoresizingMaskIntoConstraint = false
NSLayoutConstraint.activate([
	stack.topAnchor.constraint(equalTo: view.topAnchor, constant: 50),
	stack.leadingAnchor.constraint(equalTo: view.leadingAnchor),
	stack.trailingAnchor.constraint(equalTo: view.trailingAnchor)
])

for _ in 0..<10 {
	let view = UIView()
	view.backgroundColor = .black
	view.frame.size = CGSize(width: 0, height: 50)

	stack.push(view, spacing: 10)
}
```

`VStack`자체를 추가할 때는 어쩔 수 없이 `Constraint`코드가 들어가긴 했지만 그 다음부터 stack에 view를 추가할 때는 view의 크기만 정해주면 된다.

![VStackExample](/assets/iOS/Stackable_vStackExample.png){:.aligncenter height="700"}

## VStackScroll

이렇게 만든 `VStack`을 사용하면 자동으로 스크롤 범위가 늘어나는 스크롤뷰를 쉽게 만들어볼 수 있다. `UIScrollView`는 안에 있는 `contentView`의 크기에 따라서 스크롤 범위가 늘어나기 때문에 `VStack`을 `contentView`로 하용하면 자동으로 스크롤 범위가 늘어나도록 만들 수 있다.

```swift
class VStackScroll: UIScrollView {
	let contentView: VStackView = VStackView()
	
	init() {
		super.init(frame: .zero)
		
		contentView.translatesAutoresizingMaskIntoConstraints = false
		addSubview(contentView)
		// 중요
		let height = contentView.heightAnchor.constraint(equalTo: heightAnchor)
		height.priority = .defaultHigh
		NSLayoutConstraint.activate([
			contentView.topAnchor.constraint(equalTo: topAnchor),
			contentView.centerXAnchor.constraint(equalTo: centerXAnchor),
			contentView.widthAnchor.constraint(equalTo: widthAnchor),
			height
		])
	}
	
	func push(_ stack: UIView, spacing: CGFloat) {
		contentView.push(stack, spacing: spacing)
		
		contentView.layoutIfNeeded()
		contentSize = contentView.frame.size
	}
}
```

`Constraint` 코드에서 높이 제약이 중요한데 `contentView.bottomAnchor.constraint(equalTo: bottomAnchor)`로 설정해도 스크롤 영역은 자동으로 확장되지만 view debug를 해보면 warning 비슷한게 나와서 거슬린다.

![LayoutIssue](/assets/iOS/Stackable_layoutIssue.png){:.aligncenter}

height를 애매하지 않게 제약조건을 추가해주면 해당 warning이 사라진다.

![vStackScrollExample](/assets/iOS/Stackable_vStackScrollExample.mp4){:.aligncenter height="500"}

다음에는 이렇게 추가하는 view들이 한번에 추가되는게 아니라 위에서부터 순서대로 시간간격을 두고 추가되는 것처럼 보이게 애니메이션을 추가해보자.