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

## 배경 추가하기
배경을 추가할 때는 큰 이미지를 추가한 뒤 `alpha을 통해 투명도를 조절`하고 <br>
오브젝트가 맨 뒤로 깔리게 하기 위해 `순서를 위로 옮긴다.`

![1](https://github.com/novicehog/comments/assets/131991619/866e98cf-9452-4550-ad47-48a342fec578)

## 스위치 추가하기
오브젝트 중 스위치를 추가하고 다음과 같이 설정하여 스크립트에 연결하여 사용한다.

![2](https://github.com/novicehog/comments/assets/131991619/7920900d-13c9-4864-876b-64e8be3f114e)

스위치는 다음과 같이 sender를 통해 스크립트에서 사용할 수 있음.

```swift
@IBAction func onoff(_ sender: UISwitch) {
        if sender.isOn{
            print("On")
        }
        else{
            print("Off")
        }
    }
```

## 새그먼티드 컨트롤 추가하기

![3](https://github.com/novicehog/comments/assets/131991619/cf9f853a-1a51-4d5f-bb68-f4a937bc8d42)


새그먼티드 컨트롤 UI의 text는 이곳에서 바꾼다.

![4](https://github.com/novicehog/comments/assets/131991619/775d086b-6009-4155-a8e2-76acb399c381)


이것도 다음과 같이 추가하여 사용한다.

![5](https://github.com/novicehog/comments/assets/131991619/c9a7b309-4bba-44df-ab74-75f8bd61d7a8)


또한 스크립트에서 사용은 다음처럼 한다


```swift
@IBAction func SC(_ sender: UISegmentedControl) {
        if sender.selectedSegmentIndex == 0{
            print("여성")
        }
        else
        {
            print("남성")
        }
    }
```



## 탭바 추가하기

뷰 선택 후 에디터에서 추가

![6](https://github.com/novicehog/comments/assets/131991619/fcff54eb-946e-4bde-b414-484619ee3ebb)

![7](https://github.com/novicehog/comments/assets/131991619/8dfda9ba-6593-47f5-af0d-ab48424bc82e)

<br>

탭바 아이콘은 추가된 별표 모양의 오브젝트를 수정하면 된다.

![8](https://github.com/novicehog/comments/assets/131991619/15ab79ed-067c-4eba-ac0d-95d8d71cdc99)

<br>

화면은 뷰컨트롤러를 추가하여 추가한다.

![9](https://github.com/novicehog/comments/assets/131991619/775a81be-eebd-461e-b1a3-0e063c7885e9)



뷰끼리의 연결은 컨트롤키를 누르고 드래그한 뒤 릴레이션십 세그웨이에서 뷰 컨트롤러를 선택해준다.

![10](https://github.com/novicehog/comments/assets/13199161999c70b1d-703b-4b44-a96d-75a2c2b50ca7)


tab bar의 UI에 대해서는 https://developer.apple.com/design/human-interface-guidelines/tab-bars에서 찾아보면 된다.


## 동영상 추가
![11](https://github.com/novicehog/comments/assets/131991619/322d1ac7-2651-44b4-9e62-962e8cc76c7e)

![12](https://github.com/novicehog/comments/assets/131991619/b5264c9f-9c59-4346-80ed-7f7bbd06870f)

동영상 추가

동영상을 드래그해서 추가할 때 꼭 Add to targets를 체크해야함

![13](https://github.com/novicehog/comments/assets/131991619/4c3de89a-a590-4bb8-992e-0c10defb3b99)

스크립트와 뷰 는 꼭 연결해주어야함

![14](https://github.com/novicehog/comments/assets/131991619/8f33a1d1-d85b-4bc9-8378-ffaef69713bf)


### 동영상 버튼 연결

버튼을 추가하고 버튼을 액션으로 추가한 뒤 다음 코드를 안에 넣어준다.

```swift
let file:String? = Bundle.main.path(forResource:"bmi", ofType: "mp4")
let url = NSURL(fileURLWithPath: file!)
let playerController = AVPlayerViewController()
let player = AVPlayer(url: url as URL)
playerController.player = player
self.present(playerController, animated: true)
player.play()
```

## 웹페이지 로드하기
버튼과 웹킷뷰를 추가한다.

![15](https://github.com/novicehog/comments/assets/131991619/cee7f481-7edc-4c9e-8b87-b786d470c543)

새 소스코드를 만들어서 버튼, 새그먼티드 컨트롤, 웹킷뷰를 연결한 뒤 다음과 같이 작성해준다.

```swift
import UIKit
import WebKit

class WebViewController: UIViewController {

    @IBOutlet var webView: WKWebView!
    override func viewDidLoad() {// 시작할 때 실행
        super.viewDidLoad()
				// 기본으로 블로그 주소 로드
        let myURL = URL(string:"https://novicehog.github.io/")
        let myRequest = URLRequest(url: myURL!)
        webView.load(myRequest)

    }

		// 새그먼티드 컨트롤 값을 바꿀 때마다 페이지 로드
    @IBAction func GoWeb(_ sender: UISegmentedControl) {
        if sender.selectedSegmentIndex == 0{
            let myURL = URL(string:"https://www.naver.com")
            let myRequest = URLRequest(url: myURL!)
            webView.load(myRequest)
        }
        else{
            let myURL = URL(string:"https://www.google.com")
            let myRequest = URLRequest(url: myURL!)
            webView.load(myRequest)
        }
        
    }
    
    // 버튼을 누르면 네이버로 로드
    @IBAction func GoNaver(_ sender: UIButton) {
        let myURL = URL(string:"https://www.naver.com")
        let myRequest = URLRequest(url: myURL!)
        webView.load(myRequest)
    }
}
```

1