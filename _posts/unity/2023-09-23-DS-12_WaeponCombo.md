---
layout: single
title:  "유니티 다크소울 따라만들기 ch_12 콤보 공격"
categories: 
    - Unity
tag: [Unity, DarkSouls]
published: true

author_profile: true # 옆에뜨는 프로파일

search: true #검색해도 안뜨게함

date: 2023-09-22
last_modified_at: 2023-09-22

---

이 글은 유튜브 **Sebastian Graves Create Dark Souls**를 보고 따라 만들면서 헷갈리는 부분을 정리한 글입니다.
{: .notice--warning}


# 구현 개요
연속으로 공격하면 그에 맞는 공격이 나가게 되도록 구현한다.

# 구현 원리

간단한 구현 원리는 다음과 같다.

1. 마지막으로 한 공격을 저장함.
2. 공격 도중에 콤보 공격을 할 수 있는 시간을 애니메이션 이벤트를 통해 만듬<br>
2-1. 공격 가능 시간은 애니메이션 이벤트를 통해 만들어준다.
3. 그 시간에 공격키를 한 번 더 누르면 마지막 공격 애니메이션을 통해서 다음 공격 애니메이션 재생됨.

# 추가된 부분

## 애니메이션 파라미터 추가

애니메이션에 새로운 파라미터를 만든다.

![image](https://github.com/novicehog/comments/assets/131991619/584d157c-e8ec-4b6e-8642-3bb67ff02bbd)

<br>

## PlayerAttacker 스크립트 내용 추가

마지막 공격을 저장하게 하고 연속 공격을 구현하는 함수를 추가한다.

```c#
public class PlayerAttacker : MonoBehaviour
{
    AnimatorHandler animatorHandler;
    InputHandler inputHandler;
    public string lastAttack;

    private void Awake()
    {
        animatorHandler = GetComponentInChildren<AnimatorHandler>();
        inputHandler = GetComponent<InputHandler>();
    }

    public void HandleWeaponCombo(WeaponItem weapon)
    {
        if(inputHandler.comboFlag)
        {
            animatorHandler.anim.SetBool("canDoCombo", false);
            if (lastAttack == weapon.OH_Light_Attack_01)
            {
                animatorHandler.PlayTargetAnimation(weapon.OH_Light_Attack_02, true);
            }
        }

    }

    public void HandleLightAttack(WeaponItem weapon)
    {
        animatorHandler.PlayTargetAnimation(weapon.OH_Light_Attack_01, true);
        lastAttack = weapon.OH_Light_Attack_01;
    }

    public void HandleHeavyAttack(WeaponItem weapon)
    {
        animatorHandler.PlayTargetAnimation(weapon.OH_Heavy_Attack_01, true);
        lastAttack = weapon.OH_Heavy_Attack_01;
    }
}
```

<br>


## PlayerManager 스크립트 내용 추가

파라미터 `candoCombe`의 값을 프레임마다 가져와 저장하는 변수를
만들어준다.

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
    public bool canDoCombo;         // 추가된 부분
    void Start()
    {
        inputHandler = GetComponent<InputHandler>();
        anim = GetComponentInChildren<Animator>();
        playerLocomotion = GetComponent<PlayerLocomotion>();
    }

    private void Awake()
    {
        cameraHandler = FindAnyObjectByType<CameraHandler>();
    }

    void Update()
    {
        float delta = Time.deltaTime;
        isInteracting = anim.GetBool("isInteracting");
        canDoCombo = anim.GetBool("canDoCombo");        // 추가된 부분

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
        inputHandler.rb_Input = false;
        inputHandler.rt_Input = false;

        if (isInAir)
        {
            playerLocomotion.inAirTimer = playerLocomotion.inAirTimer + Time.deltaTime;
        }
    }
}

```

<br>

## InputHandler 수정

원래는 특정 행동 중에는 공격이 불가능했으나 <br>
수정하여 공격 도중 `콤보공격이 가능한 시점`에는 `콤보공격이 가능`하게 수정되었다.


```c#
public class InputHandler : MonoBehaviour
{
    public float horizontal;
    public float vertical;
    public float moveAmount;
    public float mouseX;
    public float mouseY;

    public bool b_Input;
    public bool rb_Input;
    public bool rt_Input;

    public bool rollFlag;
    public bool sprintFlag;
    public bool comboFlag;          // 추가된 부분
    public float rollInputTimer;

    PlayerControls inputActions;
    PlayerAttacker playerAttacker;
    PlayerInventory playerInventory;
    PlayerManager playerManager;

    CameraHandler cameraHandler;

    Vector2 movementInput;
    Vector2 cameraInput;


