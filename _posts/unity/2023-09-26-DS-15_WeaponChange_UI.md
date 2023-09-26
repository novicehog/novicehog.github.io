---
layout: single
title:  "유니티 다크소울 따라만들기 ch_15 무기  Slot UI"
categories: 
    - Unity
tag: [Unity, DarkSouls]
published: true

author_profile: true # 옆에뜨는 프로파일

search: true #검색해도 안뜨게함

date: 2023-09-26
last_modified_at: 2023-09-26

---

이 글은 유튜브 **Sebastian Graves Create Dark Souls**를 보고 따라 만들면서 헷갈리는 부분을 정리한 글입니다.
{: .notice--warning}


# 구현 개요
무기 교체 시스템과 UI를 연동시킨다.

# UI 추가

Image를 통해 SLot UI를 다음과 같이 생성

![image](https://github.com/novicehog/comments/assets/131991619/4a77ffbf-d66d-4feb-ab6b-9e20f6fc6dbd)



![image](https://github.com/novicehog/comments/assets/131991619/86b027c4-eefc-4485-afe9-8079a5390387)

<br>

# 스크립트 수정

## QuickSlotUi 스크립트 추가

WeaponItem을 매개변수로 받아서 아이콘을 UI에 띄운다.

```c#
public class QuickSlotsUI : MonoBehaviour
{
    public Image leftWeaponIcon;
    public Image rightWeaponIcon;

    public void UpdateWeaponQucikSlotsUI(bool isLeft, WeaponItem weapon)
    {
        if(isLeft == false)
        {
            if(weapon.itemIcon != null)
            {
                rightWeaponIcon.sprite = weapon.itemIcon;
                rightWeaponIcon.enabled = true;
            }
            else
            {
                rightWeaponIcon.sprite = null;
                rightWeaponIcon.enabled = false;
            }
        }
        else
        {
            if (weapon.itemIcon != null)
            {
                leftWeaponIcon.sprite = weapon.itemIcon;
                leftWeaponIcon.enabled = true;
            }
            else
            {
                leftWeaponIcon.sprite = weapon.itemIcon;
                leftWeaponIcon.enabled = false;
            }
        }
    }
}
```

<br>

## weaponSlotManager 수정

QuickSlotUI를 가져와서 무기를 바꿀 때 마다 함수를 실행한다.

```c#
QuickSlotsUI quickSlotsUI;

private void Awake()
{
    ...
    quickSlotsUI = FindAnyObjectByType<QuickSlotsUI>();
    ...
}

public void LoadWeaponOnSlot(WeaponItem weaponItem, bool isLeft)
{
    if (isLeft)
    {
        leftHandSlot.LoadWeaponModel(weaponItem);
        LoadLeftWeaponDamageCollider();
        quickSlotsUI.UpdateWeaponQucikSlotsUI(true, weaponItem); // 추가
        ...
    }
    else
    {
        rightHandSlot.LoadWeaponModel(weaponItem);
        LoadRightWeaponDamageCollider();
        quickSlotsUI.UpdateWeaponQucikSlotsUI(false, weaponItem); // 추가
        ...
    }
}
```


# 아이콘 만들기

간단하게 아이템의 사진을 찍는다.

<img width="155" alt="sword01" src="https://github.com/novicehog/comments/assets/131991619/512885e7-7ea9-4cb2-86d9-1b28ebeb9133">

<br>

찍은 사진을 다음 사이트에서 PNG로 전환한다.
[**PNG변환 사이트**](https://www.remove.bg/ko)

<img width="155" alt="Sword01" src="https://github.com/novicehog/comments/assets/131991619/71c68d29-d0dd-40d7-86e2-6660fdb4ccc1">

<br>

가져온 Png를 Sprite로 변환한다.

<img width="205" alt="image" src="https://github.com/novicehog/comments/assets/131991619/ef428057-1f55-48bc-8a68-b2c7f958a907">


weaponItem 스크립터블 오브젝트에 넣어준다.

![image](https://github.com/novicehog/comments/assets/131991619/900dd327-f355-4887-8ad4-f8f20f1d755b)

<br>

# 결과

![ezgif com-video-to-gif](https://github.com/novicehog/comments/assets/131991619/641cfee2-1f2d-4b38-93c5-46751e86dcfe)


# 배운점
- Png로 변환하는 간단한 방법을 찾아낼 수 있었다.


