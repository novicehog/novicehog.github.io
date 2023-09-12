---
layout: single
title:  "1.swift 데이터 타입(자료형) "
categories: 
    - iOS
tag: [iOS,Swift]
published: true

author_profile: true # 옆에뜨는 프로파일

date: 2023-09-12
last_modified_at: 2023-09-12
---

<br/>

<details>
<summary>iOS 관련해서 유용한 사이트</summary>
<div markdown="1">       

앱스토어 현황
    https://www.pocketgamer.biz/metrics/app-store/
    
    마켓 매출 순위
    https://www.mobileindex.com/mi-chart/realtime-rank
    
    스위프트 문서
    https://bbiguduk.gitbook.io/swift
    
    스위프트 변경 사항 정리
    https://docs.swift.org/swift-book/documentation/the-swift-programming-language/revisionhistory/
    
    스위프트 스타일 추천
    https://github.com/swift-kr/swift-style-guide-raywenderlich/blob/master/ko_style_guide.md

</div>
</details> 
    
<br/>



# 데이터 타입(자료형)
{: .H1-font}

swift의 자료형은 다른 언어와 달리 대문자로 시작함.

자료형은 대표적으로 다음과 같은 것들이 있음

<span style="color:Skyblue"> **Bool, Character, Int, Float, Double, String, Void** </span>

## 선언
{: .H2-font}


아래와 같이 변수를 선언할 때 var또는 let 과 `:` 으로 자료형을 명시해서 선언을 함

```swift
var 변수_명_v : Int
let 변수_명_l : Int
```

초기 값이 있을 경우 컴파일러가 `타입 추론(type inference)` 를 할 수 있기 때문에
자료형 명시를 생략할 수 있음

```swift
var 변수_명_v = 10
let 변수_명_l = 10
```
<br/>

swift의 특이한 문법으로는
`=` 의 양 옆에 일정한 공백이 없으면 에러가 발생

```csharp
var x =10
print(x)
//error: '=' must have consistent whitespace on both sides var x =10
```
<br/>


또한 `;` 은 한 줄 안에서 구분 시켜주는 용도로 사용하며,
일반적인 언어처럼 끝마다 붙이는 것 대신 줄 바꿈을 하여도 됨

```csharp
var x = 10 ; var y = 20
print(x)
print(y)
// 10
// 20
```

### 변수 var 과 상수 let의 차이점
{: .H2-font}

var 외에도 let 으로 변수 선언을 할 수 있다

`var` 로 선언된 변수는 나중에 변경이 가능하지만

`let` 으로 선언된 변수는 변경이 불가능함

알아둬야 할 점으로 swift에서 `let` 은 `맨 처음으로 할당된 값을 상수로 저장`하므로

`초기 값을 설정하지 않고 나중에 값을 할당`해도 됨

비슷한 언어인 kotlin과의 차이는 

`let` 대신 `val` 로 상수를 선언됨

**swift**

```swift
var variableName: String = "Hello, Swift"
let constantName: Int = 10
```

**kotlin**

```kotlin
var variableName: String = "Hello, Kotlin"
val constantName: Int = 10
```

### print 함수
{: .H2-font}

![print](https://github.com/novicehog/NetworkProject/assets/131991619/a2fa612f-7221-4276-89e0-301fd62fd471)

swift의 print 함수는 몇 개의 인자든 받아서 출력할 수 있고

separator 를 통해 인자의 출력 간에 문자를 지정할 수 있으며

terminator를 통해 출력 끝에 붙을 문자를 지정할 수 있다.

```swift
print("Apple", "Banana", "Cherry", separator: " 중간 ", terminator: " 끝 ")
//Apple 중간 Banana 중간 Cherry 끝
```
<br/>

### 자료형 종류와 크기
{: .H2-font}

`type(of:변수_명)` 으로 크기를 알 수 있음

특이하게도 swift에서 Int형은 4바이트가 아닌 `8바이트`로 나오는 것을 알 수 있음

```swift
var x = 10
print(type(of:x))

let xs = MemoryLayout.size(ofValue: x)//8
let ds = MemoryLayout<Double>.size
let ss = MemoryLayout<String>.size
print(xs, ds, ss)
//Int
//8 8 16
```
<br/>

Int32 자료형을 통해 4바이트 Int를 사용할 수 있음

```swift
var x : Int = 10
print(x); print("x"); print("\(x)");print("값은 \(x)입니다.")
print("Int32 Min = \(Int32.min) Int32 Max = \(Int32.max)")
//Int32 Min = -2147483648 Int32 Max = 2147483647

print("Int Min = \(Int.min) Int32 Max = \(Int.max)")
//Int Min = -9223372036854775808 Int32 Max = 9223372036854775807
```
<br/>

### Float과 Double
{: .H2-font}

소수 점이 있는 변수 선언 시 `기본적으로 **Double**` 자료형이 사용됨

Float형은 소수점 6번째 까지 정확함

Double형은 소수점 15번째 까지 정확함

```swift
var f : Float  = 0.1234567890123456789
var d : Double = 0.1234567890123456789
print(f, d, separator: "\n")
//0.12345679
//0.12345678901234568
```
<br/>

### Bool
{: .H2-font}

다른 언어의 Bool과 같이

참 또는 거짓(1 또는 0) 조건을 처리할 데이터 타입

```swift
var 변수_명 : Bool
var 변수_명 : Bool = true
```
<br/>

## Character 과 String
{: .H2-font}

### Charactor
{: .H2-font}

문자, 숫자, 문장 부호, 심볼 같은 유니코드(Unicode) 문자 하나를 저장

```swift
var 변수_명: Character = "a"
```
<br/>

주의할 점은 초기 값은 작은 따옴표가 아니고 `큰 따옴표`로 감싸야 하며

자료형을 명시해 주지 않을 경우 `자동으로 String`형이 됨

### String
{: .H2-font}

단어나 문장을 구성하는 일련의 문자

변수의 값을 string으로 바꿔서 넣고 싶으면 ( 다른 언어에서는 `+ 연산자` 와 비슷)

`\(변수_명)`을 사용함

```swift
var message = "\(userName)의 나이는 \(age)입니다."
```
<br/>

## 튜플 Tuple
{: .H2-font}

여러 값을 하나의 개체에 일시적으로 묶는 것

튜플의 자료형

튜플은 여러 값을 가지고 있으므로 자료형도 여러 개를 가지고 있음

```swift
let myTuple = (179.8, 12.1, "Hi")
print(type(of:myTuple))
//(Double, Double, String)
```
<br/>

별명이 있는 튜플의 자료형

```swift
let myTuple = (height: 179.8, weight: 72.3, message: "Hi")
print(type(of:myTuple))
//(height: Double, weight: Double, message: String)
```
<br/>

별명이 있는 튜플이라도 번호를 통해 호출할 수 있음

```swift
let myTuple = (height: 179, length:  72.3, message: "Hi")
print(myTuple.message, myTuple.2)
//Hi Hi
```
<br/>

()처럼 괄호 안에 공백을 둬서 빈 튜플을 만들 수 있으며

Void가 대표적인 빈 튜플임

```csharp
typealias Void = () //아무 내용도 없는 튜플, typealias name = existing_type
```

