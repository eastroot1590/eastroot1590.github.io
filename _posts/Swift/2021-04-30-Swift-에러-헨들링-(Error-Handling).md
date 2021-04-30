---
layout: post
title:  "Swift 에러 헨들링 (Error Handling)"
date:   2021-04-30 10:14:00 +0900
categories: Swift
---

이 글은 [Swift 5.4 Document Error Handling](https://docs.swift.org/swift-book/LanguageGuide/ErrorHandling.html){:target="_blank"} 문서를 번역한 글이며, Swift에서 어떻게 에러를 헨들링하는지 설명한다.

<br/>

**에러 헨들링(Error Handling)**은 프로그램의 에러 상황에 대응하고 복구하는 작업을 말한다. Swift는 런타임에 에러를 throw-catch 하거나, 전파, 조작, 복구할 수 있는 문법을 제공한다.

이 문법들이 항상 모든 에러에 대한 완벽한 헨들링를 보장하는 것은 아니다. 옵셔널은 값이 없다는 것을 표현하기 위해 만들어졌지만, 에러 헨들링에서 실패했을 때 어떤 원인에 의해 실패했는지 코드에서 알아내고 그에 따라 반응하도록 할 때 유용하게 사용할 수 있다.

저장되어 있는 파일을 열어서 데이터를 읽고 처리하는 예제를 떠올려보자. 코드가 실패할 수 있는 경우의 수는 수없이 많다. 저장소에 파일이 없거나, 파일은 있지만 읽기 권한이 제한되어 있거나, 파일의 내용이 호환되지 않는 포맷으로 작성되어 있는 등 작업이 실패할 수 있다. 이러한 실패를 감지할 수 있으면 적절한 대응 로직을 돌릴 수도 있고, 도저히 복구가 불가능할 경우 사용자에게 실패를 전달할 수 있게 해준다. (여전히 프로그램은 실행중인 채로)

> Swift의 에러 헨들링은 Cocoa와 Objective-C에서 사용하던 `NSError` 패턴과 유사하다. 더 자세한 정보는 [Handling Cocoa Errors in Swift](https://developer.apple.com/documentation/swift/cocoa_design_patterns/handling_cocoa_errors_in_swift){:target="_blank"} 문서를 참고.

## 에러 만들고 던지기 (Representing and Throwing Errors)

Swift에서 에러는 `Error` 프로토콜을 채택한 value type으로 통합 사용한다. `Error` 프로토콜은 비어있는 프로토콜이며 단지 에러 헨들링에 사용된다는 것을 명시하고, 다른 일반 enum 자료형과 구분한다.

Swift enum은 관련된 에러들을 그룹으로 묶어서 사용하기 좋으며, 연관된 에러에 대해 추가적인 정보를 제공할 수도 있다. 예를 들어, 게임 내에 존재하는 자판기가 발생시키는 에러를 다음과 같이 추상화할 수 있다.

```swift
enum VendingMachineError: Error {
    case invalidSelection
    case insufficientFunds(coinsNeeded: Int)
    case outOfStock
}
```

에러를 던진다는 것은 무언가 잘못되었고, 더이상 일반적인 흐름으로 프로그램을 실행시킬 수 없다는 것을 프로그램에게 전달하는 것이다. `throw` 키워드를 사용해서 에러를 던질 수 있다. 예를 들어, 다음 코드는 자판기가 정상적으로 동작하려면 코인이 5개 더 필요하다는 에러를 던진다.

```swift
throw VendingMachineError.insufficientFunds(coinsNeeded: 5)
```

<br/>

## 에러 헨들링 (Handling Errors)

던져진 에러를 받은 주변 코드에서는 에러를 바로잡거나 다른 방법으로 다시 시도하거나, 사용자에게 단지 에러 상황을 알리는 등 에러에 대해 헨들링해야 할 책임을 가지고 있다.

Swift에서 에러를 헨들링하는 방법은 총 4가지가 있다.

- 함수에서 발생한 에러를 함수를 호출한 부분으로 전파 (throwing function)
- `do`-`catch` 키워드를 사용해서 패턴 매칭
- 에러를 옵셔널로 변환해서 사용
- `assert`로 종료

각 방법은 순서대로 아래 섹션에서 자세하게 다룬다.

함수가 에러를 던졌다는 것은 프로그램의 흐름이 변경되었다는 것을 뜻하기 때문에 에러를 던질 수 있는 코드를 빠르게 찾아내는 것이 중요하다. 이 코드를 빠르게 찾아내기 위해서는 에러를 던질 수 있는 함수나 생성자를 호출할 때 `try` 키워드(`try?`나 `try!`로 조합해서 사용가능)를 추가한다. 나머지 두 키워드들은 아래 섹션에서 다룬다.

> Swift의 에러 헨들링은 `try`, `catch`, `throw` 키워드를 사용한다는 점에서 다른 언어의 예외 처리 문법과 닮아있지만, Objective-C를 포함한 다른 언어들의 예외 처리와 달리 Swift는 콜스택을 다시 되돌아가는 무거운 연산을 하지 않기 때문에 일반적인 `return`과 비슷한 성능을 낸다.

### Throwing Function으로 에러를 전파 (Propagating Errors Using Throwing Functions)

함수나 생성자가 에러를 던질 수 있다는 것을 표현하기 위해 함수 선언부의 파라미터 부분 다음에 `throws` 키워드를 추가한다. 이렇게 `throws`가 추가된 함수를 **throwing function**이라고 한다. 함수에 리턴 타입이 있을 때는 `throws` 키워드 다음에 `->`로 리턴 타입을 작성할 수 있다.

```swift
func canThrowErrors() throws -> String

func cannotThrowErrors() -> String
```

throwing function은 함수 내부에서 발생한 에러를 해당 함수를 호출한 외부에 전파한다.

> throwing function 만 에러를 전파할 수 있다. 일반 함수는 내부에서 에러가 발생하더라도 외부에 전파할 수 없고, 에러가 발생한 일반 함수 내부에서 헨들링해야 한다.

아래 에제에서 `VendingMachine` 클래스에 정의된 `vend(itemNamed:)` 함수는 파라미터를 통해 요청한 아이템이 재고가 없거나, 금액이 부족한 등 다양한 이유로 판매가 불가능할 때 적절한 `VendingMachineError`를 던진다.

```swift
struct Item {
    var price: Int
    var count: Int
}

class VendingMachine {
    var inventory = [
        "Candy Bar": Item(price: 12, count: 7),
        "Chips": Item(price: 10, count: 4),
        "Pretzels": Item(price: 7, count: 11)
    ]
    var coinsDeposited = 0

    func vend(itemNamed name: String) throws {
        guard let item = inventory[name] else {
            throw VendingMachineError.invalidSelection
        }

        guard item.count > 0 else {
            throw VendingMachineError.outOfStock
        }

        guard item.price <= coinsDeposited else {
            throw VendingMachineError.insufficientFunds(coinsNeeded: item.price - coinsDeposited)
        }

        coinsDeposited -= item.price

        var newItem = item
        newItem.count -= 1
        inventory[name] = newItem

        print("Dispensing \\(name)")
    }
}
```

`vend(itemNamed:)` 함수는 `guard` 키워드를 사용해서 함수 초기에 아이템을 판매할 수 없는 여러가지 조건들을 검사해서 적절한 에러를 던진다. `throw` 키워드는 즉시 실행되기 때문에 모든 조건을 만족했을 때만 아이템을 구매할 수 있다.

`vend(itemNamed:)`는 에러를 전파하기 때문에 이 함수를 호출하는 모든 곳에서는 `do`-`catch` 또는 `try?`, `try!`로 에러를 헨들링하거나 전파받은 에러를 다시 전파하는 등 반드시 에러를 처리하는 코드를 추가해야 한다. 예를 들어, 아래 예제 코드에 `buyFavoriteSnack(person: vendingMachine:)` 함수도 throwing function이며, `vend(itemName:)`가 던지는 모든 에러를 다시 자신을 호출한 코드로 전파한다.

```swift
let favoriteSnacks = [
    "Alice": "Chips",
    "Bob": "Licorice",
    "Eve": "Pretzels",
]
func buyFavoriteSnack(person: String, vendingMachine: VendingMachine) throws {
    let snackName = favoriteSnacks[person] ?? "Candy Bar"
    try vendingMachine.vend(itemNamed: snackName)
}
```

이 예제에서 `buyFavoriteSnack(person: vendingMachine:)` 함수는 `person`으로 전달받은 사람이 가장 좋아하는 과자를 찾고, `vend(itemNamed:)`를 호출해서 그 과자를 구매하는 함수다. `vend(itemNamed:)`가 에러를 던지기 때문에 호출할 때 `try` 키워드를 앞에 추가했다.

throwing initializer도 throwing function과 같은 방법으로 에러를 전파할 수 있다. 예를 들어, `PurchasedSnack` 구조체의 생성자는 생성 프로세스 중에 throwing function을 호출하고 전파된 에러를 다시 호출한 곳으로 전파한다.

```swift
struct PurchasedSnack {
    let name: String
    init(name: String, vendingMachine: VendingMachine) throws {
        try vendingMachine.vend(itemNamed: name)
        self.name = name
    }
}
```

### Do-Catch로 에러 헨들링 (Handling Errors Using Do-Catch)

코드 블록에서 발생하는 에러를 헨들링하기 위해 `do`-`catch` 키워드를 사용한다. `do` 블록 안에서 에러를 던지면 `catch` 블록에서 일치하는 에러를 캐치해서 처리할 수 있다.

일반적인 `do`-`catch` 사용법

```
do {
    try expression
    statements
} catch pattern 1 {
    statements
} catch pattern 2 where condition {
    statements
} catch pattern 3, pattern 4 where condition {
    statements
} catch {
    statements
}
```

`catch` 다음에 해당 블록이 어떤 에러를 헨들링할지 패턴을 지정할 수 있다. 아무 패턴도 없이 `catch`로만 블록을 만들면 해당 블록은 모든 에러를 `error`라는 이름의 상수로 매치시킨다. 패턴 매칭에 대한 더 자세한 정보는 [Patterns](https://docs.swift.org/swift-book/ReferenceManual/Patterns.html){:target="_blank"} 문서 참고.

예를 들어, 아래 코드는 `VendingMachineError`가 가진 세가지 에러를 모두 매치시킨다.

```swift
var vendingMachine = VendingMachine()
vendingMachine.coinsDeposited = 8
do {
    try buyFavoriteSnack(person: "Alice", vendingMachine: vendingMachine)
    print("Success! Yum.")
} catch VendingMachineError.invalidSelection {
    print("Invalid Selection.")
} catch VendingMachineError.outOfStock {
    print("Out of Stock.")
} catch VendingMachineError.insufficientFunds(let coinsNeeded) {
    print("Insufficient funds. Please insert an additional \\(coinsNeeded) coins.")
} catch {
    print("Unexpected error: \\(error).")
}
// Prints "Insufficient funds. Please insert an additional 2 coins."
```

위 예제에서 `buyFavoriteSnack(person: vendingMachine:)` 함수는 에러를 던질 수 있는 함수이기 때문에 `try` 키워드로 호출한다. 에러가 던져지면 즉시 매치되는 `catch` 블록이 실행된다. 아무 패턴에도 매치가 안되면 마지막에 있는 빈 `catch` 블록에 `error`로 매치된다. 에러가 던져지지 않으면 `do` 블록에 남아있는 코드들이 실행된다.

`catch` 블록은 `do`에서 발생할 수 있는 모든 에러를 매치시킬 필요는 없다. `catch` 블록에 아무것도 매치되지 않으면 그 바깥 범위로 에러가 전파된다. 단, 이렇게 에러가 밖으로 전파된다면 반드시 그 밖에서는 에러를 헨들링해야 한다. 예를 들어 throwing function이 아닌 일반 함수에서 `do`-`catch`를 사용한다면 반드시 `do`-`catch`에서 모든 에러를 처리해야 한다. 에러 처리를 계속 전파만 해서 최상위 레벨까지 에러를 처리하지 않으면 런타임 에러가 발생한다.

예를 들어, 위에 있는 예제는 `VendingMachineError`가 아닌 에러가 대신 매치될 수 있도록 작성할 수 있다.

```swift
func nourish(with item: String) throws {
    do {
        try vendingMachine.vend(itemNamed: item)
    } catch is VendingMachineError {
        print("Couldn't buy that from the vending machine.")
    }
}

do {
    try nourish(with: "Beet-Flavored Chips")
} catch {
    print("Unexpected non-vending-machine-related error: \\(error)")
}
// Prints "Couldn't buy that from the vending machine."
```

`nourish(with:)` 함수안에서 `vend(itemNamed:)`가 `VendingMachineError`에 포함된 에러를 던지면 `nourish(with:)`에서 헨들링하고, 그 밖의 에러가 던져지면 `nourish(with:)`를 호출한 쪽으로 에러를 전파하고 전파된 에러는 일반 `catch` 블록에 매치된다.

여러개의 에러 타입을 한 블록으로 헨들링하려면 쉼표로 구분해서 여러개의 pattern을 하나의 `catch` 블록으로 처리할 수 있다.

```swift
func eat(item: String) throws {
    do {
        try vendingMachine.vend(itemNamed: item)
    } catch VendingMachineError.invalidSelection, VendingMachineError.insufficientFunds, VendingMachineError.outOfStock {
        print("Invalid selection, out of stock, or not enough money.")
    }
}
```

`eat(item:)` 함수는 리스트에 있는 세개의 에러를 한 `catch` 블록에서 헨들링한다. 그 외 에러들은 바깥쪽 범위로 전파되며 그 결과 `eat(item:)`을 호출한 쪽으로 에러가 전파된다.

### 에러를 옵셔널로 변환 (Converting Errors to Optional Values)

`try?` 키워드로 에러를 옵셔널 값으로 변환해서 헨들링할 수 있다. `try?`가로 실행 중에 에러가 던져지면 그 실행 결과는 `nil`이 된다. 예를 들어, 아래 `x`와 `y`는 같은 값이고 그 과정 또한 같다.

```swift
func someThrowingFunction() throws -> Int {
    // ...
}

let x = try? someThrowingFunction()

let y: Int?
do {
    y = try someThrowingFunction()
} catch {
    y = nil
}
```

`someThrowingFunction()`이 에러를 던지면 `x`와 `y`는 `nil`이 되며, 에러를 던지지 않고 정상적으로 실행되면 `someThrowingFunction()`이 리턴하는 값이 된다. 여기에서는 `Int`를 리턴하기 때문에 `x`, `y`는 모두 `Int?` 자료형이 된다.

`try?`는 모든 에러를 같은 방법으로 헨들링하고 싶을 때 간결한 코드를 작성할 수 있게 도와준다. 예를 들어, 아래 코드는 여러 방법으로 데이터 패치를 시도하고 모두 실패하면 `nil`을 리턴한다.

```swift
func fetchData() -> Data? {
    if let data = try? fetchDataFromDisk() { return data }
    if let data = try? fetchDataFromServer() { return data }
    return nil
}
```

### 에러 전파 중지 (**Disabling Error Propagation)**

종종 throwing function이 에러를 던지지 않는다고 확신할 수 있을 때가 있다. 이런 경우에 `try!`를 써서 에러 전파를 막을 수 있다. 하지만 실제로 에러가 던져진다면 런타임 에러가 발생할 것이다.

예를 들어, 아래 코드에서 사용하는 `loadimage(atPath:)` 함수는 전달받은 경로에 있는 이미지를 로드하고 이미지 로드에 실패하면 에러를 던진다. 아래 경우에 이미지 파일이 애플리케이션 내부 리소스로 묶이기 때문에 이미지 로드에 실패할 일이 없고, 에러 전파를 중지하기에 적절하다.

```swift
let photo = try! loadImage(atPath: "./Resources/John Appleseed.jpg")
```

<br/>

## 뒷정리 (**Specifying Cleanup Actions)**

Swift는 현재 코드 블록을 벗어나기 직전에 마무리 작업을 수행하도록 하기 위해 `defer` 키워드를 제공한다. `defer` 블록은 에러를 던져서 빠져나갔는지, `return`이나 `break`로 빠져나갔는지를 따지지 않고 블록을 벗어날 때 무조건 실행되기 때문에 필요한 뒷정리 작업을 할 수 있다. 예를 들어, `defer` 블록에 열었던 파일을 닫는 코드를 추가하면 파일이 열린 채로 남아있는 것을 방지할 수 있다.

`defer`는 현재 블록을 벗어날 때 까지 실행을 미룬다. `defer` 블록에는 `return`이나 `break`처럼 코드의 흐름을 중단하는 키워드나 에러를 던지는 코드는 포함시키지 않는 것을 권장한다. `defer`는 일반적인 코드와 반대 순서로 실행된다. 즉, 코드상에 가장 먼저 선언된 `defer` 블록이 가장 나중에 실행되고 가장 나중에 선언된 블록이 가장 먼저 실행된다.

```swift
func processFile(filename: String) throws {
    if exists(filename) {
        let file = open(filename)
        defer {
            close(file)
        }
        while let line = try file.readline() {
            // Work with the file.
        }
        // close(file) is called here, at the end of the scope.
    }
}
```

위 예제에서 `defer`를 사용해서 에러에 의해 함수가 종료되더라도 `close(_:)`를 호출해서 파일을 닫을 수 있도록 했다.

> 에러를 던지지 않는 코드에서도 `defer`를 사용할 수 있다.

