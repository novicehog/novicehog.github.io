---
layout: single
title:  "유니티 DecoratorDrawer"
categories: 
    - Unity
tag: [Unity, Editor]
published: true

author_profile: true # 옆에뜨는 프로파일

search: true #검색해도 안뜨게함

date: 2024-05-21
last_modified_at: 2024-05-21

---

## DecoratorDrawer 란
유니티에서는 Editor에서 보여지는 여러 요소들을 `커스터마이징` 할 수 있도록 도와준다. 

여기서 볼 DecoratorDrawer는 그 중 한가지로, property를  데이터를 기반으로 `장식적인 요소`를 그리는 것이다. `(property 자체를 그리는 것이 아님)`

즉, 속성 주변을 꾸며주는 기능이라고 보면 된다.

## 사용 상황
일반적으로 인스펙터 창에서 보여지는 변수들은 다음과 같이 밋밋하다.

![image](https://github.com/novicehog/comments/assets/131991619/9c569660-5b09-4630-a384-f7d099e8351d)
<br>

이 때 DecoratorDrawer를 통해 제목과 밑줄을 그어주는 기능을 커스터마이징하여 적용하면 다음과 같이 변수들이 보기좋게 된다.

![image](https://github.com/novicehog/comments/assets/131991619/9e667a10-d201-415f-a6bd-72b3d68bc6c4)
<br>

여기서 DecoratorDrawer를 통해 만들어진 부분은 다음과 같이 변수 자체가 아니라 변수 장식인 제목과 밑줄이다.

<img width="300" alt="image" src="https://github.com/novicehog/comments/assets/131991619/e941f1de-8336-4d3f-99e0-9b31d14b2987">

## 사용 방법
DecoratorDrawer를 사용하기 위해서는 먼저 `그릴 때 사용할 값`들이 필요하다.<br>
이를 위해 수치 값들을 받아오기 위해 PropertyAttribute를 상속받는 클래스 하나가 필요하다.

말로 하면 잘 와닿지 않는데, 그러면 유니티에서 기본적으로 제공하는 PropertyAttribute들을 보면 된다.

```cs
[Header("My Variables")]    // My Variables라는 값를 저장하여 그릴 때 제목으로 사용
[Space(10)]                 // 10이라는 값을 저장하여 공간을 띄울 때 수치로 사용          
public string myDescription;

[Range(0, 100)]             // 0과 100이라는 값을 저장하여 범위를 그릴 때 수치를 사용
public float myFloat;
```
<br>

그래서 결론은 DecoratorDrawer를 사용하기 위해선 두 개의 스크립트를 생성하여야 한다.
- PropertyAttribute : 데이터 값들을 저장
- Decorator Drawer : PropertyAttribute를 통해 받은 값들을 토대로 GUI를 그림


그럼 이제 위에서 본것처럼 제목과 밑줄을 긋는 예시를 직접 만들어보면 다음과 같다.

### PropertyAttribute를 상속받는 UnderlineTitleAttribute
먼저 그릴 때 사용할 값들을 받아와야 한다. <br>
여기서는 값을 두 가지를 받는다.
- 제목
- 띄울 공간

```cs
public class UnderlineTitleAttribute : PropertyAttribute
{
    public string Title { get; private set; }

    public int Space { get; private set; }

    public UnderlineTitleAttribute(string title, int space = 12)
    {
        Title = title;
        Space = space;
    }
}
```

### DecoratorDrawer를 상속받는 UnderlineTitleDrawer
DecoratorDrawer를 상속받아 직접 구현할 때는 2가지를 작성해주어야 한다.
- Attribute의 Type을 인자로 받는 CustomPropertyDrawer Attribute를 클래스 위에 작성해준다.
- OnGUI함수와 GetHeight함수 오버라이딩

#### CustomPropertyDrawer 작성
다음과 같이 클래스 위에 작성해주면 된다.

```cs
[CustomPropertyDrawer(typeof(UnderlineTitleAttribute))]
public class UnderlineTitleDrawer : DecoratorDrawer
{
    ...
}
```
<br>

#### OnGUI함수와 GetHeight함수 오버라이딩
첫 번째로 오버라이드할 함수 `OnGUI(Rect position)`는 GUI를 어떻게 그릴지를 직접 작성한다.

```cs
public override void OnGUI(Rect position)
{
    // attribute에는 실제 attribute 객체가 들어있음
    var attributeAsUnderlineTitle = attribute as UnderlineTitleAttribute;

    // 들여쓰기가 적용된 포지션으로 변환
    position = EditorGUI.IndentedRect(position);
    // 유니티에서 기본 한 줄 높이
    position.height = EditorGUIUtility.singleLineHeight;
    // y축을 Attribute에서 설정한 Space만큼 내림
    position.y += attributeAsUnderlineTitle.Space;

    GUI.Label(position, attributeAsUnderlineTitle.Title, EditorStyles.boldLabel);

    // 한줄 이동
    position.y += EditorGUIUtility.singleLineHeight;
    // 두께 1
    position.height = 1f;
    // 회색 선을 그림
    EditorGUI.DrawRect(position, Color.gray);
}
```

<br>
<br>

두 번째로 오버라이드할 함수 GetHeight는 내가 그런 GUI는 `이정도 높이를 가지고 있다를 반환`해준다.<br>
이 프로퍼티를 다 그린 뒤 `다음 프로퍼티를 그리기 시작할 때 그려지는 위치를 구할 때 사용`된다.

```cs
// 이 함수를 통해서 이 프로퍼티는 이정도의 높이를 가지고 있다고 알림 그래서
// 이 프로퍼티를 그린 다음, 다음에 그릴 프로퍼티가 그려지는 위치를 구할 때 사용됨
public override float GetHeight()
{
    var attributeAsUnderlineTitle = attribute as UnderlineTitleAttribute;
    // 기본 GUI 높이 + (기본 GUI 간격 * 2) 설정한 Attribute Space
    return attributeAsUnderlineTitle.Space + EditorGUIUtility.singleLineHeight + (EditorGUIUtility.standardVerticalSpacing * 2);
}
```

## 전체 코드
```cs
// Target으로 하는 Attribute의 Type
[CustomPropertyDrawer(typeof(UnderlineTitleAttribute))]
public class UnderlineTitleDrawer : DecoratorDrawer
{
    public override void OnGUI(Rect position)
    {
        // attribute에는 실제 attribute 객체가 들어있음
        var attributeAsUnderlineTitle = attribute as UnderlineTitleAttribute;

        // 들여쓰기가 적용된 포지션으로 변환
        position = EditorGUI.IndentedRect(position);
        // 유니티에서 기본 한 줄 높이
        position.height = EditorGUIUtility.singleLineHeight;
        // y축을 Attribute에서 설정한 Space만큼 내림
        position.y += attributeAsUnderlineTitle.Space;

        GUI.Label(position, attributeAsUnderlineTitle.Title, EditorStyles.boldLabel);

        // 한줄 이동
        position.y += EditorGUIUtility.singleLineHeight;
        // 두께 1
        position.height = 1f;
        // 회색 선을 그림
        EditorGUI.DrawRect(position, Color.gray);
    }

    // 이 함수를 통해서 이 프로퍼티는 이정도의 높이를 가지고 있다고 알림 그래서
    // 이 프로퍼티를 그린 다음, 다음에 그릴 프로퍼티가 그려지는 위치를 구할 때 사용됨

    public override float GetHeight()
    {
        var attributeAsUnderlineTitle = attribute as UnderlineTitleAttribute;
        // 기본 GUI 높이 + (기본 GUI 간격 * 2) 설정한 Attribute Space
        return attributeAsUnderlineTitle.Space + EditorGUIUtility.singleLineHeight + (EditorGUIUtility.standardVerticalSpacing * 2);
    }
}
```


## 실제 사용하는 코드는 다음과 같다
```cs
public class Test : MonoBehaviour
{
    [UnderlineTitle(title: "나이")]
    public int age;

    [UnderlineTitle(title: "이름")]
    public string myName;

    [UnderlineTitle(title: "몸무게")]
    public int weight;
}
```
<br>

![image](https://github.com/novicehog/comments/assets/131991619/9c569660-5b09-4630-a384-f7d099e8351d)

## 결론
Decorator Drawer는 필수로 사용해야할 기능은 아니다.<br>
그저 변수가 보여지는 모습을 그저 꾸며주기만 할 뿐이지만 나중에 인스펙터창의 정리가 필요하고 원하는 기능이 기본적으로 없다면 한 번 사용해볼만 한 기능이다.