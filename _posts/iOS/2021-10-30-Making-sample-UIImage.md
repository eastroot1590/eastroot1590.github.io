---
layout: post
title: "샘플 UIImage 만들기"
data: 2021-10-30 23:58:00 +0900
categories: iOS
---

`UIBarAppearance`나 기타 등등 필요한 곳에 사용할 수 있도록 간단한 `UIImage`를 코드로 만드는 방법을 알아보자.

## 단일 색상(solid color)

```swift
extension UIImage {
	convenience init?(color: UIColor, size: CGSize = CGSize(width: 1, height: 1)) {
		let rect = CGRect(origin: .zero, size: size)
		
		UIGraphicsBeginImageContextWithOptions(size, false, 0)
		
		color.setFill()
		UIRectFill(rect)
		
		let image = UIGraphicsGetImageFromCurrentImageContext()
		
		UIGraphicsEndImageContext()
		
		guard let cgImage = image?.cgImage else {
			return nil
		}
		
		self.init(cgImage: cgImage)
	}
}
```

## 그라데이션(gradient)

그림자에 사용할 수 있다.

```swift
extension UIImage {
	convenience init?(color: UIColor, size: CGSize = CGSize(width: 1, height: 1)) {
		UIGraphicsBeginImageContextWithOptions(size, false, 0)
		
		if let context = UIGraphicsGetCurrentContext() {
			let layer = CAGradientLayer()
			layer.frame = CGRect(origin: .zero, size: size)
			layer.colors = [color.cgColor, CGColor(gray: 0, alpha: 0)]
			layer.locations = [0, 0.8]
			layer.opacity = opacity
			layer.render(in: context)
		}
		
		let image = UIGraphicsGetImageFromCurrentImageContext()
		
		guard let cgImage = image?.cgImage else {
			return nil
		}
		
		self.init(cgImage: cgImage)
	}
}
```
