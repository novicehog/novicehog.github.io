---
layout: single
title:  "유니티 SerializeReference"
categories: 
    - Unity
tag: [Unity, Editor, Attribute]
published: true

author_profile: true # 옆에뜨는 프로파일

search: true #검색해도 안뜨게함

date: 2024-05-22
last_modified_at: 2024-05-22

---

이 글은 **`인프런`** [모듈식으로 개발하는 스킬 시스템](https://www.inflearn.com/course/%EC%9C%A0%EB%8B%88%ED%8B%B0-%EB%AA%A8%EB%93%88%EC%8B%9D-%EC%8A%A4%ED%82%AC-%EC%8B%9C%EC%8A%A4%ED%85%9C)를 보고 공부한 내용을 정리한 글입니다.
{: .notice--warning}

## SerializeField란?
유니티에서는 내가 만든 클래스를 인스펙터창에 띄우고 싶다면 직렬화를 해야한다. <br>
이때 보통 사용하는 것이 바로 `SerializeField` 이다. <br>

보통은 다음과 같이 사용된다.
```cs
[Serializable]
public class Entity
{
    public string name;
}

public class Test : MonoBehaviour
{
    [SerializeField]
    Entity entity;
}
```

<br>

위 코드는 인스펙터창에서 다음과 같이 보인다 <br>
![image](https://github.com/novicehog/comments/assets/131991619/5b9f11dc-26a0-4f9e-8b15-87b39f1d1ee3)

<br>

위 예시를 보면 아무런 문제 없이 잘 작동하지만 상속과 관련된다면 불편한 점이 생기게 된다. <br>
바로 다음 예시를 보자

```cs
[Serializable]
public class Entity
{
    public string name;
}


// Entity를 상속받는 자식 클래스 2개를 정의
[Serializable]
public class Human : Entity
{
    public float Attack;
}

[Serializable]
public class Zombie : Entity
{
    public float defense;
}

// Entity 자료형의 변수에 각각 Human, Zombie객체를 담음
public class Test : MonoBehaviour
{
    [SerializeField]
    Entity humanEntity = new Human();

    [SerializeField]
    Entity zombieEntity = new Zombie();
}
```
<br>

![SerializeField불편함](https://github.com/novicehog/comments/assets/131991619/54f4b7f6-0dc9-4b19-b1ed-cb1fad66db6c)

위 코드에서는 추가적으로 `Entity`클래스를 상속받는 Human클래스와 Zombie클래스를 새로 작성했다.<br>
그 뒤 `Entity`를 자료형으로 하는 `humanEntity변수`와 `zombieEntity변수`에 각각 `Human,Zombie객체를 할당`했다.<br>
하지만 결과 사진을 보면 보이다 싶이 인스펙터창에서 나타나는 변수들은 Entity의 필드들만 나타나게 된다.<br>

이러한 문제를 해결하기 위해 사용하는 Attribute가 바로 SerializeReference이다.
## SerializeReference란
SerializeReference는 클래스의 다형성을 지원하는 직렬화 Attribute이다.<br>
이를 사용하면 자료형을 기준으로 직렬화 하지 않고 할당된 객체를 기준으로 직렬화하게 된다.

위의 예시코드를 다음과 같이 바꿨다.


```cs

[Serializable]
public class Entity
{
    public string name;
}

[Serializable]
public class Human : Entity
{
    public float Attack;
}

[Serializable]
public class Zombie : Entity
{
    public float defense;
}

public class Test : MonoBehaviour
{
    // 수정된 부분
    [SerializeReference]
    Entity humanEntity = new Human();

    // 수정된 부분
    [SerializeReference]
    Entity zombieEntity = new Zombie();
}
```
<br>

![image](https://github.com/novicehog/comments/assets/131991619/428500c9-d050-491c-8085-1690cc866ddd)

<br>

위의 사진을 보면 알 수 있듯 SerializeField에서 발생했던 문제가 해결된 모습을 볼 수 있다.

## 결론
SerializeReference는 위 처럼 할당된 객체를 기준으로 직렬화 하기 때문에 `상황에 맞게 다른 자식 객체를 할당`하여 `유동적인 구성`이 가능하도록 한다.


## 추가적인 정보
SerializeReference의 불편한점이라고 한다면 인스펙터창에서 SerializeReference변수에 할당되는 객체를 바꾸는 기능은 따로 없다는 것이다.

이러한 불편함을 해결해주는 [다른 사람이 만든 코드가 있다.](https://github.com/mackysoft/Unity-SerializeReferenceExtensions/releases) <br>
이 코드는 유니티 공식 기능이 아니므로 유니티가 업데이트됨에 따라 작동하지 않을 수 있다.

### 사용법
패키지를 다운받아 유니티에 적용한 뒤, SerializeReference된 변수에 SubclassSelector Attribute를 추가해주면 된다.

```cs
[Serializable]
public class Entity
{
    public string name;
}

[Serializable]
public class Human : Entity
{
    public float Attack;
}

[Serializable]
public class Zombie : Entity
{
    public float defense;
}

public class Test : MonoBehaviour
{
    [SerializeReference, SubclassSelector]
    Entity Entity;
}
```

<br>

![sub](https://github.com/novicehog/comments/assets/131991619/f9f424a5-caf7-4409-98b4-8faa0caf5f80)

<br>

![huamn](https://github.com/novicehog/comments/assets/131991619/457c19df-b302-47f7-837b-e7ba8f55955c)

<br>

![zombie](https://github.com/novicehog/comments/assets/131991619/c2aafff4-b79c-4d81-ad40-6da157bbf5c9)