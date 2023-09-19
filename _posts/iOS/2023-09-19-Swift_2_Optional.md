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

# 연산자

swift의 연산자와 다른 언어의 연산자 사이에 차이가 있는 부분이 있다.

<details>
<summary>관련 사이트</summary>
<div markdown="1">    
https://bbiguduk.gitbook.io/swift/language-guide-1/basic-operators

https://developer.apple.com/documentation/swift/operator-declarations   
</div>
</details> 


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


# 클래스, 객체, 인스턴스

Class : 인스턴스를 찍어내는 틀

Object :  하나의 사물을 의미

Instance : 클래스를 기반으로 생성된 객체

![객체](https://github.com/novicehog/comments/assets/131991619/b97999fb-57fb-43ae-81fc-d5d655e084be)

<br>