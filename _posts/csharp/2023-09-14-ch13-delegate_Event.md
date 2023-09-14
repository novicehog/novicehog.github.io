---
layout: single
title:  "C# Chapter13 delegate 대리자와 event 이벤트"
categories: 
    - C Sharp
tag: [c#]
published: true

author_profile: true # 옆에뜨는 프로파일

date: 2023-09-14
last_modified_at: 2023-09-14
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

# 대리자 delegate 란?
- `delegate`는 메소드를 대신 호출해주는 대리인으로써 한마디로 말하면, 
delegate는 메소드에 대한 `참조`이다.

## delegate의 기본 형태
delegate의 기본 형태는 다음과 같다.
```c#
// 기본 형태
한정자 delegate 반환_형식 대리자_이름(매개변수_목록);

// 예시
public delegate int MyDelegate(int a, int b);
```

delegate는 메소드에 대한 참조이기 때문에 자신이 참조할 메소드의 `반환 형식`과 `매개변수`를 명시해 주어야함.
그렇게 떄문에 `delegate`는 메소드와 매우 유사한 모양을 가지고 있음.

여기서 한가지 알아두어야 할 것은 delegate는 `인스턴스(객체)`가 아닌 `형식(Type)`인 점이다. <br>
즉, int, string과 같은 형식이며 `메소드를 참조하는 그 무엇`을 만드려면 `인스턴스화` 해줘야한다.
```c#
delegate int MyDelegate(int a); 
static void Main(string[] args)
{
    MyDelegate delegateCallBack; // 대리자 변수 생성
}
```
<br>


이렇게 `인스턴스화`된 delegate 객체는 메소드들의 주소를 참조 할 수 있다.

```c#
MyDelegate delegateCallBack = new MyDelegate(MyMethod); // 메소드 참조

Console.WriteLine(delegateCallBack(123456)); // 대리자 호출 즉, delegate가 참조하는 함수 실행 // 123456

int MyMethod(int a) // 메소드
{
    //...
    return a;
}
```
<br>

![image](https://github.com/novicehog/comments/assets/131991619/e765c271-97f5-462d-a014-f09e6a724e54)

지금까지의 과정 순서는 다음과 같다.
1. 대리자를 선언한다.
2. 대리자의 인스턴스를 생성, 인스턴스를 생성할 떄는 대리자가 참조할 메소드를 인수로 넘긴다.
3. 대리자를 호출한다.


## delegate는 왜, 언제 사용하나?

