---
layout: single
title:  "유니티 다크소울 따라만들기 ch_1 플레이어 이동"
categories: 
    - Unity
tag: [Unity, DarkSouls]
published: true

author_profile: true # 옆에뜨는 프로파일

search: true #검색해도 안뜨게함

date: 2023-09-15
last_modified_at: 2023-09-15

---
이 글은 유튜브 **Sebastian Graves Create Dark Souls**를 보고 따라 만들면서 헷갈리는 부분을 정리한 글입니다.
{: .notice--warning}

# 초기 설정

## InputSystem 설정 

먼저 인풋 시스템을 패키지에서 설치

![image](https://github.com/novicehog/comments/assets/131991619/4d760224-515d-448b-bcb0-8fd7e031991b)

<br>

그 후 유니티를 재시작하고<br>
Create -> InputAction 을 `PlayerControls`이름으로 생성 <br>
그 뒤 아래 사진 처럼 C# 스크립트 생성을 Apply함
![image](https://github.com/novicehog/comments/assets/131991619/17e2797f-67b4-4587-be48-7efd74500ec7)

<br>
키보드 전용 설정값들을 등록해줌
![image](https://github.com/novicehog/comments/assets/131991619/f836a962-6aea-4b20-b193-0cce0bfa6f30)lp
<br>

카메라 설정값도 추가해줌
![image](https://github.com/novicehog/comments/assets/131991619/3f8c8156-82b7-4690-af33-3008ca4e04f8)

![image](https://github.com/novicehog/comments/assets/131991619/48050f04-90bc-460d-aadc-1ccc021f1619)

![image](https://github.com/novicehog/comments/assets/131991619/fcdd1b12-b068-44e6-bd41-30c719068108)
<br>

edit -> project Setting -> Input System Package에서 다음과 같이 해줌
![image](https://github.com/novicehog/comments/assets/131991619/346017b8-1e55-4752-9e93-7ae5a7918d3d)

## Scripts
스크립트는 총 세개의 스크립트가 있다.
**<span style="color:#44C9A1">InputHandler</span>** <br>
**<span style="color:#44C9A1">AnimatorHandler</span>**<br>
**<span style="color:#44C9A1">PlayerLocomotion</span>**<br>

### InputHandler
***
플레이어의 입력과 관련된 기능을 모아둔 스크립트이다.
<details>
<summary>InputHandler 스크립트</summary>
<div markdown="1">       
InputHandler의 스크립트는 다음과 같다.

```c#

public class InputHandler : MonoBehaviour
{
    public float horizontal;
    public float vertical;
    public float moveAmount;
    public float mouseX;
    public float mouseY;

    PlayerControls inputActions;

    Vector2 movementInput;      // 이동 관련 입력 값을 받는 변수
    Vector2 cameraInput;        // 카메라 관련 입력 값을 받는 변수

    public void OnEnable()
    {
        if (inputActions == null)
        {
            inputActions = new PlayerControls();
            inputActions.PlayerMovement.Movement.performed +=
                inputActions => movementInput = inputActions.ReadValue<Vector2>();
            inputActions.PlayerMovement.Camera.performed += i => cameraInput = i.ReadValue<Vector2>();
        }

        // 인풋 액션 활성화
        inputActions.Enable();
    }

    private void OnDisable()
    {
        inputActions.Disable();
    }

    public void TickInput(float delta)
    {
        MoveInput(delta);
    }

    private void MoveInput(float delta)
    {
        horizontal = movementInput.x;
        vertical = movementInput.y;
        moveAmount = Mathf.Clamp01(Mathf.Abs(horizontal) + Mathf.Abs(vertical));
        mouseX = cameraInput.x;
        mouseY = cameraInput.y;
    }
}
```

</div>
</details> 


#### inputAction 관련 부분
***

inputAction 관련해서 보면 다음과 같다.

```c#
PlayerControls inputActions; // 만들어 뒀던 InputAction

Vector2 movementInput;
Vector2 cameraInput;

public void OnEnable()
{
    if (inputActions == null)
    {
        inputActions = new PlayerControls();  // 1

        // 
        inputActions.PlayerMovement.Movement.performed +=  
            inputActions => movementInput = inputActions.ReadValue<Vector2>();
        inputActions.PlayerMovement.Camera.performed += 
            i => cameraInput = i.ReadValue<Vector2>();
    }

    // 인풋 액션 활성화
    inputActions.Enable();
}

private void OnDisable()
{
    inputActions.Disable();
}
```

<br>

**<span style="color:skyblue">1.</span>** 객체가 활성화 될 때 inputActions이 비어있다면 새롭게 만든다.<br>

**<span style="color:skyblue">2.</span>** PlayerControls에서 만들어 두었던 inputAction에 performed(입력이 되고 있다면 계속 호출)액션에 람다식을 통해
    이동 관련의 함수를 넣어준다.
![인풋액션경로](https://github.com/novicehog/comments/assets/131991619/3f07f88c-76e0-48f2-b916-28e3a81909bd)
<br>
람다식은 함수를 따로 선언하지 않고 익명의 함수를 만드는 방법으로 여기서 사용된 람다식을 
굳이 함수로 선언한다고 하면 다음과 같다고 볼 수 있다.

```c#
//inputActions.PlayerMovement.Movement.performed +=  
//inputActions => movementInput = inputActions.ReadValue<Vector2>();

// 위의 내용은 메소드를 선언하지 않는다는 것 외에는 아래와 같은 기능을 한다.
inputActions.PlayerMovement.Movement.performed += playerMove;
void playerMove(InputAction.CallbackContext inputActions)
{
    movementInput = inputActions.ReadValue<Vector2>();
}
```
<br>

**<span style="color:skyblue">3.</span>** inputActions.ReadValue<Vector2>();를 통해 입력 값을 받아온다.
이때 여기서 받아오는 값은 사전에 설정한 InputAction의 값과 일치해야한다.

![vector2](https://github.com/novicehog/comments/assets/131991619/9d676698-3664-4226-b17e-d9ad27f44f5c)

**<span style="color:skyblue">4.</span>** inputActions.Enable()으로 `inputActions을 활성화` 해준다.

**<span style="color:skyblue">4.</span>** inputActions.Disabl()을 통해 게임 오브젝트가 비활성화되면 입력값을 받지 않도록 한다.

#### 이동 관련 부분
***
inputAction을 통해 받은 입력 값을 토대로 이동 관련 로직이 구현된다.

`중요한 것`은 **<span style="color:#44C9A1">InputHandler</span>** 스크립트는 기능을 구현만을 담당하며
직접 이것들을 사용하는 것은 **<span style="color:#44C9A1">PlayerLocomotion</span>** 스크립트가 담당한다.

```c#
public float horizontal;
public float vertical;
public float moveAmount;
public float mouseX;
public float mouseY;



public void TickInput(float delta)
{
    MoveInput(delta);
}

private void MoveInput(float delta)
{
    horizontal = movementInput.x;
    vertical = movementInput.y;
    moveAmount = Mathf.Clamp01(Mathf.Abs(horizontal) + Mathf.Abs(vertical));
    mouseX = cameraInput.x;
    mouseY = cameraInput.y;
}
```

어려운 부분은 딱히 없는 부분이다.

**<span style="color:skyblue">1.</span>** MoveInput을 외부에서도 사용할 수 있도록 TickInput 함수로 묶는다.
**<span style="color:skyblue">2.</span>** MoveInput이 실행되면 변수들에 값을 저장한다.  


### AnimatorHandler
***
<details>
<summary>AnimatorHandler 스크립트</summary>
<div markdown="1">    

애니메이션과 관련된 부분을 담당하는 스크립트이다
```c#
public class AnimatorHandler : MonoBehaviour
{
    public Animator anim;
    int vertical;
    int horizontal;
    public bool canRotate;

    public void Initialized()
    {
        anim = GetComponent<Animator>();
        vertical = Animator.StringToHash("Vertical");
        horizontal = Animator.StringToHash("Horizontal");
    }

    public void UpdateAnimatorValues(float verticalMovement, float horizontalMovement)
    {
        #region vertical
        float v = 0;

        if (verticalMovement > 0 && verticalMovement < 0.55f)
        {
            v = 0.5f;
        }
        else if (verticalMovement > 0.55f)
        {
            v = 1;
        }
        else if (verticalMovement < 0 && verticalMovement > -0.55f)
        {
            v = -0.5f;
        }
        else if(verticalMovement < -0.55f)
        {
            v = -1;
        }
        else
        {
            v = 0;
        }
        #endregion
        #region horizontal

        float h = 0;

        if (horizontalMovement > 0 && horizontalMovement < 0.55f)
        {
            h = 0.5f;
        }
        else if (horizontalMovement > 0.55f)
        {
            h = 1;
        }
        else if( horizontalMovement < 0 &&  horizontalMovement > -0.55f)
        {
            h = -0.5f;
        }
        else if ( horizontalMovement < -0.55f)
        {
            h = -1;
        }
        else
        {
            h = 0; 
        }
        #endregion

        anim.SetFloat(vertical, v, 0.1f, Time.deltaTime);
        anim.SetFloat(horizontal, h, 0.1f, Time.deltaTime);
    }

    public void CanRotate()
    {
        canRotate = true;
    }
    public void StopRotate()
    {
        canRotate = false;
    }

}
```

</div>
</details> 

#### 초기화 부분
***
이 스크립트는 플레이어의 자식인 모델에 붙어 있기 때문에
초기화의 타이밍을 통일 시켜주기위해 초기화 함수를 만들고 실행을 **<span style="color:#44C9A1">PlayerLocomotion</span>** 에서 함께 초기화한다.

초기화의 내용은 애니메이터 컴포넌트를 저장하고
좀더 나은 성능을 위해서 애니메이터 파라미터들을 해쉬로 미리 저장해두는 작업을 한다.
```c#
public void Initialized()
{
    anim = GetComponent<Animator>();
    vertical = Animator.StringToHash("Vertical");
    horizontal = Animator.StringToHash("Horizontal");
}
```

#### 애니메이션 업데이트 부분
***
코드가 세로로 긴 이유로 여기에 또 적진 않겠다.
내용은 간단하게 매개변수로 받은 값에 따라 애니메이션을 업데이트 해준다.
물론 이것도 기능만 구현되어 있고 실제 실행은  **<span style="color:#44C9A1">PlayerLocomotion</span>** 스크립트가 한다.


#### 스크립트 외에 필요한 것
애니메이터의 파라미터와 블렌드 트리가 설정되어 있어야 한다.
![image](https://github.com/novicehog/comments/assets/131991619/c8fd1904-02ab-4c51-9a82-c0eac48425e4)


### PlayerLocomotion
***
플레이어의 모든 행동을 담당한다.
기능들을 직접 구현하기 보다는 다른 스크립트들을 
이용해서 한 번에 사용하는 느낌이 있다.

details>
<summary>AnimatorHandler 스크립트</summary>
<div markdown="1">    

애니메이션과 관련된 부분을 담당하는 스크립트이다

```c#

public class PlayerLocomotion : MonoBehaviour
{
    Transform cameraObject;
    InputHandler inputHandler;
    Vector3 moveDirection;


    [HideInInspector]
    public Transform myTransform;
    [HideInInspector]
    public AnimatorHandler animatorHandler;

    public new Rigidbody rigidbody;
    public GameObject normalCamera;

    [Header("Status")]
    [SerializeField] float movementSpeed = 5;
    [SerializeField] float rotationSpeed = 10;

    void Start()
    {
        rigidbody = GetComponent<Rigidbody>();
        inputHandler = GetComponent<InputHandler>();
        animatorHandler = GetComponentInChildren<AnimatorHandler>();
        cameraObject = Camera.main.transform;
        myTransform = transform;
        animatorHandler.Initialized();

    }

    #region Movement
    Vector3 normalVector;
    Vector3 targetPosition;

    private void HandleRotation(float delta)
    {
        // 정면의 방향을 계산함
        Vector3 targetDir = Vector3.zero;
        float moveOverride = inputHandler.moveAmount;

        targetDir = cameraObject.forward * inputHandler.vertical;
        targetDir += cameraObject.right * inputHandler.horizontal;

        targetDir.Normalize();
        targetDir.y = 0;

        if (targetDir == Vector3.zero)  // 움직이지 않을떄는 마지막으로 보던 곳을 보게 함
            targetDir = myTransform.forward;



        float rs = rotationSpeed;

        Quaternion tr = Quaternion.LookRotation(targetDir);
        Quaternion targetRotation = Quaternion.Slerp(myTransform.rotation, tr, rs * delta);

        myTransform.rotation = targetRotation;
    }
    #endregion
    public void Update()
    {
        float delta = Time.deltaTime;

        inputHandler.TickInput(delta);

        moveDirection = cameraObject.forward * inputHandler.vertical;
        moveDirection += cameraObject.right * inputHandler.horizontal;
        moveDirection.Normalize();

        float speed = movementSpeed;
        moveDirection *= speed;

        Vector3 projectedVelocity = Vector3.ProjectOnPlane(moveDirection, normalVector);
        rigidbody.velocity = projectedVelocity;

        animatorHandler.UpdateAnimatorValues(inputHandler.moveAmount, 0);

        if (animatorHandler.canRotate)
        {
            HandleRotation(delta);
        }
    }
}

```

</div>
</details> 


#### 회전 관련 함수
***

캐릭터의 바라보는 방향을 설정하는 부분
이 부분은 간단하니 주석으로만 적어놓음.


```c#

private void HandleRotation(float delta)
{
    // 정면의 방향을 계산함
    Vector3 targetDir = Vector3.zero;
    float moveOverride = inputHandler.moveAmount;

    targetDir = cameraObject.forward * inputHandler.vertical;
    targetDir += cameraObject.right * inputHandler.horizontal;

    targetDir.Normalize();
    targetDir.y = 0;

    if (targetDir == Vector3.zero)  // 움직이지 않을떄는 캐릭터의 현재 정면을 정면으로함
        targetDir = myTransform.forward;



    // 정면 방향을 향해 부드럽게 바라보게 함
    float rs = rotationSpeed;

    Quaternion tr = Quaternion.LookRotation(targetDir);
    Quaternion targetRotation = Quaternion.Slerp(myTransform.rotation, tr, rs * delta);

    myTransform.rotation = targetRotation;
}

```
<br>
#### 이동 관련 부분
***

위에서 봤던 회전 관련 함수와 매우 유사한 방식으로 처리됨.

```c#
public void Update()
{
    float delta = Time.deltaTime;

    // InputHandler의 변수들의 값을 갱신해줌
    inputHandler.TickInput(delta);

    moveDirection = cameraObject.forward * inputHandler.vertical;
    moveDirection += cameraObject.right * inputHandler.horizontal;
    moveDirection.Normalize();

    float speed = movementSpeed;
    moveDirection *= speed;

    // 현재 이동방향을 (1번째 인자)를 면의 노벌벡터(2번쨰 인자)를 통해 면을 알아내어 투영시킨 방향으로 설정함
    // 즉 면을 따라 이동방향이 설정됨, 하지만 여기선 아직 normalVector가 설정되지 않아 의미없음
    Vector3 projectedVelocity = Vector3.ProjectOnPlane(moveDirection, normalVector);
    rigidbody.velocity = projectedVelocity;

    // 현재 이동 정도에 따라 애니메이션을 결정
    animatorHandler.UpdateAnimatorValues(inputHandler.moveAmount, 0);

    // 애니메이터 핸들러에 만들어둔 변수를 통해 
    // 캐릭터 회전을 할지 말지를 정함
    if (animatorHandler.canRotate)
    {
        HandleRotation(delta);
    }
}

```

<br>

## 배운점
1. 스크립트들을 총괄하는 하나의 스크립트를 만들고 `(PlayerLocomotion)` 다른 스크립트들은
   기능만 구현해 두어 한 곳에서 처리하는 작성 방법이 매우 인상적이였다.<br>
   객체지향언어의 코드 작성법에 대해 모르는 투성이였는데 어떤 식으로 되는지 볼 수 있었다. <br>
2. InputSystem과 action , 람다식 등 delegate와 관련된 부분에 대해 조금더 이해할 수 있었다. <br>



