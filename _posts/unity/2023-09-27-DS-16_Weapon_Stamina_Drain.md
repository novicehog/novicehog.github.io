---
layout: single
title:  "유니티 다크소울 따라만들기 ch_16 무기 스테미나 고갈"
categories: 
    - Unity
tag: [Unity, DarkSouls]
published: true

author_profile: true # 옆에뜨는 프로파일

search: true #검색해도 안뜨게함

date: 2023-09-27
last_modified_at: 2023-09-27

---

이 글은 유튜브 **Sebastian Graves Create Dark Souls**를 보고 따라 만들면서 헷갈리는 부분을 정리한 글입니다.
{: .notice--warning}

# 구현 개요
무기를 휘두를 때 스테미너가 감소되게 하고 UI로 표시한다.

# stamina bar 추가

체력바와 똑같은 방식으로 추가한다.

![image](https://github.com/novicehog/comments/assets/131991619/b64ab751-9529-4ea2-be4d-407475a77723)

<br>

## StaminaBar 스크립트 추가

체력바와 똑같은 방식의 StaminaBar 스크립트를 추가한다.

```c#
using UnityEngine.UI;

public class StaminaBar : MonoBehaviour
{
    public Slider slider;
    private void Start()
    {
        slider = GetComponent<Slider>();
    }
    public void SetMaxHealth(int maxHealth)
    {
        slider.maxValue = maxHealth;
        slider.value = maxHealth;
    }

    public void SetCurrentHealth(int health)
    {
        slider.value = health;
    }
}
```

<br>

# 그 외 스크립트 수정

## PlayerStats 스크립트 수정

스테미너 소모를 담당하는 TakeStaminaDamage 함수를 만든다.

```c#
public class PlayerStats : MonoBehaviour
{
    public int healthLevel = 10;
    public int maxHealth;
    public int currentHealth;

    public int staminaLevel = 10;
    public int maxStamina;
    public int currentStamina;

    HealthBar healthbar;
    StaminaBar staminaBar;
    AnimatorHandler animatorHandler;



    private void Awake()
    {
        healthbar = FindAnyObjectByType<HealthBar>();
        staminaBar = FindAnyObjectByType<StaminaBar>();
        animatorHandler = GetComponentInChildren<AnimatorHandler>();
    }

    private void Start()
    {
        maxHealth = SetMaxHealthFromHealthLevel();
        currentHealth = maxHealth;
        healthbar.SetMaxHealth(maxHealth);
        maxStamina = SetMaxStaminaFromFromStaminaLevel();
        currentStamina = maxStamina;
        staminaBar.SetMaxHealth(maxStamina);
    }

    private int SetMaxHealthFromHealthLevel()
    {
        maxHealth = healthLevel * 10;
        return maxHealth;
    }

    private int SetMaxStaminaFromFromStaminaLevel()
    {
        maxStamina = staminaLevel * 10;
        return maxStamina;
    }

    public void TakeDamage(int damage)
    {
        currentHealth = currentHealth - damage;

        healthbar.SetCurrentHealth(currentHealth);

        animatorHandler.PlayTargetAnimation("Damage_01", true);

        if(currentHealth <= 0)
        {
            currentHealth = 0;
            animatorHandler.PlayTargetAnimation("Dead_01", true);
        }
    }

    public void TakeStaminaDamage(int damage)
    {
        currentStamina = currentStamina - damage;
        staminaBar.SetCurrentHealth(currentStamina);
    }
}
```


## WeaponItem 스크립트 수정

스테미너 부분을 추가해준다.

기본 스테미너 소모량을 가지고 강공격, 약공격에 따라 기본 스테미너 소모량에 기반하여 <br>
스테미너를 소모함

```c#
public class WeaponItem : Item
{
    public GameObject modelPrefab;
    public bool isUnarmed;

    [Header("Idle Animations")]
    public string right_Hand_Idle;
    public string left_hand_idle;

    [Header("Attack Animations")]
    public string OH_Light_Attack_01;
    public string OH_Heavy_Attack_01;
    public string OH_Light_Attack_02;

    // 추가된 부분
    [Header("Stamina Costs")]
    public int baseStamina;
    public float lightAttackMultiplier;
    public float heavyAttackMultiplier;
}
```

<br>

<img width="209" alt="image" src="https://github.com/novicehog/comments/assets/131991619/6c3d5461-7681-4e1b-b62c-150cff8fc1de">

<br>

## WeaponSlotManager 스크립트 수정

애니메이션 이벤트에 추가할 함수를 추가한다.

```c#
public class WeaponSlotManager : MonoBehaviour
{
    public WeaponItem attackingWeapon; // 추가된 부분

    ...

    PlayerStats playerStats;

    private void Awake()
    {
        ...
        playerStats = GetComponentInParent<PlayerStats>();
        ...
    }

    ...

    #region Handle Weapon's Stamina Drainage

    public void DrainStaminaLightAttack()
    {
        playerStats.TakeStaminaDamave(Mathf.RoundToInt(attackingWeapon.baseStamina * attackingWeapon.lightAttackMultiplier));
    }

    public void DrasinStaminaHeavyAttack()
    {
        playerStats.TakeStaminaDamave(Mathf.RoundToInt(attackingWeapon.baseStamina * attackingWeapon.heavyAttackMultiplier));
    }
    #endregion
}
```

<br>


## PlayerAttecker 수정

WeaponSlotManager의 attacingWeapon을 공격할 때마다 수정해준다.

```c#
WeaponSlotManager weaponSlotManager;

private void Awake()
{
    ...
    weaponSlotManager = GetComponentInChildren<WeaponSlotManager>();
    ...
}

public void HandleLightAttack(WeaponItem weapon)
{
    weaponSlotManager.attackingWeapon = weapon; // 추가된 부분
    animatorHandler.PlayTargetAnimation(weapon.OH_Light_Attack_01, true);
    lastAttack = weapon.OH_Light_Attack_01;
}

public void HandleHeavyAttack(WeaponItem weapon)
{
    weaponSlotManager.attackingWeapon = weapon; //추가된 부분
    animatorHandler.PlayTargetAnimation(weapon.OH_Heavy_Attack_01, true);
    lastAttack = weapon.OH_Heavy_Attack_01;
}
```

# 애니메이션 이벤트에 추가

공격 애니메이션에 스테미너 소모 함수를 추가한다.

![image](https://github.com/novicehog/comments/assets/131991619/e5c936e0-8591-4738-8c59-90bccfc714cd)

<br>


# 결과

![ezgif com-optimize](https://github.com/novicehog/comments/assets/131991619/afc369fa-ed8d-4ff0-81bd-f5a3c46f4a2d)