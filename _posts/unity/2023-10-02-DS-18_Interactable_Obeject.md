---
layout: single
title:  "유니티 다크소울 따라만들기 ch_18 아이템"
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

아이템 줍기의 연장선으로 상호작용 UI를 만든다.

1. 아이템과 상호작용이 가능하면 UI창이 뜨게함.
2. 상호작용 뒤에 따로 UI를 다시 띄워줌.


# 구현 시작

## UI 추가

Pop Up 창 두개를 만들어줌.

아이템에 닿았을 때 나오는 팝업창과, <br>
아이템과 상호작용 한 뒤 나오는 팝업창을 만든다.

<img width="347" alt="image" src="https://github.com/novicehog/comments/assets/131991619/6763928f-7115-4de9-8114-34dc4350e405">

## InteractableUI 스크립트 추가

상호작용 UI를 위한 스크립트를 만들어준다.

```c#
public class InteractableUI : MonoBehaviour
{
    public Text interactableText;   // 닿을 때 나오는 팝업창의 자식 Text

    public Text itemText;       // 상호작용 후 나오는 팝업창의 자식 Text
    public RawImage itemImage;  // 상호작용 후 나오는 팝업창의 아이템 RawImage
}
```

<br>

![image](https://github.com/novicehog/comments/assets/131991619/bd189be8-8499-4df1-88f9-f93d860ba3f3)


<br>


## PlayerManager 수정

UI 정보들을 가져와서 기능을 구현한다.

아이템과 닿아있을 때 팝업을 띄우고 상호작용키를 누르면 아이템 팝업을 띄운다.

아이템 팝업은 다시 한번 키를 눌러서 닫을 수 있다.

```c#
public class PlayerManager : MonoBehaviour
{
    ...

    InteractableUI interactableUI;
    public GameObject interactableUIGameObject;
    public GameObject itemInteractableGameObject;

    ...

    void Start()
    {
        ...
        interactableUI = FindAnyObjectByType<InteractableUI>();
    }

    public void CheckForInteractableObject()
    {
        RaycastHit hit;

        if (Physics.SphereCast(transform.position, 0.3f, transform.forward, out hit, 1f , cameraHandler.ignoreLayers))
        {
            if (hit.collider.tag == "Interactable")
            {
                Interactable interactableObject = hit.collider.GetComponent<Interactable>();

                if(interactableObject != null)
                {
                    string interactableText = interactableObject.interactableText;
                    interactableUI.interactableText.text = interactableText;
                    interactableUIGameObject.SetActive(true);

                    if (inputHandler.a_Input)
                    {
                        hit.collider.GetComponent<Interactable>().Interact(this);
                    }
                }
            }
        }
        else
        {
            if(interactableUIGameObject != null)
            {
                interactableUIGameObject.SetActive(false);
            }

            if(itemInteractableGameObject != null && inputHandler.a_Input)
            {
                itemInteractableGameObject.SetActive(false); 
            }
        }
    }
}
```

<br>


## WeaponPickUp 스크립트 수정

상호작용시 아이템 팝업의 텍스트와 이미지를 상호작용한 아이템 정보로 바꿔준다.

```c#
public class WeaponPickUp : Interactable
{
    ...

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
        // 추가된 부분
        playerManager.itemInteractableGameObject.GetComponentInChildren<Text>().text = weapon.itemName;                 
        playerManager.itemInteractableGameObject.GetComponentInChildren<RawImage>().texture = weapon.itemIcon.texture;  
        playerManager.itemInteractableGameObject.SetActive(true);     
        // ------                                                  
        Destroy(gameObject);
    }
}
```

<br>

# 결과

![ezgif com-optimize](https://github.com/novicehog/comments/assets/131991619/496652cf-88c3-4415-8241-2a046a5fa533)

<br>

# 배운점
- UI에 대해 좀 더 알아내는 시간이었다.
