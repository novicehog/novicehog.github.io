---
layout: single
title:  "유니티 다크소울 따라만들기 ch_11 데미지"
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



# 구현 원리
TriggerEnter를 통해 데미지를 구현한다. <br>
이때 애니메이션 이벤트를 이용하여 콜라이더를 자연스럽게
켰다 끈다.

<br>

# 기초 작업

## 적 추가

### Enemy Stats 스크립트 추가
***

적에게 넣어줄 스크립트를 추가한다.

데미지를 맞으면 피격 애니메이션 재생되고 체력이 0이되면 사망 애니메이션이 재생된다.

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class EnemyStats : MonoBehaviour
{
    public int healthLevel = 10;
    public int maxHealth;
    public int currentHealth;

    Animator animator;

    private void Awake()
    {
        animator = GetComponentInChildren<Animator>();
    }

    private void Start()
    {
        maxHealth = SetMaxHealthFromHealthLevel();
        currentHealth = maxHealth;
    }

    private int SetMaxHealthFromHealthLevel()
    {
        maxHealth = healthLevel * 10;
        return maxHealth;
    }

    public void TakeDamage(int damage)
    {
        currentHealth = currentHealth - damage;


        animator.Play("Damage_01");

        if (currentHealth <= 0)
        {
            currentHealth = 0;
            animator.Play("Dead_01");
        }
    }
}

```

### 적 완성
***
만든 스크립트와 콜라이더 리지드바디를 넣어준다.

태그도 Enemy로 바꿔준다.

![image](https://github.com/novicehog/comments/assets/131991619/733a0cec-7e23-441d-8be3-1ad6495d18e7)

<br>

적의 모델의 애니메이터에는 Humanoid를 임시로 넣어준다.

![image](https://github.com/novicehog/comments/assets/131991619/b9d0bbb3-5c78-4012-90fe-be752c234b38)

<br>



## 무기
### 콜라이더 추가
***
검에 콜라이더 추가한다.

![image](https://github.com/novicehog/comments/assets/131991619/4061dc4d-08cd-4893-ad8c-689fa09bfd53)

<br>


### DamageCollider 스크립트 추가
***
데미지를 가하는 스크립트인 `DamageCollider`스크립트를 생성한다.

콜라이더를 끄거나 키는 기능이 있고, <br>
충돌한 대상의 `태그를 감지`해서 데미지를 가할 수 있다.

```c#
public class DamageCollider : MonoBehaviour
{
    Collider damageCollider;

    public int currentWeaponDamage = 25;

    private void Awake()
    {
        damageCollider = GetComponent<Collider>();
        damageCollider.gameObject.SetActive(true);
        damageCollider.isTrigger = true;
        damageCollider.enabled = false;
    }

    public void EnableDamageCollider()
    {
        damageCollider.enabled = true;
    }

    public void DisableDamageCollider()
    {
        damageCollider.enabled = false;
    }

    private void OnTriggerEnter(Collider collision)
    {
        if(collision.tag == "Player")
        {
            PlayerStats playerStats = collision.GetComponent<PlayerStats>();
            

            if(playerStats != null )
            {
                playerStats.TakeDamage(currentWeaponDamage);
            }
        }

        if(collision.tag == "Enemy")
        {
            EnemyStats enemyStats = collision.GetComponent<EnemyStats>();
            if(enemyStats != null )
            {
                enemyStats.TakeDamage(currentWeaponDamage);
            }
        }
    }
}


```



# 그 외 수정사항

## WeaponSlotManager 스크립트 수정

양손의 DamageCollider 변수를 선언한다.

무기가 슬롯에 로드될 때마다 DamageCollider를 참조한다.

애니메이션 이벤트에 사용할 Open, Close 함수들을 만든다.

```c#
public class WeaponSlotManager : MonoBehaviour
{
    WeaponHolderSlot leftHandSlot;
    WeaponHolderSlot rightHandSlot;

    DamageCollider leftHandDamageCollider;      
    DamageCollider rightHandDamageCollider;

    private void Awake()
    {
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

    public void LoadWeaponOnSlot(WeaponItem weaponItem, bool isLeft)
    {
        if(isLeft)
        {
            leftHandSlot.LoadWeaponModel(weaponItem);
            LoadLeftWeaponDamageCollider();
        }
        else
        {
            rightHandSlot.LoadWeaponModel(weaponItem);
            LoadRightWeaponDamageCollider();
        }
    }

    #region Handle Weapon's Damage Collider


    private void LoadLeftWeaponDamageCollider()
    {
        leftHandDamageCollider = leftHandSlot.currentWeaponModel.GetComponentInChildren<DamageCollider>();
    }

    private void LoadRightWeaponDamageCollider()
    {
        rightHandDamageCollider = rightHandSlot.currentWeaponModel.GetComponentInChildren<DamageCollider>();
    }

    public void OpenRightDamageCollider()
    {
        rightHandDamageCollider.EnableDamageCollider();
    }

    public void OpenLeftDamageCollider()
    {
        leftHandDamageCollider.EnableDamageCollider();
    }

    public void CloseRightHandDamageCollider()
    {
        rightHandDamageCollider.DisableDamageCollider();
    }

    public void CloseLeftHandDamageCollider()
    {
        leftHandDamageCollider.DisableDamageCollider();
    }
    #endregion
}
```

<br>

# 애니메이션 이벤트
WeaponSlotManager에 추가된 Open, Close함수를 애니메이션 이벤트를 통해 실행한다.

## 애니메이션 이벤트 추가

애니메이션 창을 띄운다.

![image](https://github.com/novicehog/comments/assets/131991619/ccb8250e-a5bd-48e2-a540-643916502ffd)

<br>

Scene창에서 Player를 찾아 클릭한다.

![image](https://github.com/novicehog/comments/assets/131991619/671565c4-4c44-45b1-914b-7e9240c79e3f)

<br>

이벤트를 추가할 애니메이션을 클릭한다.

![image](https://github.com/novicehog/comments/assets/131991619/9a8a91ef-affe-419f-bd0a-2360668bd50a)

<br>

`이때 애니메이션 ReadOnly 상태면 이곳에서 이벤트를 추가할 수 없다.`
<details>
<summary>해결방법</summary>
<div markdown="1">       

애니메이션을 복사하여 갈아껴준다.

단순히 애니메이션만 복사하여 갈아껴주면된다.

![image](https://github.com/novicehog/comments/assets/131991619/cc581e35-c820-40a0-8ee4-dd9ae4496224)


<img width="443" alt="image" src="https://github.com/novicehog/comments/assets/131991619/8d13ec49-37c6-4bba-a7b8-c61237932c7b">


<br>

</div>
</details> 



애니메이션을 재생해보며 원하는 타이밍에 
검의 콜라이더를 끄고 키는 스크립트를 추가한다.



![image](https://github.com/novicehog/comments/assets/131991619/f225a64a-436a-4131-84e3-b6fcf9d95483)

![image](https://github.com/novicehog/comments/assets/131991619/7e064cd8-d9de-4fe0-94d4-fd81bc8b94e2)




# 결과

![ezgif com-optimize](https://github.com/novicehog/comments/assets/131991619/509d547a-c079-4114-861c-8dcbc9808d7a)

<br>

# 배운점
- 애니메이션 이벤트에 대해 알 수 있었다.