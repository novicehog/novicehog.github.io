---
layout: single
title: "클로저, 클래스"
categories: 
    - iOS
tag: [iOS,Swift]
published: true

author_profile: true # 옆에뜨는 프로파일

date: 2023-10-23
last_modified_at: 2023-10-23
---

# 클로저

**Swift 언어에서 클로저는 이름 없이 작성되거나 변수나 상수에 할당될 수 있는 독립적인 코드 블럭**

 클로저나 함수를 대입하교 호출할 떄는 `argument labels`이 필요없음

```swift
// 함수
func add(x: Int, y: Int) -> Int {
return(x+y)
}
print(add(x:10, y:20))

// 클로저를 변수 add1에 대입
let add1 = { (x: Int, y: Int) -> Int in
return(x+y)
}
print(add1(x:10, y:20)) //error: extraneous argument labels 'x:y:' in call
print(add1(10, 20))     // 클로저나 함수를 대입하교 호출할 때는 argument labels이 필요없음
```

클로저는 `익명메소드`이며 swift의 `메소드는 1급 객체`이기 떄문에

`대입, 매개변수로 넣기, 리턴 값으로 받기가 모두 가능`하다.

```swift
// 변수에 대입 ----------------------------------
let add = {(a: Int, b: Int) -> Int in
return a + b
}

// 매개변수로 받기 -------------------------------
func math(x: Int, y: Int, cal: (Int, Int) -> Int) -> Int {
return cal(x, y)
}

// 리턴값으로 받기 -------------------------------
func IsMul(x: Bool) -> (Int, Int) -> Int { 
if x {
return {(a: Int, b: Int) -> Int in return a * b}
} else {
return {(a: Int, b: Int) -> Int in return a + b}
}
}
let mul2 = IsMul(x: true)

print(mul2(3, 6)) // 18

let add2 = IsMul(x: false)

print(add2(3, 6)) // 9
```

### 후행 클로저(trailing closure)

클로저가 `함수의 마지막 argument`라면 마지막 `매개변수명을 생략`한 후 `함수 소괄호 외부`에
클로저를 `작성`.

```swift
func math(x: Int, y: Int, cal: (Int, Int) -> Int) -> Int {
return cal(x, y)
}

//클로저 소스를 매개변수에 직접 작성
result = math(x: 10, y: 20, cal: {(a: Int, b: Int) -> Int in
return a + b
}) 

//trailing closure
result = math(x: 10, y: 20) {(a: Int, b: Int) -> Int in
return a + b
}
```

### 클로저의 축약형

1. 리턴형을 생략할 수 있다.

```swift
func math(x: Int, y: Int, cal: (Int, Int) -> Int) -> Int {
return cal(x, y)
}
// 원본
var result = math(x: 10, y: 20, cal: {(val1: Int, val2: Int) -> Int in
return val1 + val2
}) 

//리턴형 생략
result = math(x: 10, y: 20, cal: {(val1: Int, val2: Int) in
return val1 + val2
}) 
print(result)
```

<br>

2. 매개변수를 생략하고 단축 인자(**Shorthand Argument Name**)를 사용 가능
    
    이때 키워드 `in`도 함께 생략 가능
    

```swift
// 매개변수 생략
result = math(x: 10, y: 20, cal: {
return $0 + $1
}) //매개변수 생략하고 단축인자(shorthand argument name)사용
print(result)
```

<br>

3. return 생략 가능

```swift
// return 생략
result = math(x: 10, y: 20, cal: {
$0 + $1
}) //클로저에 리턴값이 있으면 마지막 줄을 리턴하므로 return생략
print(result)
```

<br>

모든 축약과 후행클로저까지 적용하면 한 줄로 나타낼 수 있다.

```swift
result = math(x: 10, y: 20) { $0 + $1 } //return 생략
```

<br>

## 클래스

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/39b6f9eb-a9b6-4f51-8e35-fada792e5a1c/1cd1b40b-5d9b-4e54-bb4d-7361dc66ff92/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/39b6f9eb-a9b6-4f51-8e35-fada792e5a1c/ca154985-81bb-438b-ab88-6c935123bbcb/Untitled.png)

`다른 언어에서 필드`는 `swift에선 프로퍼티`라고 한다.

### 클래스 만들기

#### 저장 프로퍼티(stored property) 추가하기

stored property는 반드시 `초기값이 있거나`, `생성자가 있거나`, `옵셔널형으로 선언`되어야 함.

```swift
// 초기 값 추가
class Man{
    var age : Int = 0 // 초기 값을 지정해주지 않으면 에러 발생
    var weight : Double = 3.5
}
```

```swift
// 옵셔널 선언
class Man{
    var age : Int?
    var weight : Double!
}
```

#### 인스턴스 메서드와 인스턴스 만들기

```swift
class Man{
var age : Int = 1
var weight : Double = 3.5
func display(){ //인스턴스 메서드
print("나이=\(age), 몸무게=\(weight)")
}
}

var sung : Man = Man()
sung.display()//나이=1, 몸무게=3.5
```

#### 클래스 메소드

메소드 앞에 `class나 static 키워드`를 붙여서 선언할 수 있고,

이 메소드들은 인스턴스가 아닌 `클래스가 가지고 노는 메소드`이다.

class키워드로 만든 클래스 메서드는 자식 클래스에서 override가능

```swift
class Man{
var age : Int = 1
var weight : Double = 3.5
func display(){ //인스턴스 메서드
print("나이=\(age), 몸무게=\(weight)")
}
class func cM(){
print("cM은 클래스 메서드입니다.")
}
static func scM(){
print("scM은 클래스 메서드(static)")
}
}

var sung : Man = Man()
sung.display()//나이=1, 몸무게=3.5
Man.cM();
Man.scM();
```

### 생성자

인스턴스가 만들어질 때 실행되는 함수

생성자를 만들지 않으면 기본적으로 생성자가 만들어지지만

직접 생성자를 만들 경우 기본 생성자가 없어짐.

모든 프로퍼티 초기화 해주는 생성자의 경우 `designated initializer`라고 함

```swift
class Man{
var age : Int = 1
var weight : Double = 3.5
func display(){
print("나이=\(age), 몸무게=\(weight)")
}
init(yourAge: Int, yourWeight : Double){
age = yourAge
weight = yourWeight
} //designated initializer
}

var sung : Man = Man(yourAge : 30, yourWeight : 80.5)
sung.display();
```

### seft

다른 언어의 self나 this와 비슷한 역할

현재 클래스 내 메서드나 프로퍼티를 가리킬 때 메서드나 프로퍼티 앞에 `self.`을 붙임

```swift
class Man{
var age : Int = 1
var weight : Double = 3.5
func display(){
print("나이=\(age), 몸무게=\(weight)")
}
init(age: Int, weight : Double){
self.age = age //프로퍼티 = 매개변수
self.weight = weight
}
}
var sung : Man = Man(age:10, weight:20.5)

```

### 메소드 오버로딩

매개변수의 개수와 자료형이 다른 같은 이름의 함수를 여러 개 정의
매개변수가 다른 두 생성자를 통해 두가지 방법으로 인스턴스를 만들 수 있음

```swift
class Man{
var age : Int = 1
var weight : Double = 3.5
func display(){
print("나이=\(age), 몸무게=\(weight)")
}
init(age: Int, weight : Double){
self.age = age
self.weight = weight
} 
init(age: Int){ 
self.age = age
}
}

var sung : Man = Man(age : 30, weight : 80.5)
var sung1 : Man = Man(age:10) //2

sung.display()
sung1.display()
```