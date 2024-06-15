---
layout: single
title:  "[C#] 메모리 베리어"
categories: 
    - C Sharp
tag: [c#]
published: true

author_profile: true # 옆에뜨는 프로파일

date: 2024-06-15
last_modified_at: 2024-06-15
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

## 메모리 베리어
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