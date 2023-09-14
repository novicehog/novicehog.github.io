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


## 사용자 정의 예외 클래스 만들기

System.Exception 클래스를 상속 받아서  사용자 정의 예외 클래스를 만들 수 있다.

```csharp
class MyException : Exception
{
	//...
}
```

사용자 정의 예외는 꼭 필요하지 않으나 다음과 같은 상황에 유용하게 사용할 수 있음

- 특별한 데이터를 담아서 예외 처리 루틴에 추가 정보를 제공하고 싶을 때
- 예외 상황을 더 잘 설명하고 싶을 때 사용

다음은 사용자 예외 처리의 예시이다.

 0~255 사이의 값인 ARGB를 받아서 0~255사이의 값이 아닐 경우

직접 만든 예외가 발생하도록 하는 코드이다.

```csharp
class InvalidArgumentException : Exception
{
    public InvalidArgumentException()
    {
    }

    public InvalidArgumentException(string message) : base(message) 
    {
    }

    public object Argument
    {
        get;
        set;
    }

    public string Range
    {
        get;
        set;
    }
}

internal class Program
{
		// 정적 메소드 아무곳에서나 호출
    static uint MergeARGB(uint alpha, uint red, uint green, uint blue) 
    {
        uint[] args = new uint[] { alpha, red, green, blue };

        foreach(uint arg in args)
        {
            if (arg > 255)
			    // 받은 값이 255 이상이면 예외를 던지고
				// 정보를 저장함
                throw new InvalidArgumentException()
                {
                    Argument = arg,
                    Range = "0~255"
                };
        }
				
		// 예외 던짐이 없을 시 받은 값을 리턴해줌
        return  (alpha << 24 & 0xFF000000) |
                (red   << 16 & 0x00FF0000) |
                (green << 8  & 0x0000FF00) |
                (blue        & 0x000000FF);
    }
    static void Main(string[] args)
    {
        try
        {
            Console.WriteLine("0x{0:X}", MergeARGB(255, 111, 111, 111));
            Console.WriteLine("0x{0:X}", MergeARGB(1, 65, 192, 128));
            Console.WriteLine("0x{0:X}", MergeARGB(0, 255, 255, 300)); //예외 발생
        }
        catch(InvalidArgumentException e)
        {
			// 예외 발생시 직접 저장한 값을 같이 출력해줌
            Console.WriteLine(e.Message);
            Console.WriteLine($"Argument:{e.Argument}, Range:{e.Range}");
        }
    }
}

//결과값
//0xFF6F6F6F
//0x141C080
//Exception of type 'InvalidArgumentException' was thrown.
//Argument:300, Range:0~255
```


## 예외 필터하기

C# 6.0부터는 catch 절이 받아들일 예외 객체에 `제약 사항을 명시`해서 해당 `조건을 만족하는 예외 객체에 대해서만` 예외 처리 코드를 실행할 수 있도록 하는  `예외 필터`가 도입되었음.

만드는 방법은 간단하게 catch절 뒤에 when 키워드를 이용해서 제약 조건을 기술할 수 있음.

즉, 예외 발생에 대한 catch실행을 세분화 할 수 있음.

다음 코드는 0~10의 값의 숫자만 받도록 설계된 간단한 코드이다.

```csharp
class MyException : Exception
{
    public int ErrorNo { get; set;}
}

internal class Program
{
    static void Main(string[] args)
    {
        Console.WriteLine("Enter Number Between 0~10");
        string input = Console.ReadLine();
        try
        {
            int num = Int32.Parse(input);

            if (num < 0 || num > 10)
                throw new MyException() { ErrorNo = num };
            else
                Console.WriteLine($"Output : {num}");
        }
        catch (FormatException e) // 입력 값이 숫자가 아닐 경우
        {
            Console.WriteLine("숫자 변환이 불가능합니다" + e);
        }
        catch (MyException e) when (e.ErrorNo < 0) // 입력 값이 0 이하일 경우
        {
            Console.WriteLine("음수는 불가능합니다");
        }
        catch (MyException e) when (e.ErrorNo > 10) // 입력 값이 10 초과일 경우
        {
            Console.WriteLine("너무 큰 숫자는 불가능합니다");
        }
    }
}
```