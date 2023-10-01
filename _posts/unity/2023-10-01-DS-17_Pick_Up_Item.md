---
layout: single
title:  "유니티 다크소울 따라만들기 ch_17 아이템"
categories: 
    - Unity
tag: [Unity, DarkSouls]
published: true

author_profile: true # 옆에뜨는 프로파일

search: true #검색해도 안뜨게함

date: 2023-10-01
last_modified_at: 2023-10-01

---

이 글은 유튜브 **Sebastian Graves Create Dark Souls**를 보고 따라 만들면서 헷갈리는 부분을 정리한 글입니다.
{: .notice--warning}

# 구현 개요

다크소울의 아이템 줍기를 구현한다. 여기서는 무기 아이템 줍기만을 구현한다.

다크소울의 아이템 줍기는 다음과 같음.

![ezgif com-optimize](https://github.com/novicehog/comments/assets/131991619/40522541-e18d-4f17-9a0f-7c4163bf18c1)

<br>

1. 닿으면 상호작용이 활성화.
2. 상호작용 시 애니메이션 실행.
3. 아이템 획득

<br>

# 구현 시작

## inputSystem 추가

상호작용 키를 할당해준다.

![image](https://github.com/novicehog/comments/assets/131991619/02ab6040-2193-4d84-a4ea-6f9b3b02bb9d)

<br>

## 애니메이션 추가

줍는 애니메이션을 애니메이터에 추가해준다.

![image](https://github.com/novicehog/comments/assets/131991619/2bd75af8-f681-4aca-8473-84c747b21890)

## PlayerInventoryScript 수정

아이템이 담길 `인벤토리`를 리스트로 추가해준다.<br>
지금은 무기 인벤토리만 만든다.

```c#
public List<WeaponItem> weaponsInventory;
```

## Interactable 스크립트 추가

실제로 맵에 떨어진 아이템과 데이터상의 아이템은 다른 것이다.

데이터상의 무기 아이템은 이미 있으니 이번에는 맵에 떨어져있는 아이템이 가질 스크립트를 만든다.

Interactable 스크립트는 상호작용 가능한 모든 아이템이 공통으로 가질 특징들을 모아놓은 부모 클래스이다.

```c#

public class Interactable : MonoBehaviour
{
    public float radius = 0.6f;
    public string interactableText;
    private void OnDrawGizmosSelected()
    {
        Gizmos.color = Color.blue;
        Gizmos.DrawWireSphere(transform.position, radius);
    }

    public virtual void Interact(PlayerManager playerManager)
    {
        Debug.Log("You interacted with an object!");
    }
}

```

<br>

## WeaponPickUp 스크립트 추가

Interactable을 상속받는 필드 드랍 무기아이템을 위한 스크립트를 작성한다.

Interact를 오버라이드 하여 아이템을 줍는 함수를 구현해서 추가해준다.

```c#
public class WeaponPickUp : Interactable
{
    public WeaponItem weapon;

    public override void Interact(PlayerManager playerManager)
    {
        base.Interact(playerManager);

        PickUpItem(playerManager);
    }

    private void PickUpItem(PlayerManager playerManager)
    {
        PlayerInventory playerInventory;
        PlayerLocomotion playerLocomotion;
        AnimatorHandler animatorHandler;

        playerInventory = playerManager.GetComponent<PlayerInventory>();
        playerLocomotion = playerManager.GetComponent<PlayerLocomotion>();
        animatorHandler = playerManager.GetComponentInChildren<AnimatorHandler>();

        playerLocomotion.rigidbody.velocity = Vector3.zero;         
        animatorHandler.PlayTargetAnimation("Pick Up Item", true);
        playerInventory.weaponsInventory.Add(weapon);
        Destroy(gameObject);
    }
}
```

<br>

## InputHandler 스크립트 수정

다음과 같이 입력을 받는 함수를 추가해준다.

```c#
public bool a_Input;

...

public void TickInput(float delta)
{
    ...
    HandleInteractingButtonInput();
}

private void HandleInteractingButtonInput()
{
    inputActions.PlayerActions.A.performed += inputActions => a_Input = true;
}
```

<br>

## PlayerManager 스크립트 수정

아이템과 충돌될 떄만 상호작용이 가능하도록 스크립트를 작성한다.

```c#
void Update()
{
    ...
    CheckForInteractableObject();
}
private void LateUpdate()
{
    ...
    inputHandler.a_Input = false;
    ...
}
public void CheckForInteractableObject()
{
    RaycastHit hit;

    if (Physics.SphereCast(transform.position, 0.3f, transform.forward, out hit, 1f, cameraHandler.ignoreLayers))
    {
        if (hit.collider.tag == "Interactable")
        {
            Interactable interactableObject = hit.collider.GetComponent<Interactable>();

            if (interactableObject != null)
            {
                string interactableText = interactableObject.interactableText;
                // UI에 표시될 내용
                //print(1);

                if (inputHandler.a_Input)
                {
                    hit.collider.GetComponent<Interactable>().Interact(this);
                }
            }
        }
    }
}
```

<br>

## 아이템 추가

아이템을 다음과 같이 추가해줌

![image](https://github.com/novicehog/comments/assets/131991619/a3b232b9-3992-4fe8-b1e4-3dc1e678311c)


<br>

# 결과

![ezgif com-video-to-gif](https://github.com/novicehog/comments/assets/131991619/89e31941-b34f-49dd-a652-1379e2473f8a)

<br>

![image](https://github.com/novicehog/comments/assets/131991619/028b8d09-8119-4d94-97d2-c5e06ece0155)
