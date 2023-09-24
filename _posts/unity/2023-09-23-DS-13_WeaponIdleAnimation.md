---
layout: single
title:  "유니티 다크소울 따라만들기 ch_13 무기  Idle 애니메이션"
categories: 
    - Unity
tag: [Unity, DarkSouls]
published: true

author_profile: true # 옆에뜨는 프로파일

search: true #검색해도 안뜨게함

date: 2023-09-23
last_modified_at: 2023-09-23

---

이 글은 유튜브 **Sebastian Graves Create Dark Souls**를 보고 따라 만들면서 헷갈리는 부분을 정리한 글입니다.
{: .notice--warning}

# 구현 개요

무기를 꽉 쥐게 만든다.

애니메이터의 레이어를 추가로 만들고
`아바타 마스크`를 통해 특정 부위에만 애니메이션을 재생하여

손을 꽉 쥐게 만든다.


# 추가된 부분

## 레이어와 아바타 마스크 추가

왼손과 오른손의 아바타 마스크를 만들어준다.

![image](https://github.com/novicehog/comments/assets/131991619/df149123-d069-435a-8080-90251044fde7)

![image](https://github.com/novicehog/comments/assets/131991619/4090cfef-e1b6-4f35-b418-017ae84a7e50)

<br>

왼손과 오른손의 레이어를 추가하고 애니메이션을 추가해준다.

오른손

![image](https://github.com/novicehog/comments/assets/131991619/43b3cb34-7751-416a-bbad-db327de07095)

![image](https://github.com/novicehog/comments/assets/131991619/cd224671-279a-4c67-8cad-4b3cb211e395)

<br>

왼손

![image](https://github.com/novicehog/comments/assets/131991619/d56a0d22-8dd0-45ac-bc7a-cff381c18aa9)

![image](https://github.com/novicehog/comments/assets/131991619/1acc619e-628d-40ba-a106-3f5ffe76b1fe)


이때 가중치는 애니메이션에 맞게 설정해준다.



## WeaponItem 스크립트 수정

무기를 꽉 잡는 애니메이션을 추가해준다.

```c#
[CreateAssetMenu(menuName = "Items/Weapon Item")]
public class WeaponItem : Item
{
    public GameObject modelPrefab;
    public bool isUnarmed;

    [Header("Idle Animations")]
    public string right_Hand_Idle;  //추가 
    public string left_hand_idle;  // 추가

    [Header("Attack Animations")]
    public string OH_Light_Attack_01;
    public string OH_Heavy_Attack_01;
    public string OH_Light_Attack_02;
}

```

<br>

## WeaponSlotManager 스크립트 수정

무기에 내장된 무기를 잡는 애니메이션을 <br>
무기가 로드될 때 자동을 실행되도록 한다.

```c#
public void LoadWeaponOnSlot(WeaponItem weaponItem, bool isLeft)
{
    if (isLeft)
    {
        leftHandSlot.LoadWeaponModel(weaponItem);
        LoadLeftWeaponDamageCollider();
        #region Handle Weapon Idel Animations;
        if (weaponItem != null)
        {
            animator.CrossFade(weaponItem.left_hand_idle, 0.2f);
        }
        else
        {
            animator.CrossFade("Left Arm Empty", 0.2f);
        }
        #endregion
    }
    else
    {
        rightHandSlot.LoadWeaponModel(weaponItem);
        LoadRightWeaponDamageCollider();
        #region Handle Right Weapon Idle Animations
        if (weaponItem != null)
        {
            animator.CrossFade(weaponItem.right_Hand_Idle, 0.2f);
        }
        else
        {
            animator.CrossFade("Right Arm Enpty", 0.2f);
        }
        #endregion
    }
}
```

<br>

# 배운점
- 아바타 마스크를 통해 부위별 애니메이션을 다르게 주는 방법을 알았다.