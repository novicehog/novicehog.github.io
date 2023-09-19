---
layout: single
title:  "2.swift 연산자와 옵셔널 "
categories: 
    - iOS
tag: [iOS,Swift]
published: true

author_profile: true # 옆에뜨는 프로파일

date: 2023-09-19
last_modified_at: 2023-09-19
---

# 기본 연산자

swift의 연산자와 다른 언어의 연산자 사이에 차이가 있는 부분이 있다.

<details>
<summary>관련 사이트</summary>
<div markdown="1">    
https://bbiguduk.gitbook.io/swift/language-guide-1/basic-operators

https://developer.apple.com/documentation/swift/operator-declarations   
</div>
</details> 

## 연산자의 우선순위(**precedence)**와 결합성(**associativity**)

https://www.programiz.com/swift-programming/operator-precedence-associativity

우선순위 : 우선순위가 높은 연산자가 먼저 계산됨

결합성 : 우선순위가 같은 연산자는 왼쪽에서 오른쪽 방향으로 계산

*대입 연산자의 경우만 왼쪽에서 오른쪽

## 증감 연산자

++ --와 같은 증감 연산자는 swift에서 없어졌다.

증감연산자 대신

+= , -=를 사용해야함

```swift
var x = 0
x += 2
// 또는
var y = 0
y = y + 2 
```

## 범위 연산자

### 닫힌 범위 연산자(closed range operator)
***
`x…y`  x부터 y까지 
 
```swift
 let names = ["A", "B", "C", "D", "E", "F", "G"]
for name in names[2...] 
{ 
    print(name, terminator:  " ")
} 
// C D E F G

for name in names[2...3] 
{ 
    print(name, terminator:  " ")
} 
// C D

for name in names[...3] 
{ 
    print(name, terminator:  " ")
} 
// A B C D

```
<br>

### 반 열린 범위 연산자(half-open range operator)

`x..<y` x부터 y바로 전 까지 (y제외)

```swift
let names = ["A", "B", "C", "D", "E", "F", "G"]
for name in names[..<2] // 2번째 바로 전까지
{ 
    print(name, terminator:  " ")
} 
// A B
```




# 클래스, 객체, 인스턴스

Class : 인스턴스를 찍어내는 틀

Object :  하나의 사물을 의미

Instance : 클래스를 기반으로 생성된 객체

![객체](https://github.com/novicehog/comments/assets/131991619/b97999fb-57fb-43ae-81fc-d5d655e084be)

<br>

# 형 변환

swift는 `type safety` 한 언어라서 다른 자료형 끼리의 연산이 엄격하다.

그래서 swift에서는 다른 자료형끼리는 연산이나 대입이 불가능하다.

```swift
var x = 15 // Int
var y = 4.0 // Double
print(x/y)
// error: binary operator '/' cannot be applied to operands of type 'Int' and 'Double'

x = y
// error: cannot assign value of type 'Double' to type 'Int'
```

대입이나 연산을 하려면 형 변한 과정이 필요하다

```swift
var x : Double = 15.0
var y : Int = Int(x)    // 15(Int)로 바뀜
print(Int(x) + y) //30 출력
```

# 옵셔널 Optional

## 옵셔널

swift는 값이 없음을 나타내는 nil을 담을 수 있는 옵셔널 타입이 있다.

옵셔널 타입은 변수 또는 상수에 아무런 값이 할당되지 않는 상황을 안전하게 처리하기 위한
방법 제공한다.

```swift
var y : Int? = 10 //y? 

var z : Int! //z?
```



## String의 Optinal
String 형을 숫자 자료형으로 바꾸면 Optional로 나온다

이는 String이 전부 숫자라면 변환이 가능하겠지만 문자일 경우는 변환이

불가능하기 때문

```swift
print(Int("123"))
//Optional(123)

print(Int("hi"))
```

```swift
var x : Int?
var y : Int = 0

x = 10      // 주석처리하면 x가 nil이 되어 에러발생
print(x)    // optional(10)
print(x!)   // 강제로 언래핑하여 10
print(y)

x = x! + 2
y = x!
```

### Optinal로 된 값을 안전하게 언래핑하기

UnWrapping은 옵셔널 타입 안에 감싸진 값을 벗겨내어 사용하는 것인데, <br>
아무런 값이 없는 nil일 경우 UnWrapping하게되면 앱이 다운되어버린다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/39b6f9eb-a9b6-4f51-8e35-fada792e5a1c/ad9efcd0-77a4-453b-b573-6b1f9f88a28c/Untitled.png)

```swift
var x : Int?
x = 10

if x != nil {print(x!)}
else {print("nil")}

var x1 : Int?
if x1 != nil {print(x1!)}
else {print("nil")}
```