    private void Awake()
    {
        playerAttacker = GetComponent<PlayerAttacker>();
        playerInventory = GetComponent<PlayerInventory>();
        playerManager = GetComponent<PlayerManager>();
    }


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
        HandleRollInput(delta);
        HandleAttackInput(delta);
    }

    private void MoveInput(float delta)
    {
        horizontal = movementInput.x;
        vertical = movementInput.y;
        moveAmount = Mathf.Clamp01(Mathf.Abs(horizontal) + Mathf.Abs(vertical));
        mouseX = cameraInput.x;
        mouseY = cameraInput.y;
    }

    private void HandleRollInput(float delta)
    {
        //b_Input = inputActions.PlayerActions.Roll.phase == UnityEngine.InputSystem.InputActionPhase.Started;
        //b_Input = inputActions.PlayerActions.Roll.triggered;
        // 눌릴 때만 true
        b_Input = inputActions.PlayerActions.Roll.IsPressed();

        if (b_Input)
        {
            rollInputTimer += delta;
            sprintFlag = true;
        }
        else
        {
            if (rollInputTimer > 0 && rollInputTimer < 0.5f)
            {
                sprintFlag = false;
                rollFlag = true;
            }

            rollInputTimer = 0;
        }
    }

    private void HandleAttackInput(float delta)
    {
        inputActions.PlayerActions.RB.performed += i => rb_Input = true;
        inputActions.PlayerActions.RT.performed += i => rt_Input = true;

        if (rb_Input)   // 수정된 부분
        {
            if (playerManager.canDoCombo)  // 콤보공격 가능 시점에는 입력시 콤보공격 실행
            {
                comboFlag = true;   // 중복 실행 방지
                playerAttacker.HandleWeaponCombo(playerInventory.rightWeapon);
                comboFlag = false;
            }
            else
            {
                // 특정 동작 중에는 공격 불가능
                if (playerManager.isInteracting)
                    return;
                playerAttacker.HandleLightAttack(playerInventory.rightWeapon);
            }
        }

        if (rt_Input)
        {
            playerAttacker.HandleHeavyAttack(playerInventory.rightWeapon);
        }
    }
}
```

<br>

## AnimatorHandler 수정

애니메이션 이벤트를 이용하여 콤보공격 가능 시점을 설정해준다.
그를 위해 파라미터 `canDocombo`를 끄거나 키는 간단한 함수를 만들어준다.

```c#
public class AnimatorHandler : MonoBehaviour
{
    private PlayerManager playerManager;
    public Animator anim;
    private InputHandler inputHandler;
    private PlayerLocomotion playerLocomotion;

    int vertical;
    int horizontal;
    public bool canRotate;

    public void Initialized()
    {
        playerManager = GetComponentInParent<PlayerManager>();
        anim = GetComponent<Animator>();
        inputHandler = GetComponentInParent<InputHandler>();
        playerLocomotion = GetComponentInParent<PlayerLocomotion>();
        vertical = Animator.StringToHash("Vertical");
        horizontal = Animator.StringToHash("Horizontal");
    }

    public void UpdateAnimatorValues(float verticalMovement, float horizontalMovement, bool isSprinting)
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

        if (isSprinting)
        {
            v = 2;
            h = horizontalMovement;
        }

        anim.SetFloat(vertical, v, 0.1f, Time.deltaTime);
        anim.SetFloat(horizontal, h, 0.1f, Time.deltaTime);
    }

    public void PlayTargetAnimation(string targetAnim, bool isInteracting)
    {
        anim.applyRootMotion = isInteracting;
        anim.SetBool("isInteracting", isInteracting);
        anim.CrossFade(targetAnim, 0.2f);
    }

    public void CanRotate()
    {
        canRotate = true;
    }
    public void StopRotate()
    {
        canRotate = false;
    }

    public void EnableCombo()       // 추가된 부분
    {
        anim.SetBool("canDoCombo", true);
    }

    public void DisableCombo()      // 추가된 부분
    {
        anim.SetBool("canDoCombo", false);
    }


    private void OnAnimatorMove()
    {
        if (playerManager.isInteracting == false)
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


## 애니메이션 이벤트 등록

애니메이션 핸들러를 통해 만든 EnableCombo(), DisableCombo()함수를 <br>
애니메이션 이벤트로 추가해준다.

`EnableCombo()`

<img width="803" alt="image" src="https://github.com/novicehog/comments/assets/131991619/674b905c-1872-4811-a2dc-5c396c285975">

<br>

`DisableCombo()`

<img width="776" alt="image" src="https://github.com/novicehog/comments/assets/131991619/42c217fa-1131-4b19-b58c-04141c80fbd1">


# 결과

![ezgif com-optimize (1)](https://github.com/novicehog/comments/assets/131991619/1a970e09-5a6b-4084-a64a-9170087a9ae7)



# 배운점
- 애니메이션 클립을 string으로 관리하는 예를 콤보공격을 통해 알 수 있었다.
- 콤보 공격의 구현 원리에 대해 알게 되었다.
 