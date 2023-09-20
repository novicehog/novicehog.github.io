---
layout: single
title:  "유니티 다크소울 따라만들기 ch_8 무기 슬롯"
categories: 
    - Unity
tag: [Unity, DarkSouls]
published: true

author_profile: true # 옆에뜨는 프로파일

search: true #검색해도 안뜨게함

date: 2023-09-19
last_modified_at: 2023-09-20

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
무기를 손에 들고 있는 것을 만든다.

## 구현 내용
대략적인 구현 내용은 다음과 같다.

1. `Item` 클래스와 그것을 상속받는 `WeaponItem` 클래스를 만든다
2. `양 손에 무기를 생성`하여 자식으로 넣거나 `장착 중인 무기를 파괴`하여 안보이게하는<br>
   기능이 구현된 `WeaponHolderSlot` 스크립트를 만들어 넣는다.
3. 양 손에 장착된 `WeaponHolderSlot스크립트를 관리`하는 `WeaponSlotManager` 스크립트를 만든다.
4. `PlyaerInventory` 스크립트를 하나 더 만들어 `WeaponSlotManager 스크립트와 상호작용`한다.


# item 세팅

## Item 스크립트
모든 아이템 클래스의 부모가 될 Item 스크립트를 만들어준다.

Item 를 상속받는 모든 클래스는 공통으로 아이콘과 이름을 가질 것이다.

아이템 스크립트의 경우 유니티에서 제공하는 스크립터블 오브젝트를 상속받게 한다.
```c#
public class Item : ScriptableObject
{
    [Header("Item Information")]
    public Sprite itemIcon;         // 아이템 아이콘
    public string itemName;         // 아이템 이름
}
```

<br>

## WeaponItem 스크립트
Item 스크립트를 상속받으며 유니티 메뉴창에서 생성할 수 있도록 <br>
`[CreateAssetMenu] 속성`을 달아준다.

```c#
[CreateAssetMenu(menuName = "Items/Weapon Item")]
public class WeaponItem : Item
{
    public GameObject modelPrefab;      // 무기 프리팹
    public bool isUnarmed;              // 비무장 무기를 사용할때 체크(현재는 미사용)
}
```

createAssetMenu를 통해 WeaponItem은 메뉴에서 `생성이 가능`하게 되었다.

