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
***

1. PlayerControls(InputSystem) 에 새로운 입력 할당
2. PlayerManager 스크립트 추가
3. 애니메이터에 애니메이션 추가
4. 여러 스크립트들에 내용 추가
5. 애니메이션에 ResetIsInteracting 스크립트를 Behavior로 추가
6. 영상의 방법으로 되지 않는 버그 수정

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

roFlag나 isInteracting의 변수들이 다른 곳에서 true가 되어 쓰이더라도
그 다음 프레임에서 여기서 false로 갱신을 해준다.


## 애니메이터에 애니메이션 추가

애니메이션과 파라미터 추가.
구르기의 경우 애니메이션 전환을 스크립트에서 설정해주기 때문에
따로 Conditions에 조건을 추가해주지 않음.

여기서 추가되는 isInteracting 파라미터는
구르기나 백스텝같은 애니메이션 고유의 움직임을 따라하야하는 애니메이션이 실행될 때 
true가 되는 변수다.


![image](https://github.com/novicehog/comments/assets/131991619/52affa5c-4e06-4cb7-a74c-73278526006c)



## 여러 스크립트들에 내용 추가

### AnimatorHandler 추가 내용
***
<details>
<summary>AnimatorHandler</summary>
<div markdown="1"> 

```c#

public class AnimatorHandler : MonoBehaviour
{
    public Animator anim;
    public InputHandler inputHandler;
    public PlayerLocomotion playerLocomotion;

    int vertical;
    int horizontal;
    public bool canRotate;

    public void Initialized()
    {
        ...
    }

    public void UpdateAnimatorValues(float verticalMovement, float horizontalMovement)
    {
        ...
    }

    public void PlayTargetAnimation(string targetAnim, bool isInteracting)
    {
        anim.applyRootMotion = isInteracting;
        anim.SetBool("isInteracting", isInteracting);
        anim.CrossFade(targetAnim, 0.1f, 0) ;
    }

    public void CanRotate()
    {
        canRotate = true;
    }
    public void StopRotate()
    {
        canRotate = false;
    }

    private void OnAnimatorMove()
    {
        if (inputHandler.isInteracting == false)
            return;

        float delta = Time.deltaTime;
        playerLocomotion.rigidbody.drag = 0;
        Vector3 deltaPosition = anim.deltaPosition;
        deltaPosition.y = 0;
        Vector3 velocity = deltaPosition / delta;
        playerLocomotion.rigidbody.velocity = velocity;

    }
}


```     
</div>
</details> 

#### PlayTargetAnimation 함수

이 부분은 외부 스크립트에서 애니메이션을 실행할 수 있도록 해주는 부분이다.



***
```c#
public void PlayTargetAnimation(string targetAnim, bool isInteracting)
{
    anim.applyRootMotion = isInteracting;           
    // isInteracting 파라미터 설정
    anim.SetBool("isInteracting", isInteracting);   
    // 애니메이션을 Fade In, Out 효과를 주며 부드럽게 실행
    anim.CrossFade(targetAnim, 0.1f, 0) ;           
}
```
<br>


#### OnAnimatorMove 유니티 이벤트
***

구르기에 경우 애니메이션 자체에 내장된 움직임을 이용하여야 자연스럽기 때문에
유니티에서 제공하는 `OnAnimatorMove`에 움직임을 구현한다.

OnAnimatorMove는 애니메이션의 rootPosition이 움직일 때 계속 호출된돠.
밑의 사진이 루트 포지션

![image](https://github.com/novicehog/comments/assets/131991619/f5982b30-0bb3-4da8-bb55-6e9b88bb045b)

<br>


이 이벤트의 경우 애니메이터의 rootmotion이 다음과 같이 `Handled by Script`로 바뀐다.

![image](https://github.com/novicehog/comments/assets/131991619/b0f79c43-3346-4aec-b2da-14e7b960487e)

<br>

```c#
private void OnAnimatorMove()
{
    // 
    if (inputHandler.isInteracting == false)
        return;

    float delta = Time.deltaTime;
    playerLocomotion.rigidbody.drag = 0;            // 마찰력 제거
    Vector3 deltaPosition = anim.deltaPosition;     // 거리 가져옴
    deltaPosition.y = 0;                            // y축 이동 제거
    Vector3 velocity = deltaPosition / delta;       // 거리 가져옴
    playerLocomotion.rigidbody.velocity = velocity; // 속도 적용
}
```

구현된 코드의 원리는 다음과 같다.
0. isInteracting 중이 아닐 경우 실행하지않음 `(isInteracting은 rootMotion을 통해 움직이겠다는 변수)`
1. 애니메이션 자체에 내장된 움직임을 가져온다.  (anim.deltaPosition로 가져옴)
2. 속력을 구하기 위해 거리를 시간으로 나눠준다.    (거리 / 시간 = 속도)
3. y축 속력 제거
4. 속력 적용
<br>



### InputHandler에 추가된 부분

다음 내용이 추가됐다.

```c#

public bool rollFlag;
public bool isInteracting;

private void HandleRollInput(float delta)
{
    //b_Input = inputActions.PlayerActions.Roll.phase == UnityEngine.InputSystem.InputActionPhase.Started;
    b_Input = inputActions.PlayerActions.Roll.triggered;

    if (b_Input)
    {
        rollFlag = true;
    }
}
```


### PlayerLocomotion에 추가된 부분
***

AnimatorHandler 스크립트에 추가된 함수를 조건을 판별하여
실행시킨다.


```c#
public void Update()
{
    ...

    HandleRollingAndSprinting(delta);
}

public void HandleRollingAndSprinting(float delta)
{
    // 한 번 실행하고 애니메이션이 끝날 때 까지 다시 실행 불가능
    if (animatorHandler.anim.GetBool("isInteracting"))
        return;

    if (inputHandler.rollFlag)
    {
        moveDirection = cameraObject.forward * inputHandler.vertical;
        moveDirection += cameraObject.right * inputHandler.horizontal;

        if (inputHandler.moveAmount > 0)
        {
            animatorHandler.PlayTargetAnimation("Rolling", true);
            moveDirection.y = 0;
            Quaternion rollRotation = Quaternion.LookRotation(moveDirection);
            myTransform.rotation = rollRotation;
        }
        else
        {
            animatorHandler.PlayTargetAnimation("Backstep", true);
        }
    }
}
```

<br>



## 애니메이션에 ResetIsInteracting 스크립트를 Behavior로 추가

애니메이션 Behavior에 다음과 같은 스크립트를 추가하여, <br>
애니메이션 종료시 자동으로 isInteracting 파라미터가 false가 되도록 한다.

이 스크립트를 추가함으로써 따로 isInteracting 

```c#
public class ResetIsInteracting : StateMachineBehaviour
{
    //OnStateExit is called when a transition ends and the state machine finishes evaluating this state
    override public void OnStateExit(Animator animator, AnimatorStateInfo stateInfo, int layerIndex)
    {
        animator.SetBool("isInteracting", false);
    }
}

```

<br>




## 영상의 방법으로 되지 않는 버그 수정

### InputSystem의 started
***

구르기의 입력버튼을 감지하는 부분이 영상대로 하면 제대로 되지 않음. <br>
버전 업의 이유로 되지 않는다 하며 다음과 같이 바꾸면 해결.

```c#
    //b_Input = inputActions.PlayerActions.Roll.phase == UnityEngine.InputSystem.InputActionPhase.Started;
    b_Input = inputActions.PlayerActions.Roll.triggered; // 이렇게 해야함
```


### 구르기 애니메이션 오작동
***
나의 경우 mixamo에서 애니메이션을 가져왔는데 구르기가 오작동하는 문제가 있었다.

rootmotion의 위치가 불안정 했기 때문에 문제가 발생했으나, <br>

이것은 애니메이션을 다음과 같이 설정해주고 apply 해주니 되었다.

![image](https://github.com/novicehog/comments/assets/131991619/e6bd9e74-b2e4-45cd-9e69-ccaeb2a2dd67)


## 배운점
- 애니메이션에 내장된 애니메이션을 통해 자연스럽게 움직이는 법을 알게되었다.
- 애니메이션에 Behavior 을 추가하여 사용하는 법을 알았다.
- InputSystem의 triggered를 통해 입력 여부를 받는 법을 알았다. 