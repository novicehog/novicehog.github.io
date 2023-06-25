---
layout: single
title:  "C# Chapter 10 array, collection, indexer"
categories: 
    - C Sharp
tag: [c#]
published: true

author_profile: true # 옆에뜨는 프로파일

date: 2023-06-25
last_modified_at: 2023-06-25
---
이 글은 한빛미디어의 **이것이 C#이다**를 보고 헷갈리는 부분 위주로 정리한 글입니다.
{: .notice--warning}




## 배열의 선언
{: .H2-font}

```csharp
데이터_형식[] 배열_이름 = new 데이터_형식[용량];

// 용량이 5개인 int 형식의 배열
int[] scores = new int[5];

// 주의할 점은 배열의 인덱스는 0부터 시작
scores[0];
scores[1];
scores[2];
scores[3];
scores[4];
//scores[5]; 존재하지 않음, 호출시 에러 발생
```





### ^ 연산자 ( C# 8.0 이상) System.Index
{: .H3-font}

^연산자를 이용하여 컬렉션의 마지막부터 역순으로 인덱스를 지정하는 기능.

```csharp
int[] scores = new int[5] {0,1,2,3,4};
Console.WriteLine(scores[^1]); // 4 출력
// scores[scores.Length-1]과 같다고 볼 수 있음
```



### 배열의 초기화 세가지 방법
{: .H3-font}

```csharp
int[] scores1 = new int[3] { 0, 1, 2 }; // 정석
int[] scores2 = new int[] { 0, 1, 2 }; // 용량을 생략
int[] scores3 = { 0, 1, 2 }; // new 연산자, 형식, 용량 모두 생략
```



## System.Array
{: .H2-font}



### 정적 메소드
{: .H3-font}

**`Sort()`** : 배열을 정렬합니다.

**`BinarySearch<T>()`** : 이진 탐색을 수행합니다.

**`IndexOf()`** : 배열에서 찾고자 하는 특정 데이터의 인덱스를 반환합니다.

**`TrueForAll<T>()`** : 배열의 모든 요소가 지정한 조건에 부합하는지 여부를 반환합니다. 

trueforall 메소드는 배열과 함께 조건을 검사하는 메소드를 매개변수를 받습니다.

**`FindIndex<T>()`** : 특정 조건에 부합하는 첫 번째 요소의 인덱스를 반환합니다.

**`Resize<T>()`** : 배열의 크기를 재조정합니다.

**`Clear()`** : 배열의 모든 요소를 초기화합니다. (숫자 형식이면 모두 0으로 논리 형식이면 모두 false로 참조 형식이면 null 로 초기화)

**`ForEach<T>()`** : 배열의 모든 요소에 대해 동일한 작업을 수행하게 합니다.

**`Copy<T>()`** : 배열의 일부를 다른 배열에 복사합니다.

### 인스턴스 메소드
{: .H3-font}

**`GetLength()`** : 배열에서 지정한 차원의 길이를 반환합니다. (다차원 배열에서 유용)

### 프로퍼티
{: .H3-font}

**`Length`** : 배열의 길이 반환

**`Rank`** : 배열의 차원 반환

예제

```csharp
private static bool CheckPassed(int score)
{
    return score >= 60;
}

private static void Print(int value)
{
    Console.Write($"{value} ");
}
static void Main(string[] args)
{
    int[] scores = new int[] { 80, 74, 81, 90, 34 };

    foreach (int score in scores)
    {
        Console.Write($"{score} ");
    }
    Console.WriteLine();

    Array.Sort(scores);
    // 배열의 모든 요소에 대해 동일한 작업을 수행하게 함
    Array.ForEach<int>(scores, new Action<int>(Print));
    Console.WriteLine();

    Console.WriteLine($"Number of dimensions : {scores.Rank}"); // 배열의 차원 반환

    Console.WriteLine($"Binary Search : 81 is at " +
        $"{Array.BinarySearch<int>(scores, 81)}"); // 이진 탐색을 수행

    Console.WriteLine($"Linear Search : 90 is at " +
        $"{Array.IndexOf<int>(scores, 90)}");
    // 배열에서 찾고자 하는 특정 데이터의 인덱스를 반환

    Console.WriteLine($"Everyone passed ? : " +
        $"{Array.TrueForAll<int>(scores, CheckPassed)}"); // 메소드를 매개변수로 받음
                                                          // 배열의 모든 요소가 지정한 조건에 부합하는지 여부를 반환

    // 특정 조건에 부합하는 첫 번째 요소의 인덱스를 반환합니다.
    int index = Array.FindIndex<int>(scores, (score) => score < 60); // 메소드를 매개변수로 받음

    scores[index] = 61;
    Console.WriteLine($"Everyone passed ? : " +
        $"{Array.TrueForAll<int>(scores, CheckPassed)}");

    Console.WriteLine("Old length of scroes : " +
        $"{scores.GetLength(0)}"); // 배열에서 지정한 차원의 길이를 반환

    Array.Resize<int>(ref scores, 10); // 배열의 크기를 재조정, 여기서는 5 -> 10
    Console.WriteLine($"New length of scores : {scores.Length}");

    Array.ForEach<int>(scores, new Action<int>(Print));
    Console.WriteLine();

    Array.Clear(scores, 3, 7); // 배열의 모든 요소를 초기화 3번째 방부터 7개를 초기화
    Array.ForEach<int>(scores, new Action<int>(Print));
    Console.WriteLine();

    int[] sliced = new int[3];
    Array.Copy(scores, 0, sliced, 0, 3);
    // score의 0부터 3개 요소를 sliced의 0부터 3개 요소에 복사
    Array.ForEach<int>(sliced, new Action<int>(Print));
    Console.WriteLine();
}

// 출력 결과
80 74 81 90 34
34 74 80 81 90
Number of dimensions : 1
Binary Search : 81 is at 3
Linear Search : 90 is at 4
Everyone passed ? : False
Everyone passed ? : True
Old length of scroes : 5
New length of scores : 10
61 74 80 81 90 0 0 0 0 0
61 74 80 0 0 0 0 0 0 0
61 74 80
```

## 배열 분할하기 C# 0.8이상
{: .H2-font}

분할에 대해 알알보기전에 알아둘 것은

System.Index 형식과 함께 도입된 **System.Range**

```csharp
..을 사용하여 배열의 범위를 나타냄 
System.Range range = 1..5; // 1번째 부터 ~ "5번째 전 까지"
// 위 처럼 주의 마지막 인덱스는 배열 분할 결과에서 제외됨

// 예시
int[] a = new int[] { 1, 2, 3, 4, 5, 6, 7, 8, 9 };
foreach (int i in a[range]) // a[1..5] 도 가능
{
		Console.WriteLine(i);
}
// 출력
// 2
// 3
// 4
// 5
```

시작 인덱스와 끝 인덱스를 생략하거나

System.Index 객체를 이용할 수도 있다.

```csharp
System.Range range1 = ..5; // 처음부터 4 까지 
System.Range range2 = 3..; // 3부터 끝 까지 
System.Range range3 = ..; // 처음부터 끝까지 

System.Index idx = ^1;
System.Range range4 = ..idx; // 처음 부터 끝 전 방까지
System.Range range5 = ^4..^1; // 끝에서 4번째 방부터 끝 바로 전 방 까지
```

```csharp
static void PrintArray(System.Array array)
        {
            foreach (var item in array) 
                Console.Write(item);
            Console.WriteLine();
        }
        static void Main(string[] args)
        {
            // 알파벳 개수만큼 만듬
            char[] array = new char['Z' - 'A' + 1]; //ASCII코드로 Z = 90 A = 65 즉 26
            for (int i = 0; i < array.Length; i++)
                array[i] = (char)('A' + i);

            PrintArray(array[..]); // 첨부터 끝까지
            PrintArray(array[5..]); // 5부터 끝까지

            Range range_5_10 = 5..10;
            PrintArray(array[range_5_10]); // 5부터 9까지

            Index Last = ^0; // 끝 + 1 번째
            Range range_5_last = 5..Last;
            PrintArray(array[range_5_last]); // 5부터 끝까지

            PrintArray(array[^4..^1]); // 끝에서 4번째 부터 끝에서 2번째
        }
//ABCDEFGHIJKLMNOPQRSTUVWXYZ
//FGHIJKLMNOPQRSTUVWXYZ
//FGHIJ
//FGHIJKLMNOPQRSTUVWXYZ
//WXY
```

## 2차원 배열
{: .H2-font}

1차원 배열을 원소로 같는 배열

기본적으론 1차원 배열과 선언 형식이 같지만 각 차원의 용량 또는 길이를 콤마(,)로 구분해서

대괄호 사이에 입력해줌.

```csharp
데이터_형식[,] 배열이름 = new 데이터_형식[2차원_길이, 1차원_길이];
```
### 2차원 배열의 초기화
{: .H3-font}

```csharp
int[,] array = new int[2, 3] { { 0, 1, 2 }, { 3, 4, 5 } }; // 기본
int[,] array = new int[,] { { 0, 1, 2 }, { 3, 4, 5 } }; // 배열의 길이 생략
int[,] array ={ { 0, 1, 2 }, { 3, 4, 5 } }; // 형식과 길이 모두 생략
```

예시

```csharp
static void Main(string[] args)
{
    int[,] arr = new int[2, 3] { { 1, 2, 3 }, { 4, 5, 6 } };

    Console.WriteLine("arr");

    for (int i = 0; i < arr.GetLength(0); i++)
    {
        for (int j = 0; j < arr.GetLength(1); j++)
        {
            Console.Write($"[{i}, {j}] : {arr[i, j]} ");
        }
        Console.WriteLine();
    }
    Console.WriteLine();

    int[,] arr2 = new int[,] { { 1, 2, 3 }, { 4, 5, 6 } };

    Console.WriteLine("arr2");

    for (int i = 0; i < arr2.GetLength(0); i++)
    {
        for (int j = 0; j < arr2.GetLength(1); j++)
        {
            Console.Write($"[{i}, {j}] : {arr2[i, j]} ");
        }
        Console.WriteLine();
    }
    Console.WriteLine();

    int[,] arr3 = new int[,] { { 1, 2, 3 }, { 4, 5, 6 } };

    Console.WriteLine("arr3");

    for (int i = 0; i < arr3.GetLength(0); i++)
    {
        for (int j = 0; j < arr3.GetLength(1); j++)
        {
            Console.Write($"[{i}, {j}] : {arr3[i, j]} ");
        }
        Console.WriteLine();
    }
    Console.WriteLine();
}
```

## 다차원 배열
{: .H2-font}
2차원 이상의 배열들 모두가 다차원배열

2차원은 1차원 배열을 원소로 갖는 다차원

3차원은 2차원 배열을 원소로 갖는 다차원

4차원은 3차원 배열을 원소로 갖는 다차원

…

```csharp
static void Main(string[] args)
{
    int[] array1 = new int[2]; // 1차원

    int[,] array2 = new int[3, 2] { { 1, 2 }, { 3, 4 }, { 5, 6 } }; // 2차원

    int[,,] array3 = new int[4, 3, 2] // 3차원
    {
        { { 1, 2}, { 3 ,4}, { 5, 6} },
        { { 1, 2}, { 3 ,4}, { 5, 6} },
        { { 1, 2}, { 3 ,4}, { 5, 6} },
        { { 1, 2}, { 3 ,4}, { 5, 6} }
    };

    int[,,,] array4 = new int[2, 4, 3, 2] // 4차원
    { { 
        { { 1, 2}, { 3 ,4}, { 5, 6} },
        { { 1, 2}, { 3 ,4}, { 5, 6} },
        { { 1, 2}, { 3 ,4}, { 5, 6} },
        { { 1, 2}, { 3 ,4}, { 5, 6} }
    },
    {
        { { 1, 2}, { 3 ,4}, { 5, 6} },
        { { 1, 2}, { 3 ,4}, { 5, 6} },
        { { 1, 2}, { 3 ,4}, { 5, 6} },
        { { 1, 2}, { 3 ,4}, { 5, 6} }
    } };
}
```

## 가변 배열
{: .H2-font}

```csharp
데이터_형식[][] 배열_이름 = new 데이터_형식[가변_배열의_용략][];
```

다른 다차원 배열과 비슷하지만 이 배열은 배열의 요소로 입력되는 배열의 크기가 

다 같을 필요가 없음.

```csharp
int[][] array1 = new int[3][];

    array1[0] = new int[3];
    array1[1] = new int[4];
    array1[2] = new int[5];
```

## 컬렉션 맛보기
{: .H2-font}

### ArrayList
{: .H3-font}

가장 배열과 닮은 컬렉션.

Add, RemoveAt, Insert 등을 사용함.

데이터를 담을때 박싱을, 데이터를 꺼낼때 언박싱을 하여

다양한 형식의 객체를 담을 수 있다. 

```csharp
using System.Collections;

static void Main(string[] args)
{
    ArrayList list = new ArrayList();
    for (int i = 0; i < 5; i++)
        list.Add(i);

    foreach (object obj in list)
        Console.Write($"{obj} ");
    Console.WriteLine();

    list.RemoveAt(0);

    foreach (object obj in list)
        Console.Write($"{obj} ");
    Console.WriteLine();

    list.Insert(2, 2);

    foreach (object obj in list)
        Console.Write($"{obj} ");
    Console.WriteLine();

    list.Add("abc"); // 문자열도 담긴다
    list.Add("def");

    for (int i = 0; i < list.Count; i++)
        Console.Write($"{list[i]}");
    Console.WriteLine();
}
//0 1 2 3 4
//1 2 3 4
//1 2 2 3 4
//12234abcdef
```

### Queue
{: .H3-font}

데이터나 작업을 차례대로 입력해둔 뒤 순서대로 하나씩 꺼내 처리하기 위해 사용됨.

입력은 뒤로만 출력은 앞으로만 가능하다.

Enqueue() 메소드를 통해 입력

Dequeue() 메소드를 통해 출력

예시

```csharp
static void Main(string[] args)
{
    Queue queue = new Queue();
    queue.Enqueue(1);
    queue.Enqueue(2);
    Console.WriteLine(queue.Dequeue()); // 1이 출력됨
    Console.WriteLine(queue.Dequeue()); // 다음으로 입력된 2가 출력됨
}
//1
//2
```

### Stack
{: .H3-font}

stack은 queue와 반대로 나중에 들어온 데이터가

가장 먼저 출력된다. 먼저 입력된 데이터는 가장 나중에 뽑힌다.
push()를 통해 데이터를 넣고 

Pop() 을 통해 데이터를 꺼냄

예시

```csharp
static void Main(string[] args)
{
    Stack stack = new Stack();
    stack.Push(1);
    stack.Push(2);
    Console.WriteLine(stack.Pop());
    Console.WriteLine(stack.Pop());
}
//2
//1
```

### hashtable
{: .H3-font}

키와 값의 쌍으로 이루어진 데이터를 다룰 때 사용

키를 사용하여 검색하여 데이터를 찾기에 인덱스를 이용하여 배열 요소에 접근하는 것에

준하는 탐색 속도를 자랑함.

배열이 인덱스를 사용하는 방식처럼 `[ ]` 사이에 키를 넣어주어 입력, 출력 함

예시                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     

```csharp
static void Main(string[] args)
{
    Hashtable ht = new Hashtable();
    ht["하나"] = 1;
    ht["둘"] = 2;
    ht["셋"] = 3;

    Console.WriteLine(ht["하나"]);
    Console.WriteLine(ht["셋"]);
    Console.WriteLine(ht["둘"]);
}         
```

### 컬렉션을 초기화하는 방법
{: .H3-font}

- arraylist, queue, stack은 배열의 도움을 받아 초기화 가능

```csharp
static void Main(string[] args)
{
    int[] array = { 123, 456, 789 };

    ArrayList list = new ArrayList(array); // 123, 456, 789
    Queue queue = new Queue(array); // 123, 456, 789
    Stack stack = new Stack(array); // 123, 456, 789
}
```

- arratList는 직접 컬렉션 초기자를 이용해 초기화 할 수 있음

```csharp
ArrayList list = new ArrayList() { 123, 456, 789 };
```

- hashtable은 딕셔너리 초기자를 이용하여 초기화, 컬렉션 초기자를 통한 초기화가 있음

```csharp
Hashtable htD = new Hashtable() // 딕셔너리 초기자
{
        ["하나"] = 1,
        ["둘"] = 2,
        ["셋"] = 3
} ;
Hashtable htC = new Hashtable() // 컬렉션 초기자
{
        {"하나", 1 },
        {"둘", 2 },
        {"셋", 3 }
};
```

## 인덱서 Indexer
{: .H2-font}

인덱스를 이용해서 객체 내의 데이터에 접근하게 해주는 프로퍼티라고 생각하면 이해하기 쉬움

객체를 마치 배열처럼 사용할 수 있게 해줌.

```csharp
class MyList
{
    private int[] array;

    public MyList()
    {
        array = new int[3];
    }

    public int this[int index]
    {
        get { return array[index]; }
        set
        {
            if (index >= array.Length)
            {
                Array.Resize<int>(ref array, index + 1);
                Console.WriteLine($"Array Resized : {array.Length}");
            }

            array[index] = value;
        }
    }
    public int Length
    {
        get { return array.Length; }
    }

    internal class Program
    {

        static void Main(string[] args)
        {
            MyList list = new MyList();
            for (int i = 0; i < 5; i++)
            {
                list[i] = i;
            }
            for (int i = 0; i < list.Length; i++)
                Console.WriteLine(list[i]);

        }
    }
}
```

### foreach가 가능한 객체 만들기
{: .H3-font}

foreach가 가능하기 위해선 인터페이스 IEnumerable을 상속해야함.

그러므로 IEnumerable이 가지고 있는 메소드를 구현해야 하는데 그것이 바

IEnumerator GetEnumerator() 메소드를 구현해야함

이 메소드는 IEnumerator 형식의 객체를 반환하기에 IEnumerator의 객체도 구현해야함

하지만 IEnumerator 를 상속하는 클래스를 따로 만들어 객체를 만들 필요없이

 yield문을 이용할 수 있음.

yield return문은 현재 메소드의 실행을 `일시 정지` 해놓고 호출자에게 결과를 반환함

메소드가 다시 호출되면, 일시 정지된 실행을 복구하여 `yield return` 또는 `yeild break` 문을 

만날 때 까지 나머지 작업을 실행하게 됨.

다음은 yield문을 사용하여 foreach가 가능한 객체를 만드는 예시임

```csharp
using System.Collections;
class MyEnumerator
{
    int[] numbers = { 1, 2, 3, 4 };
    public IEnumerator GetEnumerator()
    {
        yield return numbers[0];
        yield return numbers[1];
        yield return numbers[2];
        yield break;
    }
}
internal class Program
{

    static void Main(string[] args)
    {
        var obj = new MyEnumerator();
        foreach (int i in obj)
        {
            Console.WriteLine(i);
        }
    }
}

// 정리
// foreach 문을 돌릴 수 있는 객체를 만들려면
// IEnumerator를 반환하는 GetEnumerator()를 구현하면 됨
```

IEnumerator를 직접 상속받아 구현할 수도 있다.

그 경우 IEnumerator의 메소드와 프로퍼티를 구현해야만한다

다음은 그 예시이다

```csharp
using System.Collections;

class MyList : IEnumerable, IEnumerator
{
    private int[] array;
    int position = -1;

    public MyList()
    {
        array = new int[3];
    }

    public int this[int index]
    {
        get
        {
            return array[index];
        }

        set
        {
            if(index >= array.Length)
            {
                Array.Resize<int>(ref array, index + 1);
                Console.WriteLine($"Array Resized : {array.Length}");
            }

            array[index] = value;
        }
    }

    // IEnumerator 멤버
		// 현재 위치를 반환하는 프로퍼티
    public object Current
    {
        get
        {
            return array[position];
        }
    }

    // IEnumrator 멤버
		// 이동을 구현. 컬렉션의 끝을 지난 경우 false, 이동이 성공한 경우 true반환
    public bool MoveNext()
    {
        if(position == array.Length - 1)
        {
            Reset();
            return false;
        }

        position++;
        return (position < array.Length);
    }

    // IEnumerator 멤버
		// 첫 번째 위치의 '앞'으로 이동
    public void Reset()
    {
        position = -1;
    }

    // IEnumeraable 멤버
    public IEnumerator GetEnumerator()
    {
        return this; // 자신이 enumerator 이므로 yield문을 사용할 필요없이 자신을 반환
    }
}
internal class Program
{
    static void Main(string[] args)
    {
        MyList list = new MyList();
        for (int i = 0; i < 5; i++)
        {
            list[i] = i;
        }

        foreach (int e in list)
            Console.WriteLine(e);
    }
}
```

