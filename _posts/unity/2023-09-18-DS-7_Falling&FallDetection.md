---
layout: single
title:  "유니티 다크소울 따라만들기 ch_7 낙하"
categories: 
    - Unity
tag: [Unity, DarkSouls]
published: true

author_profile: true # 옆에뜨는 프로파일

search: true #검색해도 안뜨게함

date: 2023-09-18
last_modified_at: 2023-09-18

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
이번엔 플레이어 행동 중 하나인 낙하를 구현한다.

레이캐스트를 통해 땅을 인식하여 낙하를 구현하며,


# 초기 설정

1. 애니메이션 추가
2. 스크립트들 내용 추가


## 애니메이션 추가

떨어지는 애니메이션과 착지 애니메이션이 필요.

![image](https://github.com/novicehog/comments/assets/131991619/863f7fb0-4255-4f80-ba2c-7b15403f28cc)
<br>

착지 애니메이션

![image](https://github.com/novicehog/comments/assets/131991619/40b46d7a-3d90-4748-9f7b-c4d9e69fdbfb)
<br>

낙하 애니메이션

![image](https://github.com/novicehog/comments/assets/131991619/82fd2d3a-c50e-4051-83af-539c278f0016)

## Scripts 내용 추가
PlayerManager과 PlayerLocomotion이 수정됨.

### PlayerManager
***
<details>
<summary>PlayerManager</summary>
<div markdown="1">    

```c#
public class PlayerManager : MonoBehaviour
{
    InputHandler inputHandler;
    Animator anim;
    CameraHandler cameraHandler;
    PlayerLocomotion playerLocomotion;

    public bool isInteracting;

    [Header("Player Flags")]
    public bool isSprinting;
    public bool isInAir;
    public bool isGrounded;
    void Start()
    {
        inputHandler = GetComponent<InputHandler>();
        anim = GetComponentInChildren<Animator>();
        playerLocomotion = GetComponent<PlayerLocomotion>();
    }

    private void Awake()
    {
        cameraHandler = CameraHandler.singleton;
    }

    void Update()
    {
        float delta = Time.deltaTime;
        isInteracting = anim.GetBool("isInteracting");

        inputHandler.TickInput(delta);
        playerLocomotion.HandleMovement(delta);
        playerLocomotion.HandleRollingAndSprinting(delta);
        playerLocomotion.HandleFalling(delta, playerLocomotion.moveDirection);
    }

    private void FixedUpdate()
    {
        float delta = Time.fixedDeltaTime;

        if (cameraHandler != null)
        {
            cameraHandler.FollowTarget(delta);
            cameraHandler.HandleCameraRotation(delta, inputHandler.mouseX, inputHandler.mouseY);
        }
    }

    private void LateUpdate()
    {
        inputHandler.rollFlag = false;
        inputHandler.sprintFlag = false;
        isSprinting = inputHandler.b_Input;

        if (isInAir)
        {
            playerLocomotion.inAirTimer = playerLocomotion.inAirTimer + Time.deltaTime;
        }
    }
}

```   
</div>
</details> 