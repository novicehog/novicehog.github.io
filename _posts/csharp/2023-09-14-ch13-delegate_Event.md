---
layout: single
title:  "C# Chapter13 delegate 대리자와 event 이벤트"
categories: 
    - C Sharp
tag: [c#]
published: true

author_profile: true # 옆에뜨는 프로파일

date: 2023-09-14
last_modified_at: 2023-10-06
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
함수를 사용할 때 매개변수로 `값`을 넘기는 것 아니라 `함수` 자체를 넘길 수 있다. <br>

다음 코드의 경우 오름차순 비교 함수와 내림차순 비교 함수 두가지가 있다. <br>
이 둘과 같은 인자값을 갖는 delegate를 인자값으로 갖는 BubbleSort함수에 <br>
어떤 함수를 넣어주냐에 따라 다른 결과가 나온다. <br>

```c#
delegate int Compare(int a, int b);
internal class Program
{

    static int AscendCompare(int a, int b)
    {
        if (a > b)
        {
            return 1;
        }
        else if (a == b)
            return 0;
        else
            return -1;
    }

    static int DescendCompare(int a, int b)
    {
        if (a < b)
            return 1;
        else if (a == b)
            return 0;
        else return -1;
    }

    static void BubbleSort(int[] DataSet, Compare Comparer)
    {
        int i = 0;
        int j = 0;
        int temp = 0;

        for (i = 0; i < DataSet.Length -1; i++)
        {
            for (j = 0; j < DataSet.Length - (i + 1); j++)
            {
                if (Comparer(DataSet[j] , DataSet[j+1]) > 0)
                {
                    temp = DataSet[j + 1];
                    DataSet[j+1] = DataSet[j];
                    DataSet[j] = temp;
                }
            }
        }
    }

    static void Main(string[] args)
    {
        int[] array = { 3, 7, 4, 2, 10 };

        Console.WriteLine("Sorting ascending..");
        BubbleSort(array, new Compare(AscendCompare));

        for (int i = 0; i < array.Length; i++)
            Console.Write($"{array[i]} ");

        int[] array2 = { 7, 2, 8, 10, 11 };
        Console.WriteLine("\nSorting descending...");
        BubbleSort(array2, new Compare(DescendCompare));

        for (int i = 0; i < array2.Length; i++)
            Console.Write($"{array2[i]} ");

        Console.WriteLine();
    }
}

```

<br>

## 일반화 대리자
대리자는 일반화(generic)형식의 메소드도 참조할 수 있다.

```c#
delegate int Compare<T>(T a, T b); 
```

<br>

위에서 예시로든 코드를 일반화 대리자로 바꾸면 다음과 같다.

이때 여기서 보이는 CompareTo는  <br>
모든 수치 형식과 string이 IComparable을 상속해서 구현하고있는 메소드로써 <br>
자신보다 큰 값이 인자로 들어오면 -1 , 같으면 0, 작으면 1을 반환한다. <br>

```c#
delegate int Compare<T>(T a, T b);
internal class Program
{

    static int AscendCompare<T>(T a, T b) where T : IComparable<T>
    {
        return a.CompareTo(b);
    }

    static int DescendCompare<T>(T a, T b) where T : IComparable<T>
    {
        return a.CompareTo(b) * -1;
    }

    static void BubbleSort<T>(T[] DataSet, Compare<T> Comparer)
    {
        int i = 0;
        int j = 0;
        T temp;

        for (i = 0; i < DataSet.Length -1; i++)
        {
            for (j = 0; j < DataSet.Length - (i + 1); j++)
            {
                if (Comparer(DataSet[j] , DataSet[j+1]) > 0)
                {
                    temp = DataSet[j + 1];
                    DataSet[j+1] = DataSet[j];
                    DataSet[j] = temp;
                }
            }
        }
    }

    static void Main(string[] args)
    {
        int[] array = { 3, 7, 4, 2, 10 };

        Console.WriteLine("Sorting ascending..");
        BubbleSort(array, new Compare<int>(AscendCompare));

        for (int i = 0; i < array.Length; i++)
            Console.Write($"{array[i]} ");

        string[] array2 = { "abc", "def", "ghi", "jkl", "mno" };
        Console.WriteLine("\nSorting descending...");
        BubbleSort(array2, new Compare<string>(DescendCompare));

        for (int i = 0; i < array2.Length; i++)
            Console.Write($"{array2[i]} ");

        Console.WriteLine();
    }
}
```

<br>


## 대리자 체인

대리자는 `여러개의 메소드를 동시에 참조`할 수 있다.

`+=`를 통해 메소드를 여러개 넣어주면 된다.

```c#
internal class Program
{
    delegate void ThereIsAFire(string location);

    static void Call119(string location)
    {
        Console.WriteLine("소방서죠? 불났어요! 주소는 {0}", location);
    }
    static void ShotOut(string location)
    {
        Console.WriteLine("피하세요! {0}에 불이 났어요!", location);
    }
    static void Escape (string location)
    {
        Console.WriteLine("{0}에서 나갑시다!", location);
    }

    static void Main(string[] args)
    {
        ThereIsAFire Fire = new ThereIsAFire(Call119);
        Fire += new ThereIsAFire(ShotOut);
        Fire += new ThereIsAFire(Escape);

        Fire("우리집");
        //소방서죠? 불났어요! 주소는 우리집
        //피하세요! 우리집에 불이 났어요!
        //우리집에서 나갑시다!
    }
}
```

<br>

`-=`을 통해 체인을 끊어내는 것 역시 가능하다.



## 익명 메소드

익명 메소드는 이름이 없는 메소드를 뜻한다.

익명 메소드는 다음과 같이 선언한다.

```c#
대리자_인스턴스 = delegate (매개변수_목록)
                {
                    // 실행하려는 코드 ...
                }
```

<br>

예시는 다음과 같다.

```c#
delegate int Calculate(int a, int b);

public static void Main()
{
    Calculate Calc;
    Calc = delegate (int a, int b)
    {
        return a + b;
    };

    Console.WriteLine("3 + 4 : {0}", Calc(3, 4));
}
```

<br>

익명 메소드는 자신이 참조할 대리자의 형식과 동일해야 한다. <br>
`반환 형식`과 `매개변수 형식, 개수` 같은 것들을 모두 맞춰줘야한다.

사용 예시

t
```c#
internal class Program
{
    delegate int Compare<T>(T a, T b);
    static void BubbleSort<T>(T[] DataSet, Compare<T> Comparer)
    {
        int i = 0;
        int j = 0;
        T temp;

        for (i = 0; i < DataSet.Length - 1; i++)
        {
            for (j = 0; j < DataSet.Length - (i + 1); j++)
            {
                if (Comparer(DataSet[j], DataSet[j + 1])> 0)
                {
                    temp = DataSet[j + 1];
                    DataSet[j + 1] = DataSet[j];
                    DataSet[j] = temp;
                }
            }
        }
    }
    public static void Main(string[] args)
    {
        int[] array = { 3, 7, 4, 2, 10 };

        Console.WriteLine("Sorting ascending...");
        BubbleSort<int>(array, delegate (int a, int b) // 익명 메소드
        {
            if (a > b)
                return 1;
            else if (a == b)
                return 0;
            else
                return -1;
        });

        for (int i = 0; i < array.Length; i++)
            Console.Write($"{array[i]} ");

        int[] array2 = { 7, 2, 8, 10, 11 };
        Console.WriteLine("\nSorting descending...");
        BubbleSort(array2, delegate (int a, int b) // 익명 메소드
        {
            if (a < b)
                return 1;
            else if (a == b)
                return 0;
            else
                return -1;
        });

        for (int i = 0; i < array2.Length; i++)
            Console.Write($"{array2[i]} ");
        
        Console.WriteLine();
    }
}

```

<br>

## event 이벤트 : 객체에 일어난 사건 알리기

다음은 이벤트를 이용하여 3의 배수마다 호출을 하는 예시이다

```c#
delegate void EventHandler(string message);

class MyNotifier
{
    public event EventHandler SomethingHappend;

    public void DoSomething(int number)
    {
        int temp = number % 10;

        if (temp != 0 && temp % 3 == 0)
        {
            SomethingHappend(String.Format("{0} : 짝", number));
        }
    }
}

internal class Program
{
    static public void MyHandler(string message)
    {
        Console.WriteLine(message);
    }

    public static void Main(string[] args)
    {
        MyNotifier notifier = new MyNotifier();
        notifier.SomethingHappend += new EventHandler(MyHandler);

        for (int i = 0; i < 30; i++)
        {
            notifier.DoSomething(i);
        }
    }
}
```

<br>

사실 이벤트는 대리자의 한 종류로 대리자와 똑같이 사용된다.

유일한 차이로는 이벤트는 public으로 선언되어도 `외부에서 직접 호출될 수 없다`는 점이다.
```c#
public static void Main(string[] args)
{
    MyNotifier notifier = new MyNotifier();
    notifier.SomethingHappend += new EventHandler(MyHandler);
    notifier.SomethingHappend("test"); // event는 이곳에서 호출될 수 없음

    for (int i = 0; i < 30; i++)
    {
        notifier.DoSomething(i);
    }
}
```

<br>

따라서 대리자는 대리자대로 `콜백 용도`로 사용하고, <br>
이벤트는 이벤트대로 `객체의 상태 변화나 사건의 발생을 알리는 용도`로 구분하여 사용해야함