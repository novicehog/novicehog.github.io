---
layout: single
title:  "유니티 AvatarMask"
categories: 
    - Unity
tag: [Unity, Animation]
published: true

author_profile: true # 옆에뜨는 프로파일

search: true #검색해도 안뜨게함

date: 2024-03-19
last_modified_at: 2024-03-19

---

# 애니메이션 싱크
애니메이션을 넣을 때 캐릭터에 상태에 따라 애니메이션이 달라지게 하고 싶을 때가 있다.<br>
예를 들어 지침 상태, 다침 상태 등의 경우 기존의 움직임 애니메이션을 지친 움직임, 다친 움직임으로 바꾸고 싶은 경우가 있다.

이때 하나하나 파라미터값을 추가하고 트랜지션을 추가하는 방식으로 구현하게 되면 나중에 애니메이션 컨트롤러가 매우 복잡해지게 된다.

이를 방지하기위해 `애니메이션 싱크` 기능을 이용해서 레이어를 추가로 만들어 `기존 레이어 위에 덮어씌워 재생`하는 방식으로 구현이 가능하다.

## 애니메이션 싱크 사용하기
싱크 사용은 매우 간단하다. 레이어를 하나 추가로 만들어 주고 오른쪽 위에 옵션창에서 Sync를 체크해준 뒤, 덮어씌울 레이어를 골라주면 된다.

![싱크레이어생성](https://github.com/novicehog/comments/assets/131991619/5798b170-7c8a-482f-879d-ed9c57088783)
<br>

나는 여기서 기존 BaseLayer를 덮어씌우는 Tired레이어를 추가하였다.


이렇게 하면 덮어씌우는 레이어의 맞춰서 자동으로 State가 생성된다.

베이스 레이어 이미지 
![베이스레이어](https://github.com/novicehog/comments/assets/131991619/7d5353b6-8209-41d0-a46d-058b35f065a8)
<br>


싱크 레이어 이미지
![싱크레이어](https://github.com/novicehog/comments/assets/131991619/aeb83fb3-5355-4f90-b8c8-68106a8cd3ed)
<br>


이때 주의해야할 점은 만약 베이스레이어에서 구현한 State가 Blender Tree형태일 경우 싱크 레이어에서는 Blender Tree로 복제되는 것이 아니라
그냥 State로 복제되는 점은 유의해야한다. 즉, 상세하게 블렌더트리도 일일히 맞추려면 새로 블렌더 트리를 생성해줘야한다.


## 코드로 제어하기
코드로 제어하는 것은 다른 레이어들과 다르지 않다. <br>
그냥 싱크 레이어의 Weight를 0~1로 설정하여 0이면 아무런 역할도 하지 않고 1로 가까워 질 수록 기존 레이어를 덮어씌우게 된다.<br>
참고로 레이어는 위에서부터 0, 1, 2 ... 의 인덱스를 가진다.

![레이어 순서](https://github.com/novicehog/comments/assets/131991619/861f07ef-f253-439e-ab90-813f9edb8730)

```cs
animator.SetLayerWeight(layerIndex, weight);
```


## 사용 예시
나는 스테미너 기능을 구현하였기에 거기에 맞춰 스테미너가 0에 가까워 질 수록 지침 애니메이션을 덮어씌우도록 하였다. <br>
덮어씌우는 이미지는 단순하게 고개를 숙이는 정도로 하였다.

![AnimationSync움짤](https://github.com/novicehog/comments/assets/131991619/1ec1aaee-ed73-4957-ad2a-b36af27f9dbd)