---
layout: single
title:  "[C#] 원자성"
categories: 
    - C Sharp
tag: [c#, multiThread]
published: true

author_profile: true # 옆에뜨는 프로파일

date: 2024-06-17
last_modified_at: 2024-06-17
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
이 글은 Rookiss님의  **[C#과 유니티로 만드는 MMORPG 게임 개발 시리즈] Part4: 게임 서버**를 보고 헷갈리는 부분 위주로 정리한 글입니다. 틀리거나 부족한 부분이 있을 수 있습니다.
{: .notice--warning}

## 원자성이란
원자성이란 더 이상 쪼개 질수 없는, 프로그래밍에서는 즉, 작업의 최소 단위를 의미한다. <br>
만약 어떠한 작업이 원자성을 보장한다면, 이 작업은 `하나의 단위`로 처리된다.

먼저 멀티 쓰레드 환경에서 원자성이 보장되지 않아 문제가 발생하는 경우를 보자.

### 원자성 경합 조건 문제

다음의 코드는 두 쓰레드가 같은 공유 변수 `number`에 접근하며 각각 1000000번 더하거나 뺀다.

문제가 없이 정상적으로 작동한다면 마지막 출력의 결과는 0이 되어야 한다.

```cs
internal class Program
{
    static int number = 0;

    static void Thread_1()
    {
        for (int i = 0; i < 1000000; i++)
        {
            number++;
        }
    }

    static void Thread_2()
    {
        for (int i = 0; i < 1000000; i++)
        {
            number--;
        }
    }

    static void Main(string[] args)
    {
        Task t1 = new Task(Thread_1);
        Task t2 = new Task(Thread_2);
        t1.Start();
        t2.Start();

        Task.WaitAll(t1,t2);

        Console.WriteLine(number);
    }
}
```
<br>

하지만 결과는 다음과 같이 예상과 다른 값이 나온다.

![경합 문제](https://github.com/novicehog/comments/assets/131991619/59429d0b-d08c-415b-8b5e-1fdd0c2b853c)

<br>

이러한 문제가 발생하는 이유는 ++연산과 --연산의 원자성이 보장되지 않았기 때문이다.

++과 --연산은 단순히 보기에는 그저 더하기 뺴기로 이루어진 최소 단위의 작업으로 보일 수 있으나, 사실은 다음과 같은 과정은 거친다.

1. number값을 기존 값을 가져옴
2. 1을 더하거나 뺸다
3. 결과를 다시 number에 저장한다.

이러한 3단계의 과정을 거치는데 문제는 멀티 쓰레드 환경에서는 저 `작업이 끝나기 전에 다른 스레드에서 number에 접근`하여 다른 작업을 수행하는 경우가 발생한다.

위 코드에서 예시를 본다면  ++ 작업이 완료되기 전에 --작업이 시작된다면 ++작업은 의미가 없는 작업이 된다. 그렇기 대문에 코드의 결과값이 0이 아닌 엉뚱한 값이 나오게 된다.

이러한 문제를 해결하기 위해 원자성을 보장하는 기능이 필요하다.

### Interlocked
Interlocked라는 클래스를 이용하면 ++, --연산을 원자성을 보장하여 처리한다.

만약 두 쓰레드에서 동시에 작업을 수행한다 하여도, 결국 두 작업 중 먼저 처리되는 작업이 있을 것이고, 원자성이 보장되는 작업이기에
해당 작업은 하나의 작업으로 처리되어 `이 작업이 끝나야만 다른 나머지 작업도 시작될 수 있다.`

```cs
internal class Program
{
    static int number = 0;

    static void Thread_1()
    {
        for (int i = 0; i < 1000000; i++)
        {
            // ++의 원자성을 보장하는 함수
            Interlocked.Increment(ref number);
        }
    }

    static void Thread_2()
    {
        for (int i = 0; i < 1000000; i++)
        {
            // --의 원자성을 보장하는 함수
            Interlocked.Decrement(ref number);
        }
    }

    static void Main(string[] args)
    {
        Task t1 = new Task(Thread_1);
        Task t2 = new Task(Thread_2);
        t1.Start();
        t2.Start();

        Task.WaitAll(t1,t2);

        Console.WriteLine(number);
    }
}
```

![문제 해결](https://github.com/novicehog/comments/assets/131991619/c1573f29-5385-4f37-a439-fdd70a867e05)





### Monitor 클래스 이용하기
Interlocked클래스는 ++이나 --같은 미리 정해진 작업을 원자성을 보장한다면, Monitor클래스는 내가 `직접 원자성을 보장할 영역을 지정`한다.

Monitor.Enter(object)로 영역의 시작을 알리고 Monitor.Exit(object)로 영역의 끝을 알린다.<br>
여기서 인자로 넣는 object는 쉽게 말해서 자물쇠이다. `동일한 오브젝트에 대해 lock`을 걸면, 해당 오브젝트를 사용하는 다른 스레드는 `락이 해제될 때까지 대기`하게 된다.

예제는 다음과 같다.

```cs
internal class Program
{

    static int number = 0;
    static object _obj = new object();

    static void Thread_1()
    {
        for (int i = 0; i < 1000000; i++)
        {
            Monitor.Enter(_obj);

            number++;

            Monitor.Exit(_obj);
        }
    }

    static void Thread_2()
    {
        for (int i = 0; i < 1000000; i++)
        {
            Monitor.Enter(_obj);

            number--;

            Monitor.Exit(_obj);
        }
    }

    static void Main(string[] args)
    {
        Task t1 = new Task(Thread_1);
        Task t2 = new Task(Thread_2);
        t1.Start();
        t2.Start();

        Task.WaitAll(t1,t2);

        Console.WriteLine(number);
    }
}
```
<br>

주의해야할 점은 Enter다음에 Exit를 하지 않는 경우인데, Exit를 하지 않는다면 다른 작업들은 무한정 Enter함수에서 기다리게 되어 `DeadLock` 현상이 발생한다.<br>

```cs
internal class Program
{
    static int number = 0;
    static object _obj = new object();

    static void Thread_1()
    {
        for (int i = 0; i < 1000000; i++)
        {
            Monitor.Enter(_obj);

            number++;
            
            // 데드락이 발생하는 조건문
            if (number > 500)
                return;

            Monitor.Exit(_obj);
        }
    }

    static void Thread_2()
    {
        for (int i = 0; i < 1000000; i++)
        {
            // 데드락이 걸려서 무한정 기다리게 됨
            Monitor.Enter(_obj);

            number--;

            Monitor.Exit(_obj);
        }
    }

    static void Main(string[] args)
    {
        Task t1 = new Task(Thread_1);
        Task t2 = new Task(Thread_2);
        t1.Start();
        t2.Start();

        Task.WaitAll(t1,t2);

        Console.WriteLine(number);
    }
}
```

<br>


이러한 문제는 Try Finally을 사용하여 예방할 수 있다.


```cs
static void Thread_1()
{
    for (int i = 0; i < 1000000; i++)
    {
        Monitor.Enter(_obj);

        try
        {
            number++;
            if (number > 500)
                return;
        }
        finally
        {
            Monitor.Exit(_obj);
        }
    }
}

static void Thread_2()
{
    for (int i = 0; i < 1000000; i++)
    {
        Monitor.Enter(_obj);

        try
        {
            number--;
                return;
        }
        finally
        {
            Monitor.Exit(_obj);
        }
    }
}
```


### lock
Monitor클래스의 try Finally이 지저분하게 보인다면 lock키워드로 간단하게 구현할 수 있다.

이런 식으로 코드를 작성하면 `원자성을 보장하며 데드락을 방지`할 수 있다.

```cs
internal class Program
{
    static int number = 0;
    static object _obj = new object();

    static void Thread_1()
    {
        for (int i = 0; i < 1000000; i++)
        {
            // 데드락 방지
            lock (_obj)
            {
                number++;
                if (number > 500)
                    return;
            }
        }
    }

    static void Thread_2()
    {
        for (int i = 0; i < 1000000; i++)
        {
            lock (_obj)
            {
                number--;
            }
        }
    }

    static void Main(string[] args)
    {
        Task t1 = new Task(Thread_1);
        Task t2 = new Task(Thread_2);
        t1.Start();
        t2.Start();

        Task.WaitAll(t1,t2);

        Console.WriteLine(number);
    }
}
```

