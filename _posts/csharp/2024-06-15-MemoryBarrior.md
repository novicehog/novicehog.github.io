---
layout: single
title:  "[C#] 메모리 베리어"
categories: 
    - C Sharp
tag: [c#]
published: true

author_profile: true # 옆에뜨는 프로파일

date: 2024-06-15
last_modified_at: 2024-06-16
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

## 멀티 쓰레드에서 하드웨어적 문제
싱글 쓰레드 환경에서는 문제되지 않던 코드가 `멀티 쓰레드 환경에서 문제가 발생`하는 경우는 매우 잦다.<br>
이중 한 가지는 바로 하드웨어적 최적화와 관련되어 있다. <br>

먼저 다음의 코드를 보자.
밑의 코드는 x y r1 r2의 정적 변수들을 가지고 있으며 각각 두 개의 쓰레드에서 `y에 1을 Store하고 x를 Load하여 r1에 저장`, `x에 1을 Store하고 y를 Load하여 r2에 저장`의 기능을 수행한다.

메인 함수에서는 무한 루프를 돌며 두 쓰레드를 실행하고 r1과 r2모두 0일 경우 루프를 빠져나온다.

```cs
internal class Program
{
    static int x = 0;
    static int y = 0;
    static int r1 = 0;
    static int r2 = 0;

    static void Thread_1()
    {
        y = 1;          // Store y

        r1 = x;         // Load x
    }

    static void Thread_2()
    {
        x = 1;          // Store x

        r2 = y;         // Load y
    }

    static void Main(string[] args)
    {
        int count = 0;
        while (true)
        {
            count++;
            x = y = r1 = r2 = 0;

            Task t1 = new Task(Thread_1);
            Task t2 = new Task(Thread_2);
            t1.Start();
            t2.Start();

            Task.WaitAll(t1, t2);

            if (r1 == 0 && r2 == 0)
                break;
        }

        Console.WriteLine($"{count}번 만에 빠져나옴");
    }
}
```
<br>

이 코드에서 경우의 수를 따져봐도 r1과 r2가 동시에 0이되는 상황은 보이지 않는다. 그러므로
정상적인 상황이라면 메인 함수는 영원히 끝나지 않을 것이다.

하지만 실행해보면 그냥 끝나버리게 된다.


![image](https://github.com/novicehog/comments/assets/131991619/32080ba8-fc8b-4ab4-8609-cb422a4e97b0)

이러한 문제가 발생하는 이유는 이제 `CPU가 의존성이 없어보이는 명령어는 최적화를 위해 순서를 뒤바꿔서 실행`하게 된다.

위 코드의 경우에는 두 개의 쓰레드가 있는데, CPU가 보기에 각 쓰레드의 코드들은 서로 의존성이 없다고 판단되어 순서가 뒤바껴 실행이 될 수 있다.

그래서 만약 이런 식으로 뒤집히게 되고
```cs
// 순서가 마음대로 바뀜
static void Thread_1()
{
    r1 = x;         // Load x
    y = 1;          // Store y
}

static void Thread_2()
{
    r2 = y;         // Load y
    x = 1;          // Store x
}
```

쓰레드의 명령어가
R1 = X <br>
R2 = Y <br>
X = 1 <br>
Y = 1 <br>

이런 순서로 실행된다면 R1과 R2가 동시에 0이되어 루프를 빠져나가게 된다.

### 메모리 베리어 사용
이렇게 순서가 뒤집히는 문제는 메모리 베리어를 사용함으로 예방할 수 있다.

```cs
internal class Program
{
    static int x = 0;
    static int y = 0;
    static int r1 = 0;
    static int r2 = 0;

    static void Thread_1()
    {
        y = 1;          // Store y

        // 메모리 베리어
        Thread.MemoryBarrier();

        r1 = x;         // Load x
    }

    static void Thread_2()
    {
        x = 1;          // Store x

        // 메모리 베리어
        Thread.MemoryBarrier();

        r2 = y;         // Load y
    }

    static void Main(string[] args)
    {
        int count = 0;
        while (true)
        {
            count++;
            x = y = r1 = r2 = 0;

            Task t1 = new Task(Thread_1);
            Task t2 = new Task(Thread_2);
            t1.Start();
            t2.Start();

            Task.WaitAll(t1, t2);

            if (r1 == 0 && r2 == 0)
                break;
        }

        Console.WriteLine($"{count}번 만에 빠져나옴");
    }
}
```

이렇게 하면 루프를 빠져나가지 못하고 영원히 순회한다.

### 메모리 베리어의 가시성
메모리 베리어는 코드 재배치를 억제하기도 하지만 `가시성`을 보장하기도 한다.

여기서 말하는 가시성이란 `공유 변수`에 대한 `스레드 간의 접근 문제`를 말한다.

메모리 베리어가 실행될 때, 공유 변수값을 수정하였다면 수정한 값을 저장하고, 공유 변수가 다른 스레드에서 수정 되었다면
그 수정된 값을 받아오게 된다. 그러므로 `여러 스레드간에 공유 변수가 같은 값`을 가지게 된다.

위에 예시로 든 코드의 쓰레드 부분을 보면

먼저 Thread_1에서 y값이 1로 수정되고 메모리 베리어가 호출된다.
이렇게 되면 y값이 바뀐 내용이 실제로 메모리에 입력되며, 또한 x값이 바뀌었다면 그 값도 갱신하므로
바로 밑에서 x가 사용될 때 최신의 x값이 사용되게 된다.

```cs
static void Thread_1()
{
    y = 1;          // Store y

    // 메모리 베리어
    // y값이 1로 변함을 메모리에 저장
    // 동시에 최신의 x값을 가져옴
    Thread.MemoryBarrier();

    r1 = x;         // Load x
}

static void Thread_2()
{
    x = 1;          // Store x

    // 메모리 베리어
    // x값이 1로 변함을 메모리에 저장
    // 동시에 최신의 y값을 가져옴
    Thread.MemoryBarrier();

    r2 = y;         // Load y
}
```

## 결론
메모리 베리어가 하는 역할은 크게 두 가지다.
1. 코드 재배치를 억제한다.
2. 가시성(공유 변수간의 접근 문제)을 보장한다.