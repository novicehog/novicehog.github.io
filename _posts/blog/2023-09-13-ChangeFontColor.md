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
<br/>
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
귀찮지만 <span style="color:pink">세</span><span style="color:lightgreen">세</span><span style="color:skyblue">히</span> 바꿀 수 있다.


## minimal mistakes의 _sass 폴더 수정하기
minimal mistakes의 _sass 경로의 파일을 수정하여 일괄 수정이 가능하다.
_sass의 경로를 통해 수정하면 매 포스팅마다 색을 지정해주지 않아도 자동으로 색이 설정된다.

_base.scss나 _page.scss를 고치는 방법도 있지만
이번에는 skin을 통해 바꿔보겠다.

![image](https://github.com/novicehog/comments/assets/131991619/56c0d649-c656-4edc-97fb-7ab207e3b98e)

나는 _dark.scss를 쓰므로 바꿔보겠다.

바꾸고 싶은 글자를 바꾸고 싶으면 직접 페이지에 가서 F12를 눌러 무엇을 바꿀지 찾아야 한다.
나는 이번에 `타이틀 제목`을 바꿔보겠다.


페이지로 가서 F12를 누른뒤 다음 버튼을 누른다.
![image](https://github.com/novicehog/comments/assets/131991619/8eb3c9f5-9293-448e-8fb7-14b314802012)
<br/>

그 다음 수정을 원하는 부분의 마우스를 갖다 대면 다음과 같이 보인다.
![image](https://github.com/novicehog/comments/assets/131991619/51a7a82b-fa45-4802-aaf6-60f8751a547a)
<br/>

현재 이 부분은 a태그를 쓰고있음을 알 수 있고
좀 더 정확한 정보를 알기위해 개발자도구를 보면
사진과 같이 보인다.
![image](https://github.com/novicehog/comments/assets/131991619/a8b5efec-a5a4-494a-bbe0-c8287a9dc66b)
<br/>

여기서 .page__title p-name 클래스의 a태그임을 알 수 있다.
이 것을 토대로 _dark.scss로 가서


```scss
.page__title.p-name{
  a{
    color: #FF0000; //원하는 색깔, 여기선 빨간색
  }
}
```
를 맨 밑줄에 추가해주면 된다.

그러면 제목의 색이 바뀐 것을 알 수 있다.

![image](https://github.com/novicehog/comments/assets/131991619/c7da1573-4919-4ec5-b11e-fbe867fbc4c1)


***
똑같은 방법으로
h1, h2태그도 바꿔줄 수 있다.
```scss
.page__content.e-content{
  h1 {
  color: #ffb443e8; /* 원하는 색상 코드 */
 }
}
.page__content.e-content{
  h2{
    color: #ffbca2!important; /* 원하는 색상 코드 */
  }
}
```
여기서 h태그를 바꿀 때 주의할 점은
.page__content.e-content 클래스로 특정해주지 않으면 모든 h1, h2태그가 바뀌게 된다는 문제점이 있으므로
주의해야한다.


이런 식으로 매 포스팅 글마다 공통으로 바꿔야할 옵션을
개발자 도구를 통해 추적하고 _scss파일을 수정하여 커스터마이징 할 수 있다.


