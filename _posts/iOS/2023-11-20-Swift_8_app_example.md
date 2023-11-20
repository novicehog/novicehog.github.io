---
layout: single
title: "xcode 예제"
categories: 
    - iOS
tag: [iOS,Swift]
published: true

author_profile: true # 옆에뜨는 프로파일

date: 2023-11-20
last_modified_at: 2023-11-20
---


아래 소스는 다음 책의 소스를 실습한 내용입니다. **Do it! 스위프트로 아이폰 앱 만들기 입문, 송호정, 이범근 저,이지스퍼블리싱, 2023년 01월 20일**
{: .notice--warning}

인터넷에서 받은걸 처음 다운로드할때 뜨는 창

![1](https://github.com/novicehog/comments/assets/131991619/4b720de5-f051-4d92-af25-85f95d8eda2b)

<br>

***

시뮬레이터 캡처 버튼은 다음과 같다. <br>
누르면 화면이 깔끔하게 캡쳐됨

![2](https://github.com/novicehog/comments/assets/131991619/f365fdb9-8089-4446-9c8b-10bc736fec97)

![3](https://github.com/novicehog/comments/assets/131991619/7d63634c-a289-4999-a24e-74e961912f4b)


<br>

***
`프로젝트 설정`

`minimum deployments`는 `최소 버전`을 의미 <br>
`Bundle identifier`은 `앱의 식별자 역할`을 함(유일해야함) <br>
`iPhone orientation` 은 `핸드폰의 회전`을 나타냄

![4](https://github.com/novicehog/comments/assets/131991619/03d5fc3e-7273-4879-8b82-8d7dfd80318f)

<br>

***


프로젝트에 기본으로 주어지는 세개의 swift파일이 있다.

보통은 ViewController만을 거의 사용함.

![5](https://github.com/novicehog/comments/assets/131991619/26f07f49-7dc7-4bf1-8cf9-130e4320bdd6)

<br>

***
`main 스토리 보드`는 `디자인적인 요소`들을 나타낸다.

`safe Area`는 스마트폰의 기본적인 `UI 공간을 비워두고 나머지 작업 공간`을 뜻한다.

![6](https://github.com/novicehog/comments/assets/131991619/38bec02d-5e6e-41bd-8e88-b4d09437dedc)

<br>

***


버튼을 `마우스 오른쪽클릭`으로 누르면 `버튼에 여러가지 정보`를 볼 수 있음. <br>
여기서 보이는 정보 중 이벤트로 등록되어 있는 Touch up inside 는 버튼을 눌렀다 뗄 때 호출된다.

![7](https://github.com/novicehog/comments/assets/131991619/3eb62e0d-76e5-4a1d-aef6-c35c17da9e26)


***

<br>

view Controller에서 `connections inspector`를 통해 `outlets(변수)`나 `Action(함수)`의 `연결 상태`를 볼 수있다.


![13](https://github.com/novicehog/comments/assets/131991619/c4ef466d-de8d-4c81-b241-71a0a5ead7ee)

![8](https://github.com/novicehog/comments/assets/131991619/15ebad31-6d8a-4a9b-b07b-865ee3e76fd9)

<br>

***


`text입력창`의 경우 `PlayerHolder를 이용해서 안내문구를 작성`할 수 있다.

![9](https://github.com/novicehog/comments/assets/131991619/93b053be-00c0-4905-9dc3-1bd7a24c9333)

<br>

***

`date picker`

`날짜 기능`을 가지고 있는  `오브젝트`이다.

모드는 표시형식에 따라 네 가지가 있다.

![10](https://github.com/novicehog/comments/assets/131991619/5be5ee5b-4724-4745-9424-b179f88be87d)

***

`Tab bar`를 사용하여 `여러 페이지로 구성`할 수 있다.

![11](https://github.com/novicehog/comments/assets/131991619/ce204f8c-4769-4111-8980-0862fe3132c0)

tab bar와 비슷하게 `navigation controller` 도 있다.

이것도 역시 여러 페이지 구성을 가능하게 해준다.

![12](https://github.com/novicehog/comments/assets/131991619/1f432cf6-a04a-4434-8aa5-8ff1787b2bd6)


***

`간단한 BMI 코드`

```swift
import Foundation
class BMI {
    var weight : Double
    var height : Double
    init(weight:Double, height:Double){
        self.height = height
        self.weight = weight
    }
    func calcBMI() -> String {
        let bmi = weight/(height * height * 0.0001) // kg/m*m
        let shortenedBmi = String(format: "%.1f", bmi)
        var body = ""
        
        switch bmi {
        case 40...:
            body = "3단계 비만"
        case 30..<40:
            body = "2단계 비만"
        case 25..<30:
            body = "1단계 비만"
        case 18.5..<25:
            body = "정상"
        default:
            body = "저체중"
        }
        
        return "BMI:\(shortenedBmi), 판정:\(body)"
    }
}
var han = BMI(weight:62.5, height:172.3)
print(han.calcBMI())
```

**이 코드는 BMI(체질량지수)를 계산하는 클래스를 정의하고, 이를 사용해서 한 사람의 BMI를 계산하는 코드다.**

**`import Foundation`: Foundation 프레임워크를 불러옴. 이 프레임워크는 다양한 유틸리티, 데이터 타입 등을 제공.**

**`class BMI`: BMI라는 이름의 클래스를 선언.`var weight : Double`, `var height : Double`: BMI 클래스의 인스턴스 변수로, 몸무게와 키를 나타내는 두 개의 변수를 선언.**

**`init(weight:Double, height:Double)`: BMI 클래스의 초기화 메서드. 이 메서드는 몸무게와 키를 매개변수로 받아서, 이를 인스턴스 변수에 저장. 여기에서 `self`는 인스턴스 자신을 가리킴.**

**`func calcBMI() -> Double`: BMI를 계산하는 메서드입니다. 이 메서드는 몸무게를 키의 제곱으로 나눈 값을 반환합니다. 여기에서 키는 cm 단위이므로, m 단위로 변환하기 위해 0.0001을 곱합니다.**

**`var choi = BMI(weight:62.5, height:172.3)`: BMI 클래스로부터 한 객체를 생성. 이 객체의 몸무게는 62.5kg이고, 키는 172.3cm.`print(choi.calcBMI())`: choi 객체의 BMI를 계산하고, 이를 출력.**

**따라서 이 코드를 실행하면, 몸무게가 62.5kg이고 키가 172.3cm인 한 사람의 BMI를 계산해서 출력함.**



<br>

***

`앱 개발 절차`

![14](https://github.com/novicehog/comments/assets/131991619/d966bf0a-2cdb-408d-a3e0-b33bd543958c)