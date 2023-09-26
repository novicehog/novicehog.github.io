---
layout: single
title:  "3.반복문, switch문, 함수"
categories: 
    - iOS
tag: [iOS,Swift]
published: true

author_profile: true # 옆에뜨는 프로파일

date: 2023-09-26
last_modified_at: 2023-09-26
---

# 반복문, switch, 함수

# 반복문

없어진 문법

c언어의 반복문과 같은 스타일은 사용할 수 없다.

```swift
for var i = 0; i < 10; i+=1 { // for i in 0..<10 로 수정해야 함
print(i)
} //error: C-style for statement has been removed in Swift 3
```

위와 같은 반복문을 swift에서 실행하려면 다음과 같이 수정 해야함.

```swift
for i in 0..<10{
    print(i)
}
```

반복문에서 제어 변수가 사용되지 않으면 경고가 발생한다.

```swift
for i in 0..<10{
    print("Sungho Choi")
}
//warning: immutable value 'i' was never used; consider replacing with '_' or removing it
```

이 경우 제어 변수 자리에  `_` 을 대신 사용해주면 된다.

```swift
for _ in 0..<5{
    print("Sungho Choi")
}
//0 Sungho Choi
//1 Sungho Choi
//2 Sungho Choi
//3 Sungho Choi
//4 Sungho Choi
```

문자열과 제어변수를 같이 쓰고싶다면 `\()` 를 사용하면 된다.

```swift
for i in 0..<5{
    print("\(i) Sungho Choi")
}
```

배열과 함께 쓰기

```swift
let ranges = ["범", "위", "지", "정"]
for word in ranges {
print(word , terminator: " ")
}
//범 위 지 정

```

범위 지정자 연산자와 함께 쓸 수 있다.

```swift
let ranges = ["범", "위", "지", "정"]
for word in ranges[...1] {
print(word , terminator: " ")
}
//범 위
```

딕셔너리와도 같이 쓸 수 있다.

```swift
let numberOfLegs = ["Spider": 8, "Ant": 6, "Dog": 4]
//dictionary는 key:value형식의 배열
for (animalName, legCount) in numberOfLegs {
print("\(animalName)s have \(legCount) legs")
}
// 딕셔너리는 순서가 없기 때문에 매번 다르게 출력됨
//Dogs have 4 legs
//Ants have 6 legs
//Spiders have 8 legs
```

while문은 다음과 같다.

조건문에 괄호를 안해도 된다.

```swift
var myCount = 0
while myCount < 500 {
myCount+=100
print(myCount)
}
//100
//200
//300
//400
//500
```

repeat while이라고 다른 언어에서 do while문과 같은 역할을 하는 문법도 있다.

```swift
var i = 4
repeat {
i=i-1
print(i) //출력 결과
} while (i > 0)
//3
//2
//1
//0
```

반복문 탈출

break와 continue를 사용한다

break문은 반복문 자체를 탈출한다.

swift에서는 if문의 실행문이 한 줄만 있어도 무조건 중괄호 `{}`와 함께 쓰여야 한다

```swift
for i in 1..<10 {
	if i > 5 break
	//중요!! 에러 수정 과제: error: expected '{' after 'if' condition
	print(i)
}

// 수정된 코드
for i in 1..<10 {
	if i > 5 {break}
	print(i)
}
```

continue는 반복문의 현재 실행문을 넘어간다.

```swift
for i in 1...6 {
	if i % 2 != 0 { // 짝수 외에는 넘어감
		continue
	}
	print(i)
}
//2
//4
//6
```

## if문

swift에서 조건문의 `,`는 &&(and)와 같다.

```swift
var a = 1
var b = 2
var c = 3
var d = 4
if a < b && d > c {
	print("&&")
}
if a < b, d > c { 
	print(",콤마")
}
if a < b, d < c { 
	print(",콤마") // 출력되지 않음
}

//&&
//,콤마
```

다중 if else

다중 if else문을 통해 다음과 같은 코드 작성이 가능하다.

```swift
let weight = 75.0
let height = 179.0
let bmi = weight / (height*height*0.0001) // kg/m*m
var body = ""
if bmi >= 40 {
	body = "3단계 비만"
} else if bmi >= 30 && bmi < 40 {
		body = "2단계 비만"
} else if bmi >= 25 && bmi < 30 {
		body = "1단계 비만"
} else if bmi >= 18.5 && bmi < 25 {
		body = "정상"
} else {
		body = "저체중"
}
print("BMI:\(bmi), 판정:\(body)")

//BMI:23.40750912892856, 판정:정상
```

### switch문

switch문은 각 case 문 마다 break가 자동적으로 있기에 

그 case문을 실행하고 자동으로 빠져나온다.

```swift
let someCharacter: Character = "z"
switch someCharacter {
	case "a":
		print("The first letter of the alphabet")
	case "z":
		print("The last letter of the alphabet")
	default:
		print("Some other character")
}
// Prints "The last letter of the alphabet"
```

주의 사항

swift문에서 case문에는 실행 문장이 무조건 있어야 한다.

