---
layout: post
title:  "C++ 스마트 포인터 (smart pointer)"
date:   2021-05-27 21:44:00 +0900
categories: C++
---

## 개요

C++에 추가된 스마트 포인터들의 종류와 사용법에 대해 알아보자. C++11 부터 추가되기 시작했으며 버전이 올라가면서 점차 자리를 잡아가고있다. 그래서 종종 사용하고는 있었지만 한번쯤 정확히 짚고넘어가야될 필요성을 느꼈다.

특히 언리얼 엔진은 C++을 사용하기는 하지만 자체 라이브러리를 많이 사용하기 때문에 순수하게 C++만 사용하기에는 부적절한 부분이 있다. 예를들어 객체를 생성할 때 `new` 연산을 사용하지 않고 자체적인 객체생성 문법을 사용하며, C++에서 지원하지않는 어노테이션 등도 있다.

스마트포인터도 그 중 하나인데 엔진 라이브러리에 비슷한 기능을 하는 스마트포인터 템플릿이 있고, 그 템플릿을 잘 이해하려면 기본적으로 C++의 스마트포인터를 먼저 이해해야 한다는 생각이다.

C++에서 스마트포인터를 사용하려면 `<memory>` 헤더를 추가해야한다.

## class

아래 스마트포인터 코드에서 사용하는 클래스를 먼저 정의해보자. 간단하게 생성자와 소멸자가 있으며, 함수와 변수를 하나씩 추가하여 인스턴스에 접근했을 때 유효성을 확인할 수 있다.

```cpp
class Object {
public:
	Object()
	{
		std::cout << "Constructor" << std::endl;
	}

	~Object()
	{
		std::cout << "Desctructor" << std::endl;
	}

	void print()
	{
		std::cout << "print" << std::endl;
	}
	
	int getId()
	{
		return id;
	}

	void increaseId()
	{
		std::cout << "increase id" << std::endl;
		id++;
	}

private:
	int id;

};
```

## unique_ptr

하나의 포인터만이 해당 인스턴스를 가리킬 수 있도록 한다. 객체의 소멸권한을 소유권을 가진 한 포인터 변수에게만 주기 때문에 프로그래머가 원하는 곳에서 객체를 소멸시킬 수 있다.

### 생성

포인터를 유일하게 만들기 위해서 `std::make_unique<T>` 라는 특별한 생성문법을 사용한다.

```cpp
std::unique_ptr<Object> objectPtr = std::make_unique<Object>();
```

make_unique는 C++14부터 표준으로 추가되었다.

### 이동

`make_unique`로 생성된 인스턴스는 하나의 `unique_ptr`만이 가리킬 수 있으므로 일반적인 대입연산자로 다른 포인터로 옮길 수 없고, `std::move` 라는 함수를 사용해서 레퍼런스를 옮길 수 있다.

```cpp
std::unique_ptr<Object> objectPtrA = std::make_unique<Object>();
std::unique_ptr<Object> objectPtrB = std::move(objectPtrA);
```

이렇게 소유권을 이동시키면 인스턴스에 대한 접근은 `objectPtrB`만 인스턴스에 접근할 수 있고, `objectPtrA`는 소유권을 잃어서 더이상 인스턴스에 접근하거나 소멸시킬 수 없게된다.

```cpp
objectPtrA->print();
objectPtrB->print();

if (objectPtrA) 
{
	// 진입 불가
	std::cout << "objectPtrA is valid" << std::endl;
	objectPtrA->increaseId();
}

objectPtrB->increaseId();
```

코드에서 `objectPtrA->print()`는 잘 동작하는데 이는 인스턴스 변수에 직접 접근하지 않고 `cout`만 있는 함수이기 때문인 것 같다. 실제 변수에 접근하는 `increaseId()`는 호출하면 런타임 도중에 에러가 발생하기 때문에 포인터의 유효성을 검사해주어야 한다. 위 코드에서 `objectPtrA`는 참조를 잃고 `nullptr`이 되었기 때문에 `if`문을 통과하지 못한다.

