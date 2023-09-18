---
layout: single
title:  "유니티 다크소울 따라만들기 ch_6 코드 정리"
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

# 코드 정리
이번장은 너무 난잡한 코드들을 정리하는 시간이다.

PlayerManager 스크립트에 모든 실행을 맡기고 다른 스크립트는 <br>
기능 구현만을 담당하도록 바꾸며<br>
코드를 늘 헷갈리게 만들었던 isInteracting 변수도 PlayerManager로 이동시킨다.

즉, `오직 playerManager만 Update를 사용`한다.


## Scripts 변경사항

### PlayerManager

이제 모든 코드의 중심은 PlayerManager가 담당한다. <br>
`다른 코드들은 전부 기능 구현만을 담당`하며 `실행은 이곳에서 전부 처리`한다. 

또한 update, FixedUpdate, LateUpdate로 Update를 분리하여
코드의 진행 순서가 좀 더 한눈에 들어오도록 바뀌었다.

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
    }
}

```
<br>


### 다른 코드들의 변경

다른 코드들의 변경점은 모두 동일하므로 따로 코드를 작성하지 않겠다.

1. Update 문이 있으면 그 부분을 전부 PlayerManager로 옮김
2. bool 변수인 isInteracting를 PlayerManager로 옮기며 isInteracting을 쓰던 부분은
    PlayerManager.isInteracting으로 대체




# 배운점
- 이번에 정리하는 영상을 통해 깔끔한 코드 정리 방법에 대해 알 수 있었다.