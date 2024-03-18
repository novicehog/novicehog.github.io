---
layout: single
title:  "유니티 AvatarMask"
categories: 
    - Unity
tag: [Unity, Animation]
published: true

author_profile: true # 옆에뜨는 프로파일

search: true #검색해도 안뜨게함

date: 2024-03-18
last_modified_at: 2024-03-18

---

# 아바타 마스크
애니메이션 관련해서 작업을 하다보면 특정 상황에 딱 맞는 애니메이션이 없는 경우가 매우 많다.

예를 들어 앉아서 무언가를 줍거나, 달리면서 문을 열거나 등의 디테일한 애니메이션을 구하기는 쉽지 않다.

이럴 때 사용할 수 있는 방법은 두 애니메이션을 부위별로 다른 애니메이션을 재생하는 것이다.<br>
이를 위해 사용하는 것이 아바타 마스크이다.

## 아바타 마스크 생성
아바타 마스크는 프로젝트 창에서 생성한다.

![아바타마스크생성](https://github.com/novicehog/comments/assets/131991619/b5666d86-179f-424a-a3ea-8bec1f037d88)
<br>

생성하면 다음과 같은 아이콘이 생긴다.

![아바타마스크이미지](https://github.com/novicehog/comments/assets/131991619/5bd8440d-67ee-43f5-9167-3d7f0cea1a18)
<br>

아바타 마스크를 더블클릭해보면 인스펙터 창에서 Humanoid와 Transform을 볼 수 있다.

먼저 Humanoid는 매우 간편하게 사용할 수 있다. Humanoide형태의 모델만 사용할 수 있으며 다음과 같이
애니메이션을 재생할 파츠만 골라놓으면 된다.

![휴머노이드](https://github.com/novicehog/comments/assets/131991619/71913dd9-4c41-47e7-a480-d2e7ff17b5fb)
<br>

Transform은 직접 모델을 넣고 디테일하게 원하는 부위를 선택할 수 있다. <br>
Humanoid형태의 모델이 아니여도 가능하지만 디테일한 작업이기에 시간소요가 크다.

![트랜스폼](https://github.com/novicehog/comments/assets/131991619/96671014-3b7e-4e3f-9d41-5354761e449b)


## 아바타 마스크 적용
간단하게 하기위해 Humanoid를 사용하여 한다.

먼저 만든 Avatar Mask를 적용할 Animator로 간다. <br>
그 뒤,여기서 레이어 하나를 만든 뒤 오른쪽 위 톱니바퀴를 눌러 아바타 마스크를 추가해준다.

![적용](https://github.com/novicehog/comments/assets/131991619/4d58813e-b49c-494c-b200-af28bb0ab700)
<br>

이러면 이제 끝이다 이 레이어에서 실행되는 애니메이션은 위에서 설정한 Avatar Mask 부위만 애니메이션이 재생되게 된다.


## 적용 예시
문 열기 애니메이션을 아바타 마스크를 적용하여 재생하여, 앉기, 걷기, 뛰기 등 다른 애니메이션과 자연스럽게 재생된다.

![AvatarMask움짤](https://github.com/novicehog/comments/assets/131991619/b33c8b0c-6213-4b4a-8215-1c7170cdfea5)