![image](https://github.com/novicehog/comments/assets/131991619/569c0fc0-a95c-4e90-8107-fe34088cccbc)

<br>

스크립터블 오브젝으를 통해 객체를 생성하기 전에 sword_01의 프리팹을 만들어준다. <br>
sword_01 프리팹은 빈 오브젝트 Sword_01이 최상위 부모로 <br>
그 밑으로 무기의 `위치와 크기를 잡아줄 Weapon Pivot` 빈 오브젝트와<br>
`모델을 담당할 Sword_01`을 자식으로 가진다.

이때 Weapon Pivot의 크기는 100 100 100으로 해준다.<br>
플레이어 캐릭터 모델의 자식으로 들어가게되면 크기가 0.01배 되어있어<br>
무기 또한 0.01배가 되버리기 떄문<br>

![image](https://github.com/novicehog/comments/assets/131991619/259d7198-1631-4878-9051-32149a73458c)



WeaponItem 객체 Sword 생성

아이콘은 아직 없으므로 이름과 방금 만든 Sword_01 프리팹을 넣어줌

![image](https://github.com/novicehog/comments/assets/131991619/da9063f0-4d95-4c8e-a883-8b06b873fd50)
<br>

# WeaponHolderSlot
플레이어의 양손에 달아줄 스크립트인 WeaponHolderSlot 스크립트를 만든다.

```c#
public class WeaponHolderSlot : MonoBehaviour
{
    public Transform parentOverride;        // 왼손의 경우 바로 손의 자식으로 넣지않고 빈 객체 밑으로 넣는다.
    public bool isLeftHandSlot;             // 어느쪽 손인지 (왼손인지 오른손인지)
    public bool isRightHandSlot;            // 구분할 변수

    public GameObject currentWeaponModel;   // 현재 장착 중인 무기

    // 무기 해제 (비활성화 해서 해제)
    public void UnloadWeapon()
    {
        if(currentWeaponModel != null)
        {
            currentWeaponModel.SetActive(false);
        }
    }

    // 무기 해제 (무기를 파괴해서 해제)
    public void UnloadWeaponAndDestroy()
    {
        if(currentWeaponModel != null) 
        {
            Destroy(currentWeaponModel);
        }
    }

    // 무기를 파괴하여 해제하고 새로운 무기 장착
    public void LoadWeaponModel(WeaponItem weaponItem)
    {
        UnloadWeaponAndDestroy();

        if(weaponItem == null)
        {
            UnloadWeapon();
            return;
        }

        GameObject model = Instantiate(weaponItem.modelPrefab) as GameObject;
        if(model != null)
        {
            if(parentOverride != null)
            {
                model.transform.parent = parentOverride;
            }
            else
            {
                model.transform.parent = transform;
            }

            model.transform.localPosition = Vector3.zero;
            model.transform.localRotation = Quaternion.identity;
            model.transform.localScale = Vector3.one;
        }
        currentWeaponModel = model;
    }
}
```

왼손과 오른손에 스크립트들 달아줌.

왼손

![image](https://github.com/novicehog/comments/assets/131991619/921f9099-88a5-44d0-a45e-9082af1950f3)
<br>

오른손 

![image](https://github.com/novicehog/comments/assets/131991619/04991b4f-74fa-4e82-9291-c0c453cf20d8)
<br>


# Player 세팅

## WeaponSlotManager 스크립트

양손의 WeaponHolderSlot을 관리하는 WeaponSlotManager스크립트 생성

```c#
public class WeaponSlotManager : MonoBehaviour
{
    
    WeaponHolderSlot leftHandSlot;      
    WeaponHolderSlot rightHandSlot;

    private void Awake()
    {
        // 양손의 WeaponHolderSlot을 가져옴
        WeaponHolderSlot[] weaponHolderSlots = GetComponentsInChildren<WeaponHolderSlot>();
        foreach  (WeaponHolderSlot weaponSlot in weaponHolderSlots)
        {
            if(weaponSlot.isLeftHandSlot)
            {
                leftHandSlot = weaponSlot;
            }
            else if (weaponSlot.isRightHandSlot)
            {
                rightHandSlot = weaponSlot;
            }
        }
    }

    // WeaponHolderSlot를 이용하여 무기를 장착하는 함수 구현
    public void LoadWeaponOnSlot(WeaponItem weaponItem, bool isLeft)
    {
        if(isLeft)
        {
            leftHandSlot.LoadWeaponModel(weaponItem);
        }
        else
        {
            rightHandSlot.LoadWeaponModel(weaponItem);
        }
    }
}
```
<br>

플레이어의 모델에 스크립트를 넣어줌

![image](https://github.com/novicehog/comments/assets/131991619/1d41464b-9d3f-42e0-abeb-cb6d36df5d07)
<br>

## PlayerInventory 스크립트

아직은 간단히 WeaponSlotManager를 통해 무기를 장착하는 부분만 구현

```c#
public class PlayerInventory : MonoBehaviour
{
    WeaponSlotManager weaponSlotManager;

    public WeaponItem rightWeapon;
    public WeaponItem leftWeapon;

    private void Awake()
    {
        weaponSlotManager = GetComponentInChildren<WeaponSlotManager>();
    }

    private void Start()
    {
        weaponSlotManager.LoadWeaponOnSlot(rightWeapon, false);
        weaponSlotManager.LoadWeaponOnSlot(leftWeapon, true);
    }
}
```
<br>

Player에 스크립트를 넣어주고 만들어둔 Sword 객체를 넣어준다.

![image](https://github.com/novicehog/comments/assets/131991619/41125234-4c60-4713-8ad1-37e4a2ac5eab)