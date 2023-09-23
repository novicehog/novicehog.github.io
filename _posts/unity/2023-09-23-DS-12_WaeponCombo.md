---
layout: single
title:  "유니티 다크소울 따라만들기 ch_12 무기 콤보"
categories: 
    - Unity
tag: [Unity, DarkSouls]
published: true

author_profile: true # 옆에뜨는 프로파일

search: true #검색해도 안뜨게함

date: 2023-09-22
last_modified_at: 2023-09-22

---

# 구현 개요
연속으로 공격하면 그에 맞는 공격이 나가게 되도록 구현한다.

# 구현 원리

간단한 구현 원리는 다음과 같다.

1. 마지막으로 한 공격을 저장함.
2. 공격 도중에 콤보 공격을 할 수 있는 시간을 애니메이션 이벤트를 통해 만듬
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

인풋핸들러 수정

애니메이터 핸들러 수정

애니메이션 추가와 이벤트 조정