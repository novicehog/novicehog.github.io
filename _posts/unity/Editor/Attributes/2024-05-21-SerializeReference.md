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
이 글은 **`인프런`** [** 모듈식으로 개발하는 스킬 시스템**](https://www.inflearn.com/course/%EC%9C%A0%EB%8B%88%ED%8B%B0-%EB%AA%A8%EB%93%88%EC%8B%9D-%EC%8A%A4%ED%82%AC-%EC%8B%9C%EC%8A%A4%ED%85%9C)를 보고 공부한 내용을 정리한 글입니다.
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
public class Human : Entity
{
    public float Attack;
}

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
하지만 결과 사진을 보면 보이다 싶이 인스펙터창에서 나타나는 변수들은 Entity의 필드들만 나타나게 된다.



