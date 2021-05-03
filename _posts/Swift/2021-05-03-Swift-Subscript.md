---
layout: post
title: "Swift Subscript"
date: 2021-04-30 10:14:00 +0900
cateogries: Swift
---

인터넷에서 이런 저런 코드를 보다가 `self[someIndex]`라는 이상한 코드를 발견했다. `self`는 클래스 인스턴스를 말하는데 거기에 배열처럼 접근할 수 있나? 구글링을 해보니까 Swift의 `subscript`라는 문법이라고 한다. 그래서 [Swift 5.4 Document Subscript](https://docs.swift.org/swift-book/LanguageGuide/Subscripts.html) 문서를 번역하면서 공부해보기로 했다.

<br/>

`class`, `struct`, `enum`등 사용자 정의 타입은 맴버에 빠르게 접근할 수 있는 **subscript**라는 키워드를 제공한다. subscript를 사용하면 `Array` 인스턴스의 각 값을 `someArray[index]`로 접근하고 `Dictionary` 인스턴스의 각 value를 `someDictionary[key]`로 얻어오는 것처럼 따로 함수를 만들지 않고도 인덱스 값을 통해 특정 맴버로 접근해서 값을 얻어오거나 수정할 수 있다. 

> 이미 배열과 딕셔너리를 사용하면서 subscript를 알게모르게 쓰고있었다.

한 타입에 여러개의 subscript를 정의할 수도 있는데 이 subscript들 중에 전달되는 인덱스의 타입에 따라 적절한 subscript가 선택되어 실행된다. subscript는 하나의 인덱스만 한정하지 않고 다양한 타입에 접근하도록 여러개의 인덱스를 받을 수도 있다. 함수와 유사한 점이 굉장히 많다.

## Subscript 문법 (**Subscript Syntax)**

subscript를 선언할 때 `subscript` 키워드를 사용한다. 하나 이상의 파라미터를 받고 하나의 값을 리턴하는 함수와 유사한 모습이지만, 내부는 read-write 또는 read-only computed property와 비슷하게 선언한다. 선언한 인스턴스 이름뒤에 `[]`로 원하는 값 또는 값들을 전달해서 subscript를 호출할 수 있다. 인스턴스의 메소드를 호출하는 문법과 비슷하지만 호출할 때 사용하는 함수 이름이 없고, 사용하는 괄호의 종류가 다르다.

```swift
subscript(index: Int) -> Int {
    get {
        // Return an appropriate subscript value here.
    }
    set(newValue) {
        // Perform a suitable setting action here.
    }
}
```

setter에서 사용하는 `newValue`의 타입은 subscript의 리턴 타입과 같다. computed property를 만들 때와 동일하게 `newValue` 대신 원하는 이름을 사용할 수도 있고, 이름을 생략하면 자동으로 `newValue`라는 이름으로 매치된다.

`set` 블록을 생략하면 read-only subscript가 되어 값을 지정할 수 없게 된다. 이렇게 만든 read-only subscript에는 `get` 블록만 남았기 때문에 `get`을 생략해도 getter로 추론된다.

```swift
subscript(index: Int) -> Int {
    // Return an appropriate subscript value here.
}
```

Here’s an example of a read-only subscript implementation, which defines a `TimesTable` structure to represent an *n*-times-table of integers:

아래 예제 코드의 `TimesTable` 구조체는 read-only subscript를 가지고 있다. 

```swift
struct TimesTable {
    let multiplier: Int
    subscript(index: Int) -> Int {
        return multiplier * index
    }
}
let threeTimesTable = TimesTable(multiplier: 3)
print("six times three is \(threeTimesTable[6])")
// Prints "six times three is 18"
```

`multiplier`프로퍼티를 초기화하는 생성자에 3을 전달해서 3의 배수를 생성하는 `TimesTable`의 인스턴스를 생성했다. 

코드에 나와있는 것 처럼 `threeTimesTable[6]`으로 `threeTimesTable`의 subscript를 호출할 수 있다. 이 호출은 `3`의 배수 테이블의 `6`번째 값, `18`을 리턴한다.

> 위 예제는 고정된 수식을 기반으로 된 테이블을 사용했다. `threeTimesTable[someIndex]`에 새로운 값을 대입할 수 있게되면 테이블의 수식에 맞지 않기 때문에 read-only subscript를 사용했다.

## Subscript의 사용 (**Subscript Usage)**

**subscript**의 정확한 의미는 subject가 사용되는 문맥에 따라 달라진다. 일반적으로 subscript는 내부 리스트나 시퀀스에 빠르게 접근하기 위해 사용하지만, 클래스나 구조체의 기능에 맞게 재정의해서 사용할 수도 있다.

예를 들어, Swift의 `Dictionary`에 있는 subscript는 `[]` 안에 전달된 키를 통해 `Dictionary`의 값에 접근해서 딕셔너리 value를 저장하거나 얻어올 수 있다.

```swift
var numberOfLegs = ["spider": 8, "ant": 6, "cat": 4]
numberOfLegs["bird"] = 2
```

위 예제에서 `numberOfLegs`라는 `Dictionary` 변수를 선언하고 임의로 3개의 `[String: Int]` 키-값 쌍을 초기화한 후, subscript를 통해 `"bird"` 키에 `Int`값 `2`를 추가했다.

`Dictionary`의 subscript에 대해 더 자세한 정보는 [Accessing and Modifying a Dictionary](https://docs.swift.org/swift-book/LanguageGuide/CollectionTypes.html#ID116) 문서 참고.

> 위 예제의 `numberOfLegs` dictionary는 `Int?`를 리턴한다. Swift의 `Dictionary`는 모든 키에 값이 있는건 아니라는 것을 명시하고, `nil`을 대입해서 해당 키의 값을 제거하기 위해 optional subscript를 사용한다.

## Subscript 옵션 (**Subscript Options)**

subscript는 여러개의 입력 파라미터를 타입 제한 없이 받을 수 있고, 어떤 타입도 리턴할 수 있다.

함수와 마찬가지로 가변 개수 파라미터를 사용할 수 있고, 파라미터에 기본값을 대입해서 사용할 수도 있지만 함수와 달리 subscript는 `inout` 파라미터는 사용할 수 없다. 가변 개수 파라미터와 파라미터 기본값에 대한 자세한 내용은 [Variadic Parameters](https://docs.swift.org/swift-book/LanguageGuide/Functions.html#ID171)와 [Default Parameter Values](https://docs.swift.org/swift-book/LanguageGuide/Functions.html#ID169) 문서 참조. 

클래스와 구조체는 필요한 만큼 많은 subscript를 사용할 수 있다. 좋은 subscript는 `[]` 안에 포함된 값을 기반으로 의미하는 바를 유추할 수 있도록 만들어야 한다. 이렇게 여러개의 subscript를 정의하는 것을 subscript 오버로딩 이라고 한다.

가장 일반적인 subscript는 하나의 파라미터만 받는 형태이긴 하지만 필요하다면 여러개의 파라미터를 받을 수 있게 만들 수도 있다. 아래 예제는 두개의 `Double` 값으로 2차원을 표현한 `Matrix` 구조체를 정의하고, `Matrix`의 subscript는 두개의 정수 파라미터를 받는다.

```swift
struct Matrix {
    let rows: Int, columns: Int
    var grid: [Double]
    init(rows: Int, columns: Int) {
        self.rows = rows
        self.columns = columns
        grid = Array(repeating: 0.0, count: rows * columns)
    }
    func indexIsValid(row: Int, column: Int) -> Bool {
        return row >= 0 && row < rows && column >= 0 && column < columns
    }
    subscript(row: Int, column: Int) -> Double {
        get {
            assert(indexIsValid(row: row, column: column), "Index out of range")
            return grid[(row * columns) + column]
        }
        set {
            assert(indexIsValid(row: row, column: column), "Index out of range")
            grid[(row * columns) + column] = newValue
        }
    }
}
```

`Matrix`는 `rows`, `colums`라는 이름을 가진 두개의 파라미터를 받는 생성자를 제공하고, 생성자에서는 전달받은 `rows * columns`만큼 `Double`을 저장할 수 있는 배열을 초기화한다. 배열의 각 요소는 `0.0`으로 초기화한다. 특정한 크기만큼 특정한 값으로 배열을 초기화 하기 위해서 `Array`의 생성자를 사용했다. 배열 생성자에 대한 더 자세한 정보는 [Creating an Array with a Default Value](https://docs.swift.org/swift-book/LanguageGuide/CollectionTypes.html#ID501) 문서 참고

이제 원하는 크기 만큼의 row, column 값을 입력해서 `Matrix`를 생성할 수 있다.

```swift
var matrix = Matrix(rows: 2, columns: 2)
```

위 예제에서 2*2 크기의 `Matrix`를 생성했기 때문에 `grid` 배열은 이 matrix를 왼쪽 위에서부터 순서대로 단순화한 아래와 같은 모습이 된다.

![https://docs.swift.org/swift-book/_images/subscriptMatrix01_2x.png](https://docs.swift.org/swift-book/_images/subscriptMatrix01_2x.png){:.aligncenter}

matrix의 각 요소는 `,`로 구분해서 subscript로 전달된 row, column 값으로 접근할 수 있다.

```swift
matrix[0, 1] = 1.5
matrix[1, 0] = 3.2
```

위 코드는 `row`가 `0`, `column`이 `1`인 위치에 `1.5`를 대입하고 `row`가 `1`, `column`이 `0`인 위치에 `3.2`를 대입하여 아래와 같은 모습이 되도록 만든다.

![https://docs.swift.org/swift-book/_images/subscriptMatrix02_2x.png](https://docs.swift.org/swift-book/_images/subscriptMatrix02_2x.png){:.aligncenter}

`Matrix` subscript의 getter와 setter는 둘 다 전달받은 `row`, `column`이 유효한지 검사하는 assert를 가지고 있다. assert를 위해 `row`, `column`이 matrix 범위 안에 있는 값인지 검사하는 `indexIsValid(row: column:)`이라는 함수를 정의했다. 

```swift
func indexIsValid(row: Int, column: Int) -> Bool {
    return row >= 0 && row < rows && column >= 0 && column < columns
}
```

사용자가 범위 밖에 있는 값에 접근할 때 assert가 동작한다.

```swift
let someValue = matrix[2, 2]
// This triggers an assert, because [2, 2] is outside of the matrix bounds.
```

## **Type Subscripts**

앞선 예제에서 썼던 subscript는 특정 타입의 인스턴스를 통해 호출해서 썼지만 타입 메소드처럼 타입 이름을 통해 호출하는 subscript를 만들 수 있다. 이러한 subscript를 **type subscript**라고 부르고 `static` 키워드를 추가해서 type subscript를 만들 수 있다. 클래스의 type subscript는 `static` 대신 `class` 키워드를 사용한다.

```swift
enum Planet: Int {
    case mercury = 1, venus, earth, mars, jupiter, saturn, uranus, neptune
    static subscript(n: Int) -> Planet {
        return Planet(rawValue: n)!
    }
}
let mars = Planet[4]
print(mars)
```