```swift
let anotherCharacter: Character = "a"
switch anotherCharacter {
case "a": // Invalid, the case has an empty body
case "A":
	print("A글자")
default:
	print("A글자 아님")
}
```

switch case문 에서`,` 는 if문의 조건문과 다르게 `or` 의 의미를 가진다

```swift
var value = 3
var days : Int = 0
switch(value)
{
	case 1,3,5,7,8,10,12:
		print("31 일입니다")
	case 4,6,9,11:
		print("30 일입니다")
	case 2:
		print("28 or 29 일입니다")
	default:
		print("월을 잘못 입력하셨습니다")
}
```

위의 if문에서 만든 BMI측정기를 switch문을 통해 더 보기 좋게 할 수 있다.

```swift
let weight = 60.0
let height = 170.0
let bmi = weight / (height*height*0.0001) // kg/m*m
var body = ""

switch bmi{
    case 40... :
        body = "2단계 비만"
    case 30..<40 :
        body = "2단계 비만"
    case 25..<30:
        body = "1단계 비만"
    case 18.5..<25:  
        body = "정상"
    default :
        body = "저체중"
}
print("BMI:\(bmi), 판정:\(body)")
```

switch where절

부가적인 조건을 추가로 줄 때 where절을 사용한다.

과제

```swift
let ice = "민트초코"
let temperature = 30

switch ice {
case "바닐라" where temperature > 15:
    print("아니 다 녹았잖아.")
case  "바닐라" :
    print("역시 이맛이야.")
case  "민트초코" where temperature > 15:
    print("으 맛도 없는다 다 녹았네도") 
case  "민트초코":
    print("차가워도 맛이 없네") 
default :
    print("어떤 맛이든 괜찮아요.")
}
```

switch, catch, while, guard, for 등에서 사용 가능

아래는 for문의 예시다.

```swift
var numbers: [Int] = [1, 2, 3, 4, 5]
for num in numbers where num > 2 {
print(num)
}
//3
//4
//5
```

### fallthrough

switch문에 자동으로 들어있는 break가 불필요하다면 fallthrough 사용할 수 있다.

```swift
var value = 3
switch (value)
{
case 4:
print("4")
fallthrough
case 3:
print("3")
fallthrough
case 2:
print("2")
fallthrough
default:
print("1")
}

//3
//2
//1
```

# 함수

함수 정의부의 값을 `parameter(매개변수, 인자)`, 호출시의 값은 `argument(인수)`라고 부름

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/39b6f9eb-a9b6-4f51-8e35-fada792e5a1c/4c7c3782-2fe7-4e8c-9548-34681df59242/Untitled.png)

## 함수명

`함수명(외부매개변수명:외부매개변수명: ...)`

**`print(_:separator:terminator:)`**

## 메서드(method)

특정 클래스, 구조체, 열거형 내의 함수
함수를 `클래스 내에 선언`하면 `메서드`라 부름

함수 정의 방법

```swift
func <함수명> (<매개변수 이름>: <매개변수 타입>, <매개변수 이름>: <매개변수 타입>,... ) -> <반환값 타입> {
// 함수 코드
}
```

함수 예시

```swift
func sayHello() -> Void{ // 정의
    print("Hello")
}

sayHello() // 호출(call)
```

리턴값이 없으면 `→ Void`는 생략할 수 있다.

```swift
func sayHello() { // 정의
    print("Hello")
}
```

타입을 측정할 수 있다.

```swift
func sayHello() -> Void{
    print("Hello")
}
print(type(of:sayHello))
//() -> ()
```

주의 사항

매개변수가 있는 함수의 경우 호출할 때 지정 해줘야 한다.

```swift
func add(x:Int, y:Int)-> Int{ 
    return(x+y)
}
print(add(x:10,y:20)) // 인수와 매개변수 지정
// 30
```

매개변수와 반환 값이 있는 함수의 경우 타입을 측정하면 다음과 같다.

```swift
func add(x:Int, y:Int)-> Int{ 
    return(x+y)
}
print(type(of:add)) //(Int, Int) -> Int

func add2(x:Int, y:Int, z:Int)-> Int{ 
    return(x+y+z)
}
print(type(of:add2)) //(Int, Int, Int) -> Int
```

`argument label(외부)`과 `parameter name(내부)`을 각각 지정할 수 있다.

따로 지정해주지 않으면 내부가 외부를 겸한다.

```swift
func add(first x: Int, second y: Int) -> Int {
//외부 내부:자료형,외부 내부:자료형 -> 리턴형
return(x+y) //함수 정의할 때는 내부 매개변수명을 사용
} 
//add(x:10, y:20) //error: incorrect argument labels in call
add(first:10, second:20) // 30
```

`_` 를 사용해서 다른 언어처럼 호출할 수 있다.

```swift
func add(_ x: Int, _ y: Int) -> Int {
return(x+y)
} 
print(add(10,20))
```

혼합하여 사용하면 다음과 같다.

`이 방법을 제일 많이 쓴다.`

```swift
func add(_ x: Int, with y: Int) -> Int {
return(x+y)
}
print(add(10,with:20))
```

함수명을 출력해보면 다음과 같다.