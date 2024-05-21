---
layout: single
title:  "유니티 DecoratorDrawer"
categories: 
    - Unity
tag: [Unity, Editor]
published: true

author_profile: true # 옆에뜨는 프로파일

search: true #검색해도 안뜨게함

date: 2024-05-21
last_modified_at: 2024-05-21

---

## DecoratorDrawer 란
유니티에서는 Editor에서 보여지는 여러 요소들을 `커스터마이징` 할 수 있도록 도와준다. 

여기서 볼 DecoratorDrawer는 그 중 한가지로, property를  데이터를 기반으로 `장식적인 요소`를 그리는 것이다. `(property 자체를 그리는 것이 아님)`

즉, 속성 주변을 꾸며주는 기능이라고 보면 된다.

## 사용 상황
일반적으로 인스펙터 창에서 보여지는 변수들은 다음과 같이 밋밋하다.

![image](https://github.com/novicehog/comments/assets/131991619/9c569660-5b09-4630-a384-f7d099e8351d)
<br>

