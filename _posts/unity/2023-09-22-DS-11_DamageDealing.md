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

# 구현 원리
TriggerEnter를 통해 데미지를 구현한다. <br>
이때 애니메이션 이벤트를 이용하여 콜라이더를 자연스럽게
켰다 끈다.

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


DamageCollider 스크립트 추가

WeaponSlotManager 스크립트 수정


애니메이션 이벤트