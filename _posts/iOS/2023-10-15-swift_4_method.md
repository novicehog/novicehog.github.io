---
layout: single
title:  "4.함수 ,1급 시민, 클로저 기초"
categories: 
    - iOS
tag: [iOS,Swift]
published: true

author_profile: true # 옆에뜨는 프로파일

date: 2023-10-15
last_modified_at: 2023-10-15
---


## 함수

![1](https://github.com/novicehog/comments/assets/131991619/df422565-256b-41b7-8d8e-e49adc3de157)

함수의 자료형은

`(매개변수의_자료형, 매개변수의_자료형, …) → 리턴형`

로 이루어짐

![2](https://github.com/novicehog/comments/assets/131991619/c32b9903-80e5-4430-8b39-900c936a3c87)

함수는 만들 때 내부 매개변수와 외부 매개변수 두 가지를 정해줄 수 있다.

함수 이름 <br>
함수명(`외부 매개변수:` `외부 매개변수:`) 


### 디폴트 매개변수

디폴트 매개 변수을 가지는 것이 가능하다.

```swift
func sayHello(count: Int, name: String = "길동") -> String {
return ("\(name), 너의 번호는 \(count)")
}
var message = sayHello(count:100)
print(message) //길동, 너의 번호는 100
```

다음 예와 같이  `=` 에 기본적으로 값을 넣어줄 수 있으며,

함수를 호출할 때 굳이 매개변수 값을 넣지 않으면 자동으로 저 값을 사용하게된다.

### 여러개의 값을 반환하기

Tuple을 리턴 값으로 하여 여러 개의 값을 동시에 반환할 수 있다.

Tuble을 리턴 값으로 사용하려면 `반환_값의 자료형이 Tuple임을 명시`해주고  `Tuple로 반환`해주면 된다.

```swift
func converter(length: Float) -> 
	(yards: Float, centimeters: Float, meters: Float, mylength: Float) // 반환값 Tuple로 명시
{
   let yards = length * 0.0277778
   let centimeters = length * 2.54
   let meters = length * 0.0254
   let mylength = length * 50
   return (yards, centimeters, meters, mylength)
}
var lengthTuple = converter(length:10)
print(lengthTuple) //(yards: 0.277778, centimeters: 25.4, meters: 0.254)
print(lengthTuple.yards) // 0.277778
print(lengthTuple.centimeters) //25.4
print(lengthTuple.meters) //0.254
print(lengthTuple.mylength) // 500.0
```

<br>

다음과 같은 예제를 만들어 볼 수 있다

```swift
import Foundatio // String을 사용하기 위해 필요

func sss(x : Int, y : Int) -> (sum : Int, sub : Int, mul : Int, div : Double,rem : Int)
{
let sum = x+y
let sub = x-y
let mul = x*y
let div = Double(x)/Double(y) //같은 자료형만 연산 가능
let rem = x%y 
return (sum, sub, mul, div, rem)
}
var result = sss(x:10,y:3)
print(result.sum) // 13
print(result.sub) // 7
print(result.mul) // 30
print(String(format: "%.3f", result.div)) // 3.333
print(result.rem) // 1
```
<br>

### 가변 매개변수

매개변수 자료형 옆에 `…` 을 붙이면 몇 개든지 받을 수 있다.

이를 통해 다음과 같은 예제를 만들 수 있다.

```swift
func add (numbers : Int...)
{
   var sum = 0
   for num in numbers
   {
      sum += num
   }
   print(sum)
}

add(numbers : 1,3,5,6,7,8,9,4,3,5,1,3,5,7,7) // 모든 숫자가 더해서 출력됨
```
<br>

### print함수 이해하기

print 함수는 다음과 같다.

```swift
func print(
    _ items: Any...,
    separator: String = " ",
    terminator: String = "\n"
)
```

함수를 배우며 이해한 내용을 토대로 print함수를 보면

`_` 는 외부 매개변수를 생략하는 기호이고,

`...` 은 가변 매개변수로 매개변수를 몇 개든 받을 수 있다.

String 옆에 `=` 는 디폴트 매개변수로 매개변수를 지정해주지 자동으로 저 값을 사용한다.

### InOut (Call By Reference)

InOut키워드를 통해 함수의 매개변수로 값을 주소로 넘겨서 변화된 값을 변수가 가질 수 있다.

InOut 키워드를 사용하려면 매개변수에 `inout` 키워드를 넣어주고

함수를 호출할 때 인자값에 `&`를 앞에 붙여 사용해야한다.

예제는 다음과 같다.

```swift
var myValue = 10
func doubleValue (value: inout Int) -> Int {
value += value
return(value)
}
print(myValue) // 10
print(doubleValue(value : &myValue)) //20 여기서 myValue값이 변화됨
print(myValue) // 20
```

<br>

## 1급 객체 또는 1급 시민

![3](https://github.com/novicehog/comments/assets/131991619/58504d55-5b32-47ad-8629-722f2f3c9e0c)

다음 코드를 통해 swift 함수가 1급 객체임을 확인할 수 있다.

```swift
func up(num: Int) -> Int {
return num + 1
}
func down(num: Int) -> Int {
return num - 1
}

// ----------------- 대입이 가능 ------------------------
let toUp = up
print(up(num:10))           // 11
//print(toUp(num:10))       // error: extraneous argument label 'num:' in call
print(toUp(10))             // 11  대입 후에는 argument label을 쓰지 않음

let toDown = down
print(down(num:10))         // 9
//print(toDown(num:10))     // error: extraneous argument label 'num:' in call
print(toDown(10))           // 9

// 함수의 자료형은 (자료형 나열) -> 리턴형
print(type(of:up))          //(Int) -> Int
print(type(of:toUp))        //(Int) -> Int

// ----------------- 매개변수로 전달 가능 ----------------------

func upDown(Fun: (Int) -> Int, value: Int) { // 함수를 매개변수로 전달 
let result = Fun(value)
print("결과 = \(result)")
}
upDown(Fun:toUp, value: 10)     //toUp(10)      결과 = 11
upDown(Fun:toDown, value: 10)   //toDown(10)    결과 = 9

// ---------------- 리턴 값으로 사용할 수 있다. ----------------------

//매개변수 자료형 bool 리턴형 (Int) -> Int의 함수
func decideFun(x: Bool) -> (Int) -> Int { 
//매개변수형 리턴형이 함수형
if x {
return toUp
} else {
return toDown
}
}

var r = decideFun(x:true)   // r = toUp
print(type(of:r))           //(Int) -> Int
print(r(10))                // toUp(10)         // 11
r = decideFun(x:false)      // r = toDown
print(type(of:r))           //(Int) -> Int
print(r(10))                // toDown(10)       // 9
```

<br>

## guard문

![4](https://github.com/novicehog/comments/assets/131991619/9590fe48-eca2-4a92-b899-992160cd8a72)

![5](https://github.com/novicehog/comments/assets/131991619/ef15ce8d-edaf-4d0e-b75f-4f63e6136be7)

![6](https://github.com/novicehog/comments/assets/131991619/ed15fd8a-5022-46f3-9bd4-0baf6dae8d81)
