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
<!-- 
{: .notice--warning} // 알림 강조
{: .H1-font}         // 제목 색
<span style="color:Skyblue"> 색 넣기 </span>
<br/> 한줄 내리기
 -->

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


`CLR → Main → CLR → 사용자에게 알리며 강제종료`

그럼 예외를 받으려면 어떻게 해야할까

## try~catch로 예외 받기

앞에서 예시로 봤던 배열의 에러를 해결하는 것은 간단하다.

`예외를 Main() 메소드가 받으면 된다.`

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

`try` 의 코드 블록에는 예외가 일어나지 않을 경우에 실행되어야 할 코드들이 들어가고,

`catch` 절에는 예외가 발생했을 때의 처리 코드가 들어간다

`try`에서 코드가 실행되다 예외가 던져 지면 `catch`블록이 받아낸다. 이때 catch절은 try 블록에서 던질 예외 객체와 형식이 일치해야 한다. 그렇지 않으면 던져 진 예외는 아무도 받지 못해 `처리되지 않은 예외`로 남게 된다.

위와 같이 try문이 `여러 개의 예외`를 던질 가능성이 있다면, `여러 개의 catch블록`을 두어 받아낼 수 있다.

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

모든 예외 클래스의 조상이다.
앞에서 사용한 `IndexOutOfRangeException`클래스도 이 클래스를 상속받았다고 볼 수 있음

그러므로 System.Exception 을 사용하면 모든 예외를 전부 받아낼 수 있게됨

즉, 플레이어가 계산하지 않은 예외도 받아내게 됨

다음과 같은 예외 처리도

```csharp
try
{
}
catch(IndexOutOfRangeException e)
{
	//...
}
catch(DivideByZeroException e)
{
	//...
}
```
<br/>

Exception 으로 한 번에 받을 수 있다.

```csharp
try
{
}
catch(Exception e)
{
	//...
}
```

하지만 예외 상황에 따라 섬세한 예외 처리가 필요한 코드에서는 `Exception 클래스` 만으로는

대응하기 어려우므로, 귀찮다고 무조건 `Exception 클래스`를 사용하는 것은 좋지 않음.


## 예외 던지기


`try~catch` 문으로 예외를 받는다는 것은 어디선가 `예외를 던진다`는 의미임

예외는 `throw 문` 을 이용해서 던짐

다음은 기본적인 형태임

```csharp
try
{
	//...
	throw new Exception("예외를 던집니다.");
}
catch(Exception e)
{
	Console.WriteLine(e.Message);
		// 예외를 던집니다.
}
```

<br/>

다음과 같이 간단한 예외 던지기를 작성할 수 있음

```csharp
static void DoSomething(int arg)
{
    if (arg < 10)
        Console.WriteLine("arg : {0}", arg);
    else
        throw new Exception("arg가 10보다 큽니다");
}
static void Main(string[] args)
{
    try
    {
        DoSomething(13);
    }
    catch (Exception e)
    {
        Console.WriteLine(e.Message);
    }
}
```

### throw 식
***

C# 7.0부터는 throw식도 가능해짐

```csharp
static void Main(string[] args)
{
    try
    {
        int? a = null;
        int b = a ?? throw new ArgumentException("옳지 않은 값입니다.");
    }
    catch (ArgumentException e)
    {
        Console.WriteLine(e.Message);
        //옳지 않은 값입니다.
    }
}
```

## try~catch와 finally


`try~catch` 에서 예외가 발생할 경우 나머지 코드를 마저 실행하지 않고

바로 catch절으로 넘어오게 됨.

이런 문제는 만들어둔 코드가 작동되지 않는 버그를 초래함.

```csharp
try
{
    int? a = null;
    int b = a ?? throw new ArgumentException("옳지 않은 값입니다.");

    Console.WriteLine("출력되지 않습니다"); // 이 부분이 실행되지 못함
}
catch (ArgumentException e)
{
    Console.WriteLine(e.Message);
    //옳지 않은 값입니다.
}
```

<br>

이럴 때 finally를 통해 뒷 마무리를 처리할 수 있다.

finally문은 `예외가 발생하든 안하든  try문이 실행되기만 하면 finally문도 실행`되며,
심지어 `try문에서 return나 throw을 하여도 실행`된다.

 (return과 throw 이 두 문장은 프로그램의 흐름 제어를 외부 코드로 옮김)

```csharp
static void Main(string[] args)
{
    try
    {
        int? a = null;
        int b = a ?? throw new ArgumentException("옳지 않은 값입니다.");
				// return; 에러발생이 안나고 리턴하여도 finally는 실행됨
    }
    catch (ArgumentException e)
    {
        Console.WriteLine(e.Message);
        //옳지 않은 값입니다.
    }
    finally // 예외가 발생하든 안하든 마지막에 실행됨
    {
        Console.WriteLine("출력됩니다");
    }
}
```