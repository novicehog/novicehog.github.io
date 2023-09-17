---
layout: single
title:  "유니티 다크소울 따라만들기 ch_4 rolling 구르기"
categories: 
    - Unity
tag: [Unity, DarkSouls]
published: true

author_profile: true # 옆에뜨는 프로파일

search: true #검색해도 안뜨게함

date: 2023-09-17
last_modified_at: 2023-09-17

---

<!-- 
{: .notice--warning} // 알림 강조
{: .notice--success} // 초록색 강조
{: .notice--danger } // 초록색 강조
{: .notice--info}
{: .notice--primary}
{: .notice}

{: .H1-font}         // 제목 색
<span style="color:Skyblue"> 색 넣기 </span>
<br/> 한줄 내리기

<details>
<summary>VR</summary>
<div markdown="1">       
</div>
</details> 
 -->



이 글은 유튜브 **Sebastian Graves Create Dark Souls**를 보고 따라 만들면서 헷갈리는 부분을 정리한 글입니다.
{: .notice--warning}

# 구현 개요
이번엔 플레이어 행동 중 하나인 구르기가 구현된다.

이번에 구현되는 구르기는 애니메이션 자체에 저장되어있는 움직임을 통해 구현한다.

# 초기 설정

1. PlayerControls(InputSystem) 에 새로운 입력 할당
2. PlayerManager 스크립트 추가
3. 애니메이터에 애니메이션 추가
4. 애니메이션에 ResetIsInteracting 스크립트를 Behavior로 추가
5. 여러 스크립트들에 내용 추가
6. 버전 업 되면서 영상의 방법으로 되지 않는 버그 수정

##  PlayerControls(InputSystem) 에 새로운 입력 할당

버튼 형식으로 두가지 입력값 추가

![image](https://github.com/novicehog/comments/assets/131991619/f5296acc-4112-40f8-8545-26432b6b3f56)

## PlayerManager 스크립트 추가

<details>
<summary> PlayerManager  </summary>
<div markdown="1">   

```c#
public class PlayerManager : MonoBehaviour
{
    InputHandler inputHandler;
    Animator anim;
    void Start()
    {
        inputHandler = GetComponent<InputHandler>();
        anim = GetComponentInChildren<Animator>();
    }

    void Update()
    {
        inputHandler.isInteracting = anim.GetBool("isInteracting");
        inputHandler.rollFlag = false;
    }
}

```
</div>
</details> 

InputHandler의 입력 관련 값들을 여기서 갱신시켜준다.


## 애니메이터에 애니메이션 추가

애니메이션과 파라미터 추가.
구르기의 경우 애니메이션 전환을 스크립트에서 설정해주기 때문에
따로 Conditions에 조건을 추가해주지 않음.

![image](https://github.com/novicehog/comments/assets/131991619/52affa5c-4e06-4cb7-a74c-73278526006c)