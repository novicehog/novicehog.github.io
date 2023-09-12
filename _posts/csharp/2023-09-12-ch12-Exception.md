---
layout: single
title:  "C# Chapter 12 Exception 예외처리"
categories: 
    - C Sharp
tag: [c#]
published: true

author_profile: true # 옆에뜨는 프로파일

date: 2023-09-12
last_modified_at: 2023-09-12
---
이 글은 한빛미디어의 **이것이 C#이다**를 보고 헷갈리는 부분 위주로 정리한 글입니다.
{: .notice--warning}


# 예외

예외란 프로그램이 프로그래머가 생각한 시나리오에서 벗어나는 사건.

다음 길이가 3인 배열의 4번째 요소에 접근하는 예시가 있다.

```csharp
int[] a = new int[3];
Console.WriteLine(a[3]);
// 0, 1, 2의 방밖에 없는데 3번방을 접근하려함
// System.IndexOutOfRangeException: 'Index was outside the bounds of the array.'
```

이때 위와 같은 에러 메세지가 뜨며 프로그램이 강제 종료된다.

이 메세지는 `CLR`가 출력한 것이다.

잘못된 인덱스를 통해 배열에 요소에 접근하려 들면 배열 객체가 이 문제에 대한 상세 정보를 System.IndexOutOfRangeException의 객체에 담은 후 `Main()` 메소드에 던지는데, 이 예제의 Main()메소드는 이 예외를 처리할 방도가 없기 떄문에 `다시 CLR`에 던진다.

그렇게 되면 이 에러는 처리되지 않은 예외가 되고 사용자에게 출력한 뒤 프로그램이 `강제 종료`된다.

`**CLR → Main → CLR → 사용자에게 알리며 강제종료**`

그럼 예외를 받으려면 어떻게 해야할까

## try~catch로 예외 받기

앞에서 예시로 봤던 배열의 에러를 해결하는 것은 간단하다.

`**예외를 Main() 메소드가 받으면 된다.**`

C#에서는 예외를 받을 때 다음과 같이 **`try ~ catch문`** 을 이용한다.

```csharp
try
{
		// 실행하려는 코드
}
catch(예외_객체_1)
{
		// 예외가 발생했을 때의 처리
}
catch{예외_객체_2)
{
		// 예외가 발생했을 때의 처리
}
```

`**try**` 의 코드 블록에는 예외가 일어나지 않을 경우에 실행되어야 할 코드들이 들어가고,

`**catch**` 절에는 예외가 발생했을 때의 처리 코드가 들어간다

`try`에서 코드가 실행되다 예외가 던져 지면 `catch`블록이 받아낸다. 이때 catch절은 try 블록에서 던질 예외 객체와 형식이 일치해야 한다. 그렇지 않으면 던져 진 예외는 아무도 받지 못해 `**처리되지 않은 예외**`로 남게 된다.

위와 같이 try문이 `**여러 개의 예외**`를 던질 가능성이 있다면, `**여러 개의 catch블록**`을 두어 받아낼 수 있다.

다음은 예시이다

```csharp
int[] arr = { 1, 2, 3 };

try
{
    for (int i = 0; i < 5; i++)
    {
        Console.WriteLine(arr[i]);
    }
}
catch (IndexOutOfRangeException e)
{
    Console.WriteLine($"예외가 발생했습니다 : {e.Message}");
}

Console.WriteLine("종료");
```

## System.Exception 클래스