---
layout: single
title:  "[C#] ReaderWriterLock 구현"
categories: 
    - C Sharp
tag: [c#, multiThread]
published: true

author_profile: true # 옆에뜨는 프로파일

date: 2024-06-18
last_modified_at: 2024-06-18
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

## ReaderWriterLock
ReaderWriterLock은 `읽기 작업과 쓰기 작업을 구분하여 관리`한다. <br>
읽기 작업은 여러 쓰레드가 동시에 접근할 수 있고, 쓰기 작업은 배타적으로 수행된다.<br>

그러므로 `평소(공유 자원을 읽기만 함)`에는 ReaderLock을 사용하여 일반적인 코드처럼 접근하여 사용하지만, `특수한 작업(공유 자원을 수정)`이 있을 경우 그 작업을 WriteLock을 통해 수행하며
이떄는 다른 쓰레드들의 WriteLock과 ReadLock이 모두 일반적인 Lock처럼 `먼저 수행되는 WriteLock작업을 대기`하게 된다.

코드는 다음과 같다.



```cs
internal class Lock
{
    // 32비트를 쪼개서 정보를 담음 [Unused(1)] [WriteThreadId(15)] [ReadCount(16)]
    int _flag = EMPTY_FLAG;

    // flag의 원하는 정보를 가져오거나 넣기 위한 const변수
    const int EMPTY_FLAG = 0x00000000;
    const int WRITE_MASK = 0x7FFF0000;
    const int READ_MASK = 0x0000FFFF;
    const int MAX_SPIN_COUNT = 5000;

    // WriteLock을 중첩하여 사용할 경우 쓸 변수
    int _writeCount = 0;

    public void WriteLock()
    {
        // 동일 쓰레드가 WriteLock을 이미 획득하고 있는지 확인 (중첩하여 사용하는 경우)
        // 동일 쓰레드라면 따로 대기하지 않음 
        int _lockThreadId = (_flag & WRITE_MASK) >> 16;
        if(Thread.CurrentThread.ManagedThreadId == _lockThreadId)
        {
            _writeCount++;
            return;
        }

        // 아무도 WriteLock or ReadLock을 흭득하고 있지 않을 때 , 경합해서 소유권을 얻음
        int desired = (Thread.CurrentThread.ManagedThreadId << 16) & WRITE_MASK;
        while (true)
        {
            for (int i = 0; i < MAX_SPIN_COUNT; i++)
            {
                // 시도해서 성공하면 return
                if(Interlocked.CompareExchange(ref _flag, desired, EMPTY_FLAG) == EMPTY_FLAG)
                {
                    _writeCount = 1;
                    return;
                }
  
            }

            Thread.Yield();
        }
    }

    public void WriteUnlock()
    {
        int lockCount = --_writeCount;
        if (lockCount == 0)
            Interlocked.Exchange(ref _flag, EMPTY_FLAG);
    }
    public void ReadLock()
    {
        // 동일 쓰레드가 WriteLock을 이미 획득하고 있는지 확인
        int _lockThreadId = (_flag & WRITE_MASK) >> 16;
        if (Thread.CurrentThread.ManagedThreadId == _lockThreadId)
        {
            Interlocked.Increment(ref _flag);
            return;
        }

        // 아무도 WriteLock을 흭득하고 있지 않으면, ReadCount를 1 획득
        // 누군가 WriteLock을 획득하고 있다면 여기서 대기
        while (true)
        {
            for (int i = 0; i < MAX_SPIN_COUNT; i++)
            {
                int expected = (_flag & READ_MASK);
                // WriteLock을 작업하고 있는 쓰레드가 없음 && 예상하는 READ 수와 같음 -> 통과하여 READ COUNT를 증가
                if (Interlocked.CompareExchange(ref _flag, expected + 1, expected) == expected)
                    return;
            }

            Thread.Yield();
        }
    }
    public void ReadUnlock()
    {
        Interlocked.Decrement(ref _flag);
    }
}
```


## 사용 예시
```cs
internal class Program
{
    static Lock _lock = new Lock();
    static int _number = 100;

    static void ReadThread()
    {
        while (true)
        {
            // 평소에는 그냥 접근하여 출력
            // 누군가 WriteLock을 실행하면 대기
            _lock.ReadLock();
            Console.WriteLine(_number);
            _lock.ReadUnlock();
        }
    }

    static void WriteThread()
    {
        // _number의 값을 바꾸고 5000만큼 잠깐 쉬었다가 WriteLock을 품
        // 중복하여 사용하여도 됨
        //_lock.WriteLock();
        _lock.WriteLock();
        _number += 100;
        Console.WriteLine("값 수정됨");
        Thread.Sleep(5000);
        _lock.WriteUnlock();
        //_lock.WriteUnlock();
    }


    static void Main(string[] args)
    {
        Task t1 = new Task(ReadThread);
        Task t2 = new Task(WriteThread);
        t1.Start();

        Thread.Sleep(2000);
        t2.Start();

        Task.WaitAll(t1,t2);
    }
}
```
<br>


계속 숫자를 출력하다가 WriteThread에서 WriteLock을 수행하는 순간 대기상태가 됨

![image](https://github.com/novicehog/comments/assets/131991619/1d4819dc-76fe-4f72-98b5-b2f70f4d00fd)
<br>

5000만큼 흐른 뒤 WriteLock이 풀려서 다시 값을 출력함

![image](https://github.com/novicehog/comments/assets/131991619/e9212a85-8298-4cce-a2ef-e375b248891f)


## 결론
공유자원을 그냥 단순히 읽기만 하는 작업의 경우 공유자원에 대한 접근을 필요할 때만 Lock을 걸어 더 나은 성능을 가져갈 수 있다.

이때는 ReaderWriterLock을 이용할 수 있으며, 원리를 파악하고자 간단하게 구현해 볼수 있었다.