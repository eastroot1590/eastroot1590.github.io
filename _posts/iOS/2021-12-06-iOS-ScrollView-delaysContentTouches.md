---
layout: post
title: "UIScrollView delaysContentTouches"
data: 2021-12-6 14:32:00 +0900
categories: iOS
---

AppStore 앱 투데이 탭은 여러 뉴스를 피드 형식으로 보여주는 화면이다. 이 피드는 스크롤할 수 있으며, 해당 피드를 선택하면 상세 화면으로 이동한다. 이 때 사용자 터치에 대해 피드가 줄어들었다가 다시 커지는 효과가 적용되어 굉장히 부드럽게 화면이 전환되는 느낌을 준다.

`UICollectionView`로에는 사용자의 입력에 따라,

```swift
func collectionView(_ collectionView: UICollectionView, didHighlightItemAt indexPath: IndexPath)
func collectinoView(_ collectionView: UICollectionView, didUnhighlightItemAt indexPath: IndexPath)
```

와 같은 delegate 함수로 애니메이션 재생을 구현할 수 있다. 하지만 이렇게만 구현할 경우 AppStore보다 조금 입력 반응이 느린 느낌을 받는다. (딜레이가 있는 느낌?)

이유는 `UIScrollView`가 기본적으로 터치 입력을 딜레이하도록 하고있기 때문이다. `delaysContentTouches` 옵션이 이 기능에 대한 플래그인데 이 값이 기본적으로 `true`로 설정되어 있다. `UICollectionView`도 `UIScrollView`를 구체화한 클래스이기 때문에 이 값을 가지고 있고, 이 값을 `false`로 바꾸면 위 highlight 함수들이 좀 더 즉각적으로 반응하는 것을 볼 수 있다.

## 비교

> 커서에 빨간 원이 생기는 시점이 클릭이 시작된 시점이다.

### delaysContentTouches = false

![delay_off](/assets/iOS/scrollview_delay_off.mp4){:.aligncenter height="300"}

### delayesContentTouches = true
![delay_on](/assets/iOS/scrollview_delay_on.mp4){:.aligncenter height="300"}

이게 그냥 그자리에서 터치만 하는 경우에는 그렇게 큰 차이가 보이지 않지만 스크롤 입력, iOS에서 **pan**이라고 부르는 입력을 할 때 차이점이 도드라진다. 옵션을 껐을 때는 스크롤입력을 해도 cell이 줄어들었다가 커지는 애니메이션이 재생되지만 옵션이 켜져있으면 아예 애니메이션이 재생되지 않는것 처럼 보인다.