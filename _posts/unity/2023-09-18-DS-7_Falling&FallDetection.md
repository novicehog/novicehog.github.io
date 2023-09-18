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

## 구현 내용
대략적인 구현 내용은 다음과 같다.

- 레이케스트를 통해서 바닥에 닿아있는지를 계속 체크한다 (레이케스트는 길지 않아 바로 아래 바닥만 감지가능).

![image](https://github.com/novicehog/comments/assets/131991619/3867384e-f1f9-4935-b5d3-7ed6ba685119)
<br>

- 바닥이 감지될 경우 그 바닥의 위치로 플레이어의 위치를 설정한다. (플레이어 높이를 바닥에 맞춰줌)

![image](https://github.com/novicehog/comments/assets/131991619/af79e423-3406-4415-a7ba-d9c607945a4b)
<br>

- 바닥이 감지되지 않을 경우 떨어진다.
  이때 아래로 떨어지며 떨어지기 직전의 이동방향으로도 조금씩 이동한다. (떨어지면서 이동 방향 변경은 불가능)

  ![image](https://github.com/novicehog/comments/assets/131991619/9a1bb73b-56d4-4422-b40f-13248ec36f0b)
<br>

- 떨어지는 시간이 0.5초 이상일 경우 착지 애니메이션 실행

![image](https://github.com/novicehog/comments/assets/131991619/24c5dd85-454c-40fc-bd18-f6977b44aa09)
<br>

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

#### 추가된 부분
***

isInAir 변수가 추가됐으며, <br>
떨어진지 얼마나 지났는지를 나타내는 PlayerLocomotion의 inAirTimer 변수를 갱신해준다

```c#
public bool isInAir;

private void LateUpdate()
{
    ...
    if (isInAir)
    {
        playerLocomotion.inAirTimer = playerLocomotion.inAirTimer + Time.deltaTime;
    }
}
```
<br>



<details>
<summary>PlayerLocomotion</summary>
<div markdown="1">       


### PlayerLocomotion
***
```c#

public class PlayerLocomotion : MonoBehaviour
{
    private PlayerManager playerManager;
    Transform cameraObject;
    InputHandler inputHandler;
    public Vector3 moveDirection;


    [HideInInspector]
    public Transform myTransform;
    [HideInInspector]
    public AnimatorHandler animatorHandler;

    public new Rigidbody rigidbody;
    public GameObject normalCamera;

    [Header("Ground & Air Detection Stats")]
    [SerializeField] private float groundDetectionRayStartPoint = 0.5f;
    [SerializeField] private float minimumDistanceNeededToBeginFall = 1f;
    [SerializeField]
    private float groundDirectionRayDistance = 0.2f;
    private LayerMask ignoreForGroundCheck;
    public float inAirTimer;

    [Header("Movement Status")]
    [SerializeField] private float movementSpeed = 5;
    [SerializeField] private float sprintSpeed = 7;
    [SerializeField] private float rotationSpeed = 10;
    [SerializeField] private float fallingSpeed = 45;


    void Start()
    {
        playerManager = GetComponent<PlayerManager>();
        rigidbody = GetComponent<Rigidbody>();
        inputHandler = GetComponent<InputHandler>();
        animatorHandler = GetComponentInChildren<AnimatorHandler>();
        cameraObject = Camera.main.transform;
        myTransform = transform;
        animatorHandler.Initialized();


        playerManager.isGrounded = true;
        ignoreForGroundCheck = ~(1 << 8 | 1 << 11);
    }




    #region Movement
    Vector3 normalVector;
    Vector3 targetPosition;

    private void HandleRotation(float delta)
    {
        Vector3 targetDir = Vector3.zero;
        float moveOverride = inputHandler.moveAmount;

        targetDir = cameraObject.forward * inputHandler.vertical;
        targetDir += cameraObject.right * inputHandler.horizontal;

        targetDir.Normalize();
        targetDir.y = 0;

        if (targetDir == Vector3.zero)
            targetDir = myTransform.forward;

        float rs = rotationSpeed;

        Quaternion tr = Quaternion.LookRotation(targetDir);
        Quaternion targetRotation = Quaternion.Slerp(myTransform.rotation, tr, rs * delta);

        myTransform.rotation = targetRotation;
    }
    public void HandleMovement(float delta)
    {
        // 특정 애니메이션 도중 이동 x
        if (inputHandler.rollFlag || playerManager.isInteracting)
            return;

        moveDirection = cameraObject.forward * inputHandler.vertical;
        moveDirection += cameraObject.right * inputHandler.horizontal;
        moveDirection.Normalize();
        moveDirection.y = 0;

        float speed = movementSpeed;


        if (inputHandler.sprintFlag)
        {
            speed = sprintSpeed;
            playerManager.isSprinting = true;
            moveDirection *= speed;
        }
        else
        {
            moveDirection *= speed;
        }

        Vector3 projectedVelocity = Vector3.ProjectOnPlane(moveDirection, normalVector);
        rigidbody.velocity = projectedVelocity;

        animatorHandler.UpdateAnimatorValues(inputHandler.moveAmount, 0, playerManager.isSprinting);

        if (animatorHandler.canRotate)
        {
            HandleRotation(delta);
        }
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
                animatorHandler.PlayTargetAnimation("BackStep", true);
            }
        }
    }

    public void HandleFalling(float delta, Vector3 moveDirection)
    {
        playerManager.isGrounded = false;
        RaycastHit hit;
        Vector3 origin = myTransform.position;
        origin.y += groundDetectionRayStartPoint;

        // 앞에 벽이 있으면 못감
        if (Physics.Raycast(origin, myTransform.forward, out hit, 0.4f))
        {
            moveDirection = Vector3.zero;
        }

        if (playerManager.isInAir)
        {
            rigidbody.AddForce(-Vector3.up * fallingSpeed);
            rigidbody.AddForce(moveDirection * fallingSpeed / 7f);
        }

        //Vector3 dir = moveDirection;
        //dir.Normalize();
        //origin = origin + dir * groundDirectionRayDistance;

        targetPosition = myTransform.position;

        Debug.DrawRay(origin, -Vector3.up * minimumDistanceNeededToBeginFall, Color.red, 0.1f, false);

        // 발 아래 땅이 있을 때
        if (Physics.Raycast(origin, -Vector3.up, out hit, minimumDistanceNeededToBeginFall, ignoreForGroundCheck))
        {
            normalVector = hit.normal;
            Vector3 tp = hit.point;
            playerManager.isGrounded = true;
            targetPosition.y = tp.y;

            // 공중에서 낙하도중 땋에 닿은 것이라면
            if (playerManager.isInAir)
            {
                // 많이 떨어지던 중 땋에 닿음 -> 착지 애니메이션 실행
                if (inAirTimer > 0.5f)
                {
                    Debug.Log("You were in the air for " + inAirTimer);
                    animatorHandler.PlayTargetAnimation("Land", true);
                    inAirTimer = 0;
                }
                // 그냥 조그만 난간에서 떨어짐 -> 이동 애니메이션
                else
                {
                    animatorHandler.PlayTargetAnimation("Locomotion", false);
                    inAirTimer = 0;
                }
                playerManager.isInAir = false;
            }
        }
        // 공중에 떠있을 경우
        else
        {
            // 땅에 있음 변수를 false로 바꿈
            if (playerManager.isGrounded)
            {
                playerManager.isGrounded = false;
            }
            // 공중으로 떨어지기 시작할 때 첫 한번 실행
            if (playerManager.isInAir == false)
            {
                // 다른 애니메이션 실행중이 아니라면 낙하 애니메이션 실행
                if (playerManager.isInteracting == false)
                {
                    animatorHandler.PlayTargetAnimation("Falling", true);
                }

                // 가던 방향으로 살짝 느리게 감
                Vector3 vel = rigidbody.velocity;
                vel.Normalize();
                rigidbody.velocity = vel * (movementSpeed / 2);
                playerManager.isInAir = true;
            }


        }

        // 땅에 있을 때 발의 위치를 재조정함
        if (playerManager.isGrounded)
        {
            if (playerManager.isInteracting || inputHandler.moveAmount > 0)
            {
                myTransform.position = Vector3.Lerp(myTransform.position, targetPosition, Time.deltaTime);
            }
            else
            {
                myTransform.position = targetPosition;
            }
        }

    }

    #endregion
}

```

</div>
</details> 

#### 추가된 부분

HandleFalling 함수와 몇가지 변수가 추가됐다.

```c#
[Header("Ground & Air Detection Stats")]
// 레이케스트가 시작되는 좌표와 플레이어 좌표가 가질 간격
[SerializeField] private float groundDetectionRayStartPoint = 0.5f;
// 낙하가 되는 높이의 최소거리 (레이캐스트의 길이)
[SerializeField] private float minimumDistanceNeededToBeginFall = 1f;
//버그가 있어서 비활성화 해둠
//[SerializeField] private float groundDirectionRayDistance = 0.2f;
private LayerMask ignoreForGroundCheck;
public float inAirTimer;

public void HandleFalling(float delta, Vector3 moveDirection)
{
    playerManager.isGrounded = false;
    RaycastHit hit;
    Vector3 origin = myTransform.position;
    origin.y += groundDetectionRayStartPoint;

    // 앞에 벽이 있으면 못감
    if (Physics.Raycast(origin, myTransform.forward, out hit, 0.4f))
    {
        moveDirection = Vector3.zero;
    }

    if (playerManager.isInAir)
    {
        rigidbody.AddForce(-Vector3.up * fallingSpeed);
        rigidbody.AddForce(moveDirection * fallingSpeed / 7f);
    }

    //Vector3 dir = moveDirection;
    //dir.Normalize();
    //origin = origin + dir * groundDirectionRayDistance;

    targetPosition = myTransform.position;

    Debug.DrawRay(origin, -Vector3.up * minimumDistanceNeededToBeginFall, Color.red, 0.1f, false);

    // 발 아래 땅이 있을 때
    if (Physics.Raycast(origin, -Vector3.up, out hit, minimumDistanceNeededToBeginFall, ignoreForGroundCheck))
    {
        normalVector = hit.normal;
        Vector3 tp = hit.point;
        playerManager.isGrounded = true;
        targetPosition.y = tp.y;        // 플레이어가 위치해야할 좌표를 저장해둠

        // 공중에서 낙하도중 땋에 닿은 것이라면
        if (playerManager.isInAir)
        {
            // 많이 떨어지던 중 땋에 닿음 -> 착지 애니메이션 실행
            if (inAirTimer > 0.5f)
            {
                Debug.Log("You were in the air for " + inAirTimer);
                animatorHandler.PlayTargetAnimation("Land", true);
                inAirTimer = 0;
            }
            // 그냥 조그만 난간에서 떨어짐 -> 이동 애니메이션
            else
            {
                animatorHandler.PlayTargetAnimation("Locomotion", false);
                inAirTimer = 0;
            }
            playerManager.isInAir = false;
        }
    }
    // 공중에 떠있을 경우
    else
    {
        // 땅에 있음 변수를 false로 바꿈
        if (playerManager.isGrounded)
        {
            playerManager.isGrounded = false;
        }
        // 공중으로 떨어지기 시작할 때 첫 한번 실행
        if (playerManager.isInAir == false)
        {
            // 다른 애니메이션 실행중이 아니라면 낙하 애니메이션 실행
            if (playerManager.isInteracting == false)
            {
                animatorHandler.PlayTargetAnimation("Falling", true);
            }

            // 가던 방향으로 살짝 느리게 감
            Vector3 vel = rigidbody.velocity;
            vel.Normalize();
            rigidbody.velocity = vel * (movementSpeed / 2);
            playerManager.isInAir = true;
        }


    }

    // 땅에 있을 때 발의 위치를 재조정함
    if (playerManager.isGrounded)
    {
        if (playerManager.isInteracting || inputHandler.moveAmount > 0)
        {
            myTransform.position = Vector3.Lerp(myTransform.position, targetPosition, Time.deltaTime);
        }
        else
        {
            myTransform.position = targetPosition;
        }
    }
}


```
<br>


#### Raycast를 통해 땅감지가 됨
***

땅을 다음과 같이 감지한다.

```c#
if (Physics.Raycast(origin, -Vector3.up, out hit, minimumDistanceNeededToBeginFall, ignoreForGroundCheck))
```


감지될 경우 착지 상태로 바꾸고 땅의 위치를 저장해둔다.

```c#
normalVector = hit.normal;
Vector3 tp = hit.point;
playerManager.isGrounded = true;
targetPosition.y = tp.y;
```
<br>

그 다음 착지 애니메이션을 재생할지 여부를 계산한다.

```c#
// 공중에서 낙하도중 땋에 닿은 것이라면
if (playerManager.isInAir)
{
    // 많이 떨어지던 중 땋에 닿음 -> 착지 애니메이션 실행
    if (inAirTimer > 0.5f)
    {
        Debug.Log("You were in the air for " + inAirTimer);
        animatorHandler.PlayTargetAnimation("Land", true);
        inAirTimer = 0;
    }
    // 그냥 조그만 난간에서 떨어짐 -> 이동 애니메이션
    else
    {
        animatorHandler.PlayTargetAnimation("Locomotion" , false);
        inAirTimer = 0;
    }
    playerManager.isInAir = false;
}
```


#### 땅 감지가 안됨
***

이 경우엔 땅에 있다가 떨어진 경우 낙하 애니메이션을 실행

```c#
// 공중에 떠있을 경우
else
{
    // 땅에 있음 변수를 false로 바꿈
    if (playerManager.isGrounded)
    {
        playerManager.isGrounded = false;
    }
    // 공중으로 떨어지기 시작할 때 첫 한번 실행
    if (playerManager.isInAir == false)
    {
        // 다른 애니메이션 실행중이 아니라면 낙하 애니메이션 실행
        if (playerManager.isInteracting == false)
        {
            animatorHandler.PlayTargetAnimation("Falling", true);
        }

        // 가던 방향으로 살짝 느리게 감
        Vector3 vel = rigidbody.velocity;
        vel.Normalize();
        rigidbody.velocity = vel * (movementSpeed / 2);
        playerManager.isInAir = true;
    }
}
```


#### 플레이어 높이를 땅과 맞춤
***

아까 저장해둔 땅의 위치로 플레이어를 자연스럽게 이동시킴

```c#
// 땅에 있을 때 발의 위치를 재조정함
if(playerManager.isGrounded)
{
    if(playerManager.isInteracting || inputHandler.moveAmount > 0)
    {
        myTransform.position = Vector3.Lerp(myTransform.position, targetPosition,Time.deltaTime);
    }
    else
    {
        myTransform.position = targetPosition;
    }
}          
                
```
