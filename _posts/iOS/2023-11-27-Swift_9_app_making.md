---
layout: single
title: "xcode 간단한 앱"
categories: 
    - iOS
tag: [iOS,Swift]
published: true

author_profile: true # 옆에뜨는 프로파일

date: 2023-11-27
last_modified_at: 2023-11-27
---



# 간단한 BMI앱 만들기

## 사용할 메인 소스코드
```swift
import Foundation
let weight = 60.0
let height = 170.0
let bmi = weight / (height*height*0.0001) // kg/m*m
let shortenedBmi = String(format: "%.1f", bmi)
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

print("BMI:\(shortenedBmi), 판정:\(body)")
```


## 디자인 방법들

다음과 같이 디자인을 한다.

![1](https://github.com/novicehog/comments/assets/131991619/10cb9603-7bab-4f7b-9f59-9b37ae76a20e)

### 이모티콘 사용
이모티콘은 edit에서 사용가능

![2](https://github.com/novicehog/comments/assets/131991619/a3357799-ec85-4099-9e86-937eaf853a59)


### 핸드폰을 눕힐시 디자인 문제
위의 디자인은 문제가 있는데 바로 옆으로 눕히면 다음과 같이 예상하지 못한 문제점이 발생한다.

![3](https://github.com/novicehog/comments/assets/131991619/1e3521db-043c-4604-bf81-5d02e0a605a5)

위의 문제가 발생할 때는 다음과 같은 경고가 함께 발생한다.

![4](https://github.com/novicehog/comments/assets/131991619/27d6646c-61aa-4e22-9056-ce2b2465fd52)

경고에서 말하는 레이아웃 `제약조건(Layout Constraints)` 은 화면에 있는 `UI 요소들의 위치와 크기를 결정하는 규칙`을 말함. <br>
예를 들어, 버튼이 화면의 정중앙에 위치하도록 설정하려면, 해당 버튼의 중심이 화면의 중심과 일치하도록 하는 제약조건을 추가해야 함. <br>
또는, 이미지가 화면의 상단과 20포인트 떨어져 있도록 설정하려면,<br>
해당 이미지의 상단 테두리가 화면 상단과 20포인트 거리를 유지하도록 하는 `제약조건이 필요`함.

위에서는 이러한 조건들을 설정해주지 않았기에 화면 전환 시 UI배치에 문제가 생기게 됨

### conner round주기

결과는 다음과 같음.

![5](https://github.com/novicehog/comments/assets/131991619/c628e579-440a-4608-a746-09f31bc55a32)

이는 두가지 방법이 있음

하나는 attribute를 다음과 같이 설정하는 것

![6](https://github.com/novicehog/comments/assets/131991619/fe38c50f-4529-4868-88f4-68caa2d514dd)


나머지 하나는 코드를 둥글게 하고싶은 UI라벨을 코드로 가져와서 다음과 같이 설정한다.<br>

lblResult는 라벨 변수중 하나임.

```swift
lblResult.clipsToBounds = true
lblResult.layer.cornerRadius = 10
```

### text Feild 숫자만 입력받기

text Feild에 Keyboard Type을 수정함으로 입력 키보드 형태를 바꿀 수 있다.

숫자만 입력받고싶으면 Decimal Pad로 한다.

자세한 내용은 여기서 참고 https://developer.apple.com/documentation/uikit/uikeyboardtype

![7](https://github.com/novicehog/comments/assets/131991619/850ec6a1-d03f-4223-9b40-c404b5ecd52f)

![8](https://github.com/novicehog/comments/assets/131991619/dac69ca8-9b3d-4e7c-acd8-22d853f6a039)

### 변수와 함수 연결

ctrl 키를 누르고 어시스턴트로 드래그하여 연결할 수 있다. 연결하면 줄표시 영역에 원 모양이 생긴다.

![9](https://github.com/novicehog/comments/assets/131991619/c93bf699-d63c-43fd-b8b9-d7255f635bd3)

스크립트와 UI를 연결하고 난 뒤에 Connections inspertor에서 연결 상태를 확인 가능

![10](https://github.com/novicehog/comments/assets/131991619/c11e4ba5-ac49-473a-8d94-33ff9aba3e49)

### 색 바꾸기

UIColor의 displayP3 를 통해서 색깔을 설정하고 글씨색이나 배경색을 바꿀 수 있다.

```swift
var color = UIColor.white
color = UIColor(displayP3Red: 1.0, green: 0.0, blue: 0.0, alpha: 1.0)
lblResult.backgroundColor = color
lblResult.textColor = color
```

### 전체 코드
```swift
import UIKit

class ViewController: UIViewController {
    
    @IBOutlet var txtHeight: UITextField!
    @IBOutlet var txtWeight: UITextField!
    @IBOutlet var lblResult: UILabel!
    @IBAction func CalcBMI(_ sender: UIButton) {
        if txtHeight.text == "" || txtWeight.text == "" {
            lblResult.textColor = .red
            lblResult.text = "키와 체중을 입력하고 실행하세요!"
            return
        } else {
            let weight = Double(txtWeight.text!)!
            let height = Double(txtHeight.text!)!
            let bmi = weight / (height*height*0.0001) // kg/m*m
            let shortenedBmi = String(format: "%.1f", bmi)
            var body = ""
            var color = UIColor.white
            if bmi >= 40 {
                color = UIColor(displayP3Red: 1.0, green: 0.0, blue: 0.0, alpha: 1.0)
                body = "3단계 비만"
            } else if bmi >= 30 && bmi < 40 {
                color = UIColor(displayP3Red: 0.7, green: 0.0, blue: 0.0, alpha: 1.0)
                body = "2단계 비만"
            } else if bmi >= 25 && bmi < 30 {
                color = UIColor(displayP3Red: 0.4, green: 0.0, blue: 0.0, alpha: 1.0)
                body = "1단계 비만"
            } else if bmi >= 18.5 && bmi < 25 {
                color = UIColor(displayP3Red: 0.0, green: 0.0, blue: 1.0, alpha: 1.0)
                //color = UIColor.blue
                body = "정상"
            } else {
                color = UIColor(displayP3Red: 0.0, green: 1.0, blue: 0.0, alpha: 1.0)
                body = "저체중"
            }
            print("BMI:\(shortenedBmi), 판정:\(body)")
            lblResult.backgroundColor = color
            lblResult.clipsToBounds = true
            lblResult.layer.cornerRadius = 10
            lblResult.text = "BMI:\(shortenedBmi), 판정:\(body)"
        }
    }
    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.
    }
    
    
}
```


![11](https://github.com/novicehog/comments/assets/131991619/cfef719b-3880-4800-948a-3635ce9c83b8)
