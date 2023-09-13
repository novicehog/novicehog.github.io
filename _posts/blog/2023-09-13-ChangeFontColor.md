---
layout: single
title:  "minimal mistakes 글씨 색깔 바꾸기"
categories: 
    - blog
tag: [blog, minimal mistakes]
published: true

author_profile: true # 옆에뜨는 프로파일

date: 2023-09-13
last_modified_at: 2023-09-13
---

# 글씨 색 바꾸기

minimal mistakes 를 기본 테마로 하는 github.io 블로그에서
글씨 색을 바꾸는 방법은 여러가지가 있다.

방법은 다음과 같다

## html css를 통해서 바꾸기

span을 이용해서 글씨색을 직접 바꿀 수 있다.


```html
<span style="color:skyblue">하늘색 입니다.</span>
```
<span style="color:skyblue">하늘색 입니다.</span>

span을 이용할 경우 작성하는 post글 안에서 원하는 글짜마다 수정해야하므로
귀찮지만 세세히 바꿀 수 있다.

## minimal mistakes의 _sass 폴더 수정하기
minimal mistakes의 _sass 경로의 파일을 수정하여 일괄 수정이 가능하다


