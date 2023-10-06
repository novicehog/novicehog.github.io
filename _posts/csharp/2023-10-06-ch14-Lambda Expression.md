---
layout: single
title:  "C# Chapter14 람다식"
categories: 
    - C Sharp
tag: [c#]
published: true

author_profile: true # 옆에뜨는 프로파일

date: 2023-10-06
last_modified_at: 2023-10-16
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
 -->

이 글은 한빛미디어의 **이것이 C#이다**를 보고 헷갈리는 부분 위주로 정리한 글입니다.
{: .notice--warning}

# 람다식

람다식은 익명 메소드를 만들기 위해 사용한다.

기본적인 람다식의 형식은 다음과 같다.

```c#
매개변수_목록 => 식
```

=> 연산자는 `입력` 연산자로 매개변수를 전달하는 것 뿐이다. <br>
람다식에서는 `=>`를 중심으로 `왼쪽에 매개변수`, `오른쪽에 식이` 위치한다.

람다식의 예는 다음과 같다.

```c#
delegate int Calculate(int a, int b);
public static void Main(string[] args)
{
    Calculate Calc = (int a, int b) => a + b;
}
```

<br>

또 람다식에서는 형식 유추라는 기능을 제공하여, `매개변수의 형식을 제거`할 수 있다.

```c#
delegate int Calculate(int a, int b);
public static void Main(string[] args)
{
    Calculate Calc = (a,b) => a + b;
}
```

<br>

이러한 람다식을 대리자를 통한 익명메소드로 구현해보면 다음과 같이 비교적 복잡해지는 것을 알 수 있다.

```c#
delegate int Calculate(int a, int b);
public static void Main(string[] args)
{
    Calculate Calc = delegate(int a, int b)
                    {
                        return a + b;
                    }
}
```

<br>

## 문 형식의 람다식

식 형식의 람다식은 반환 형식이 없는 무명 함수를 만들 수 없지만<br>
문 형식의 람다식은 가능하다.

```c#
(매개변수_목록) => {
                        문장1;
                        문장2
                        ...
                  }
```

<br>

문 형식의 람다식을 이용하여 반환 형식도, 매개변수도 없는 대리자를
문 형식의 람다식을 통해 익명 메소드를 등록할 수 있다.

```c#
delegate void DoSomething();
public static void Main(string[] args)
{
    DoSomething Doit = () =>
    {
        Console.WriteLine("아무거나 출력하기");
    };
}
```

<br>

## Func와 Action으로 더 간편하게 무명 함수 만들기
대부분 단 하나의 익명 메소드나 무명 함수를 만들기 위해 `매번 별개의 대리자를 선언`해야 하는 것은 번거롭다.

이 문제를 해결하기 위해 `.Net`에 Func와 Action 대리자를 미리 선언해 두었다.

`Func 대리자는 결과를 반환`하는 메소드, <br>
`Action 대리자는 결과를 반환하지 않는` 메소드를 참조한다.

이; 두 대리자가 어떻게 귀찮은 문제를 해결해주는지 보겠다.

## Func 대리자
Func대리자는 결과를 반환하는 메소드를 참조하기 위해 만들어졌다. 

매개변수 개수에 따라 0개부터 ~ 16개까지 총 17가지의 버전이 있다.

마지막에 오는 형식 매개변수가 반환 형식이 된다.
```c#
Func<매개변수_자료형1,매개변수_자료형2,....,반환_형식>

Func<int> func1 = () => 10;
Func<int,int> func2 = (x) => x*2;
Func<int,int,int> func3 = (x, y) => x + y;

Console.WriteLine(func1());     // 10
Console.WriteLine(func2(2));    // 4
Console.WriteLine(func3(3, 8)); // 11
```

<br>

## Action 대리자
Action대리자는 Func 대리자아 거의 똑같으나 `반환 형식이 없다`는 것 뿐이다.

```c#
Action action1 = () => Console.WriteLine("Action1()");
Action<int> action2 = (x) => Console.WriteLine($"Action2({x})");
Action<int, int> action3 = (x, y) => Console.WriteLine($"Action2({x},{y})");

action1();      //Action1()
action2(3);     // Action2(3)
action3(6, 8);  // Action3(6,8)
```