직접 코드로 검사하는 것이 아니라 swift에서 제공되는 기능인

 optional binding 사용하는 방법도 있다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/39b6f9eb-a9b6-4f51-8e35-fada792e5a1c/ab589344-3a05-4c7a-ada3-134a6a052f4f/Untitled.png)

optional binding에는 세 가지 형태가 있다.

```swift
if let 새로운_변수_명 = 옵셔널_변수{}
if let XX = X{}
```

```swift
if let 옵셔널_변수 = 옵셔널_변수{}
if let x = X{}
```

```swift
if let 옵셔널_변수{}
if let X{}
```

여러가지 옵셔널_변수를 동시에 언래핑 할 수 있다.

!통해 옵셔널 변수를 선언할 수도 있다.

이 경우 자동으로 컴파일러가 자동으로 판단하여 언래핑을 한다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/39b6f9eb-a9b6-4f51-8e35-fada792e5a1c/f61ad134-85df-4015-92d4-d1303a3748b5/Untitled.png)

https://docs.swift.org/swift-book/documentation/the-swift-programming-language/thebasics/

```swift
let x : Int? = 1
let y : Int = x!
let z = x
print(x,y,z) //Optional(1) 1 Optional(1)
print(type(of:x),type(of:y),type(of:z))
//Optional<Int> Int Optional<Int>
let a : Int! = 1 //Implicitly Unwrapped Optional
let b : Int = a //Optional로 사용되지 않으면 자동으로 unwrap함
let c : Int = a!
let d = a //Optional로 사용될 수 있으므로 Optional형임
let e = a + 1
print(a,b,c,d,e) //Optional(1) 1 1 Optional(1) 2
print(type(of:a),type(of:b),type(of:c),type(of:d), type(of:e))

```

### Nil-Coalescing Operator (Nil합병연산자) ??

언래핑을 할 수 있는 연산자가 또한 존재한다

```swift
let defaultAge = 5
var age : Int?
age = 3
print(age) //Optional(3)
var myAge = age ?? defaultAge
//age가 nil이 아니므로 언래핑된 값이 나옴
print(myAge) //3
```

```swift
let defaultColor = "black"
var userDefinedColor: String? // defaults to nil
var myColor = userDefinedColor ?? defaultColor
//nil이므로 defaultColor인 black으로 할당됨
print(myColor) //black
userDefinedColor = "red"
myColor = userDefinedColor ?? defaultColor
//nil이 아니므로 언래핑된 red가 할당됨
print(myColor) //red, 주의 optional(red)가 아님
```

**Swift에서 옵셔널 값을 사용하려면 먼저 옵셔널을 해제(unwrapping)해야 합니다. 다음은 Swift에서 옵셔널을 해제하는 주요 방법들입니다:**

**강제 언래핑 (Forced Unwrapping): 옵셔널 값 뒤에 느낌표(!)를 붙여 값을 강제로 추출합니다. 하지만 이 방법은 해당 값이 nil인 경우 런타임 에러를 발생시키므로, 실제 값이 반드시 존재함을 확신할 때만 사용해야 합니다.**

```swift
let optionalInt: Int? = 5
let forcedUnwrappedInt: Int = optionalInt!// 5
```

**옵셔널 바인딩 (Optional Binding): `if let` 구문이나 `guard let` 구문을 사용하여 안전하게 옵셔널 값을 추출합니다. 이 방법은 nil인 경우 안전하게 처리할 수 있습니다.**

```swift
let optionalString: String? = "Hello, world"

if let unwrappedString = optionalString {
    print(unwrappedString)// "Hello, world"
} else {
    print("The value is nil.")
}

guard let unwrappedString = optionalString else {
    return// 혹은 다른 처리
}
print(unwrappedString)// "Hello, world"
```

**Nil 병합 연산자 (Nil-Coalescing Operator): `??` 연산자를 사용하여 옵셔널 값이 nil일 경우 기본값을 제공할 수 있습니다.**

```swift
let optionalDouble: Double? = nil
let doubleWithDefaultValue = optionalDouble ?? 0.0// 0.0
```

**옵셔널 체이닝 (Optional Chaining) : `?.` 연산자를 통해 옵셔널의 내부 속성에 접근하거나 메소드를 호출하는데, 만약 그 과정에서 어떤 단계가든 nil값을 만나면 즉시 멈추고 전체 결과값으로 nil을 반환합니다.**

```swift
class MyClass {
    var myProperty: String?
}

let myInstance: MyClass? = MyClass()
myInstance?.myProperty?.count// 결과는 Optional(Optional(nil))입니다.
```

**위의 네 가지 방법인 강제 언래핑, 옵셔널 바인딩, Nil 병합 연산자 그리고 옵셔널 체이닝은 Swift에서 가장 많이 사용되는 방법들입니다. 이 중 어떤 방법을 사용할지는 상황과 개발자의 판단에 따라 달라집니다.**