### 소멸

포인터 변수는 인스터스를 소멸시킬 의무를 가진다. 물론 프로그램이 종료하면 모든 메모리가 정리되겠지만 동작 중에 객체의 소유권을 가진 포인터가 없어지면 메모리 누수가 발생하기 때문에 `unique_ptr`은 이러한 상황에서 자동으로 해당 객체를 소멸시켜준다.

```cpp
objectPtrB = std::make_unique<Object>();
// 1.
// Destructor
// 2.
// Constructor
```

1. `objectPtrB`에 새로운 인스턴스를 대입하면 기존에 가리키고있던 인스턴스의 소멸자가 자동으로 호출되어 메모리 누수를 막을 수 있다.
2. 새로운 `Object` 인스턴스의 생성자가 호출된다.

위처럼 암묵적인 소멸을 시킬 수도 있고, `reset()`을 호출하여 명시적으로 메모리를 해제시킬 수도 있다.

```cpp
objectPtrB.reset();
// Destructor
```

## shared_ptr

유일한 참조를 가지도록 강제하는 `unique_ptr`과 다르게 여러 곳에서 참조할 수 있는 포인터를 말한다. 여러 곳에서 참조할 수 있기 때문에 인스턴스를 가리키는 포인터 변수의 개수(reference)를 카운팅하여 메모리 누수가 발생하지 않도록 한다. C#이나 다른 언어의 가비지 컬렉터와 비슷한 개념이라고 생각하면 된다.

### 생성

`unique_ptr`와 비슷하게 `std::make_shared<T>`를 사용해서 생성한다.

```cpp
std::shared_ptr<Object> objectPtrA = std::make_shared<Object>();
```

### use_count

인스턴스를 포인터에 대입할 때 마다 `use_count`가 증가하며 해당 포인터 변수에 다른 값을 대입하면 즉, 참조 회수가 줄어들면 `use_count`를 감소시킨다. 이 값이 0이되면 인스턴스를 자동으로 제거하여 메모리누수를 방지한다.

```cpp
std::shared_ptr<Object> objectPtrA = std::make_shared<Object>();
std::cout << "objectPtrA's use count : " << objectPtrA.use_count() << std::endl;
// objectPtrA's use count : 1

std::shared_ptr<Object> objectPtrB = objectPtrA;
std::cout << "objectPtrA's use count : " << objectPtrA.use_count() << std::endl;
// objectPtrA's use count : 2

objectPtrB = nullptr;
std::cout << "objectPtrA's use count : " << objectPtrA.use_count() << std::endl;
// objectPtrA's use count : 1

objectPtrA = nullptr;
// Destructor
```

## weak_ptr

`shared_ptr`처럼 여러 곳에서 인스턴스를 참조할 수 있지만 `use_count`를 증가시키지 않는다.

`shared_ptr`은 유용하지만 모든 포인터가 `use_count`를 증가시킨다면 순환참조와 같은 문제가 발생하였을 때 `use_count`가 영원히 0이되지않아 메모리를 계속 잡아먹고있는 상황이 생길 수가 있다. 따라서 `use_count`를 굳이 증가시킬 필요가 없는 부분에서는 `weak_ptr`를 사용해서 순환참조를 막을 수 있다.

```cpp
std::shared_ptr<Object> objectPtrA = std::make_shared<Object>();
std::cout << "objectPtrA's use count : " << objectPtrA.use_count() << std::endl;
// objectPtrA's use count : 1

std::weak_ptr<Object> objectPtrB = objectPtrA;
std::cout << "objectPtrA's use count : " << objectPtrA.use_count() << std::endl;
// objectPtrA's use count : 1

objectPtrB.reset();
objectPtrA = nullptr;
// Destructor
```

위 코드에서 볼 수 있듯이 `weak_ptr`은 `reset()`을 호출해도 실제 인스턴스를 해제할 수 없기 때문에 안전하게 인스턴스를 다룰 수 있다.