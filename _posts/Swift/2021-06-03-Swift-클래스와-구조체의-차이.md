---
layout: post
title: "Swift 클래스와 구조체의 차이"
date: 2021-06-03 10:11:00 +0900
categories: Swift
---

Swift에는 4가지 기본 인터페이스가 있다. 가장 추상적인 인터페이스인 **프로토콜**부터 OOP에서 가장 많이 사용하는 **클래스**, 그리고 **구조체**, **enum** 등이 있다. C++에서 구조체와 enum은 기본 자료형을 묶어서 구조체로 만들거나 숫자 리터럴을 키워드로 변경하기 위한 용도 등으로 사용했지만 Swift는 좀 더 많은 기능을 가지고 있다. 예를 들어 구조체와 enum 모두 class처럼 함수를 가질 수 있다.

이로써 구조체와 클래스의 경계가 모호해졌다고 볼 수 있지만 여전히 두 인터페이스는 다른 사용성을 가지고 있다.

## 차이점

### value type vs reference type

구조체는 **value type**이고 클래스는 **reference type**이다.

value type인 구조체는 값을 복사할 때 값이 통째로 복사되어 전달되고, reference type인 클래스는 값을 저장하고 있는 메모리의 주소가 복사된다. 즉, 구조체를 전달받은 함수에서 아무리 값을 바꿔도 외부에 있는 값은 전혀 영향을 안받지만 클래스를 전달받은 함수에서 값을 변경하면, 외부에서도 그 변경이 적용된다.

### 상속 받을 수 있는 인터페이스의 제한

구조체는 프로토콜만 상속 받을 수 있고, 클래스는 프로토콜 또는 다른 클래스를 상속 받을 수 있다. 

이유는 type의 차이를 생각해 보면 유추할 수 있는데, value type은 함수와 변수의 선언만 할 수 있는 프로토콜만 상속 받아서 언제나 같은 크기의 데이터를 정의할 수 있고, reference type은 프로토콜 뿐 아니라 다른 클래스를 상속 받아서 데이터의 크기가 커져도 변경된 크기에 따라 동적으로 메모리를 할당할 수 있다.

그렇다면 구조체가 클래스의 인스턴스를 가지고 있으면 어떻게 될까?

구조체는 클래스를 상속받을 수는 없지만 클래스를 인스턴스를 멤버로 가지고 있을 수는 있다. 클래스 인스턴스를 가지고 있는 구조체를 복사하면 구조체가 가지고 있던 클래스 인스턴스는 reference가 복사될까, value가 복사될까?

```swift
class Cls {
	var index: Int = 0

	// 1
	init() {
		print("init")
	}

	deinit {
		print("deinit")
	}

	func increase() {
		index += 1
	}
}

struct Stt {
	var number: Int
	// 2
	let cls: Cls

	func show() {
		print("\(number) \(cls.index)")
	}
}

// 3
var s1 = Stt(number: 5, cls: Cls())  // init
s1.show()  // 5 0
s1.cls.increase()
s1.show()  // 5 1
```

1. class `Cls`의 생성자와 소멸자가 실행될 때 알 수 있도록 `print()`를 추가한다.
2. struct `Stt`가 class `Cls`의 인스턴스를 가지고 있다.
3. struct `Stt`의 인스턴스를 생성하고 `Cls.index`를 증가시켰다.

```swift
// 1
var s2 = s1  // copy
s2.show()  // 5 1

// 2
s2.cls.increase()
s1.show()  // 5 2
```

1. `s2`에 `s1` struct를 복사한 후 출력해보면 그대로 `5 1`이 출력된다.
2. `s2.cls.index`를 증가시키고 `s1`을 출력해도 변화가 적용된 것을 볼 수 있다.

그렇다면 이 경우 `Cls`의 메모리는 언제 해제될까? 메모리 해제 시점을 알아보기 위해 미리 `deinit()`에 `print()`를 추가해 두었다.

```swift
// 1
var s1 = Stt(cls: Cls())
var s2 = Stt(cls: Cls())  // deinit
```

1. `s1`, `s2` 모두 새로운 `Stt`를 대입하면 `s2`에 대입했을 때 deinit이 출력된다. 즉, `s1`의 값이 `s2`로 복사되면서 `cls`에 대한 ARC가 증가했다는 뜻이다.

결국 구조체가 클래스 인스턴스를 가지고 있더라도 클래스 인스턴스는 reference type으로 복사되는 것을 알 수 있다.