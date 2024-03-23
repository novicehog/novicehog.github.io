---
layout: single
title:  "유니티 절차적 애니메이션 [Procedural animation]"
categories: 
    - Unity
tag: [Unity, Animation]
published: true

author_profile: true # 옆에뜨는 프로파일

search: true #검색해도 안뜨게함

date: 2024-03-23
last_modified_at: 2024-03-23

---
직접 이해한 내용을 바탕으로 작성한 글이기에 틀린 내용이 있을 가능성이 있습니다.
{: .notice--warning}

# 절차적 애니메이션
기본적으로 사용하는 키 프레임 애니메이션은 정해진 시간동안 정해진 움직임을 수행한다.<br>
그러니까 늘 같은 움직임만을 재생한다.

절차적 애니메이션이란 실시간으로 애니메이션의 움직임을 수정하는 방법이다.
예를들어 땅 지형에 맞게 발 위치가 조정된다거나, 캐릭터 머리가 어떤 사물을 바라보는 것이 있다.

기본적으로 유니티 툴 외에서 캐릭터 애니메이션을 작업한다 했을때 캐릭터는 `Rigging`이라는 작업을 한다. <br>
Rigging이란 캐릭터 모델에게 `Bone(관절)`과 그에 따른 `Constraint(제약조건)`을 설정해주는 것이다.

먼저 `Bone`이란 흔한 동물의 관절과 같은 것으로 캐릭터의 몸에 관절과 같은 부위에 위치하며
`Weight(가중치)`에 따라 자신에게 주어진 `주변 신체 부위들을 움직이는 역할`을 한다. <br>
흔히 다른 3D툴에서 Bone에 따른 주변 신체에 가중치를 주는 작업을 Weight Painting이라 한다.

그 다음 `Constraint`란 Bone에게 어떠한 규칙을 부여하는 것이다.<br>
예를 들어 현실에서의 우리는 어깨를 움직이면 어깨 밑의 팔들이 함께 움직인다.<br>
그러니까 어깨 밑의 팔들은 어깨의 움직임을 따른다 다른 제약조건을 가지는 것이다.<br>

유니티에서 제공하는 Animation Rigging은 이러한 Bone과 Constraint를 유니티에서도<br>
설정할 수 있도록 도와준다. 절차적 애니메이션은 이러한 Animation Rigging 패키지를 통해 구현하면 매우 편하다.  



## 절차적 애니메이션 적용 방법
우선은 패키지 매니저(Unity Registry)에서 Animation Rigging을 설치한다.
![패키지 다운](https://github.com/novicehog/comments/assets/131991619/72b4303e-09f7-4a01-ae47-d6b022cee4e8)
<br>


***
애니메이터가 있는 위치에 Rig Buiduer 컴포넌트를 추가해줌.
![리그빌더](https://github.com/novicehog/comments/assets/131991619/09dfb047-3c50-4be6-9af3-062a068e8cf6)


유니티에서 본이 시각적으로 보이고 선택하기 쉽게 하기 위해서 `Bone Renderer`를 추가해줌.
그리고 Bone Renderer의 트랜스폼으로 본들을 넣어줌.<br>
본들을 직접 수동으로 넣어줘도 되지만 그냥 애니메이터가 있는 오브젝트 선택 후 <br>
유니티 상단에 있는 도추창에서 Animation Rigging -> Bone Renderer Setup에서도 가능

![Animation Rigging](https://github.com/novicehog/comments/assets/131991619/ab963c33-5a79-410d-826d-a4f8f6a1066b)
<br>

![컴포넌트창](https://github.com/novicehog/comments/assets/131991619/a32dfece-55b7-43f8-86df-e0f34a5bda13)
<br>

*** 
이제 리그 빌더에 추가해줄 리그들을 생성할 차례이다.<br>
위에서도 말했듯 리그는 Bone + Constraint 이다. 즉 해당되는 본들과 그 본들의 제약규칙을 정해주어야한다.<br>

Rig를 추가하기 위해 먼저 애니메이터가 있는 게임 오브젝트의 자식으로 빈 오브젝트를 생성 후<br>
Rig컴포넌트를 넣어준다. 

![리그](https://github.com/novicehog/comments/assets/131991619/eedea219-f342-46dc-919b-d22a8d820cf2)
<br>

그 다음 다시 이 오브젝트 밑으로 빈 오브젝트를 생성한다. 그리고 유니티에서 제공하는 Constraint 컴포넌트들 중 하나를 선택하여 넣는다.

나는 간단하게 multi-Aim Constraint를 넣어주었다. 이 기능은 머리가 바라보는 방향을 설정할 때 쓰기 좋다.<br>
Constrained Object에 머리Bone을 넣어주었고 타겟으로 문을 넣어주었다.

![image](https://github.com/novicehog/comments/assets/131991619/d93c38dc-4517-48fa-ac5c-13b4e16a0610)
<br>

이제 리그가 있는 게임오브젝트를 리그 빌더의 Layer로 드래그해주면 적용된다.

![추적](https://github.com/novicehog/comments/assets/131991619/44c45c9c-5584-47c9-acc1-2d775ba2cc29)

