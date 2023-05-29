---
layout: single
title:  "C# Chapter 5 코드의 흐름 제어하기"
categories: 
    - C Sharp
tag: [c#]
published: true


author_profile: true # 옆에뜨는 프로파일

date: 2023-05-23
last_modified_at: 2023-05-23
---
이 글은 한빛미디어의 **이것이 C#이다**를 보고 공부한 내용을 정리한 글입니다.
{: .notice--warning}

헷갈리는 부분 위주로 정리했습니다.

## switch 문
{: .H2-font}

### switch문의 패턴 매칭
obj와 같이 여러 타입을 담을 수 있는 변수는 
다음과 같이 `switch`문을 통해 형식 매치를 할 수 있다.
```c#
object obj = 5;
switch (obj)
{
    case int: //실행하면 obj는 int를 박싱해서 담고있기에 이쪽 case문을 지나간다.
        break;
    case float:
        break;
    case string:
        break;
}
```

### 케이스 가드
`switch`문의 경우 `case` 뒤에 `when 조건문`을 입력받아 더 자세한 조건을 설정할 수 있다.

```c#
object obj = 5;
switch (obj)
{
    case int i when i > 5: // switch문으로 들어온 값이 5 이상인 int의 경우에만 해당.
        break;
    case int : 
        break;        
    case float:
        break;
    case string:
        break;
}
```

### switch 식
#### 식과 문의 차이
- 식은 계산을 해서 결과를 내놓음 (단일 결과값을 만들어 낼 수 있는 연산자와 연산자의 조합)
* `d = a + 123;` a + 123 과 d = a + 123은 식 
- 문은 주어진 일을 할 뿐임 

다음은 switch문의 예제이다
```c#
        int input = Convert.ToInt32(Console.ReadLine());

            // 1의 자리를 버림
        int score = (int)(Math.Truncate(input / 10.0) * 10);
        string? grade;

        switch (score)
        {
            case 90:
                grade = "A";
                break;
            case 80:
                grade = "B";
                break;
            case 70:
                grade = "C";
                break;
            case 60:
                grade = "D";
                break;
            default:
                grade = "F";
            }
```
값(성적)을 입력받아 해당 점수에 맞게 grade 변수에 등급을 넣어주는 코드이다.
이 경우 switch문이 값을 판별하고 grade에 값을 따로 넣어주게 된다.

이 코드를 switch식으로 바꾸면
```c#
        int input = Convert.ToInt32(Console.ReadLine());

        // 1의 자리를 버림
        int score = (int)(Math.Truncate(input / 10.0) * 10);
        string? grade = score switch
        {
            90 => "A",
            80 => "B",
            70 => "C",
            60 => "D",
            _ => "F"
        }; 

```
위와같이 바꿀 수 있다. 바뀐 부분은 다음과 같다.
- switch문이 =를 통해 대입되는 switch식으로 바뀌었다.
- case가 `=>`로 바뀌었고 `,`를 통해 다음 `=>` 과 구분하였다.
- default의 경우 `_`로 대체되었다.
- switch문이 switch식으로 바뀌며 하나의 결과 값을 무조건 내놓게 바뀌었다.
- 즉, 함수에서 switch식을 통해 return한다면 함수의 반환 형태는 void가 될 수 없다.

switch식의 경우도 케이스가드 when을 통해 자세한 조건을 설정할 수 있다.

```c#
        bool repeated = true;
        int input = Convert.ToInt32(Console.ReadLine());

        // 1의 자리를 버림
        int score = (int)(Math.Truncate(input / 10.0) * 10);
        string? grade = score switch
        {
            90 when repeated == true  => "B+",
            90 => "A" ,
            80 => "B",
            70 => "C",
            60 => "D",
            _ => "F"
        }; 

```
위 코드와 같이 재수강 여부를 알려주는 repeated변수를 통해
90점 이상이라도 재수강의 경우는 B+만 되도록 할 수 있다.


## foreach 문
{: .H2-font}

- foreach문은 배열(또는 컬렉션)을 순회하며 각 데이터 요소에 차례대로 접근하도록 해준다.
- 배열(또는 컬렉션) 끝에 접근하게 되면 자동으로 종료된다.
- foreach문의 기본 형태는 다음과 같다
```c#
foreach(데이터_형식 변수명 in 배열_또는_컬렉션)
    코드_또는_코드블록
```

- in키워드와 함께 사용하며 foreach문이 한 번 반복을 수행할 때마다
배열(또는 컬렉션)의 요소를 차례대로 순회하면서 in 키워드 앞에 있는 변수에 담아준다.
- 변수의 데이터 형식은 주로 var을 사용하여 모든 변수에 대처할 수 있도록 한다.

다음은 간단한 예제이다.

```c#
        int[] arr = new int[] { 0, 1, 2, 3 };

        foreach(var v in arr)
        {
            Console.Write(v);
        }
        ///0123 출력
```

## 패턴 매칭
{: .H2-font}
- 패턴 매칭은 장황하고 거추장스러운 분기문을 간결하고 읽기 쉬운 코드로 대체할 수 있다.
- switch문,switch식과 연관이 깊지만 is 연산자를 이용하여 다른 문장이나 식에서도 사용할 수 있다.

### 선언 패턴
{: .H3-font}

- 주어진 식이 `특정 형식(int나 string 등)`과 일치하는지 본다.
- 일치한다면 식을 주어진 형식으로 변환한다.

다음의 코드를 보면
```c#
        object obj = 50;

        if (obj is int i)
        {
            Console.WriteLine($"{i}은 int다");
            Console.WriteLine(obj is int);
        }
        if (obj is float f)
        {
            Console.WriteLine($"{f}은 float다");
            Console.WriteLine(obj is int);
        }
        // 출력 값
        // 50은 int다
        // True
```
- if문의 조건식 안에서 먼저 `obj` 가 int형인지 판별한다. true 또는 false 반환
- 일치한다면 int형인 i에 obj의 값을 담은 지역변수를 **`선언`**한다. 
- obj가 int형인 50을 박싱해서 담고있으니 밑에 있는 if문에선 float가 아니기에 if이 무시된다.

### 형식 패턴
{: .H3-font}

- 선언 패턴과 거의 같은 방식으로 동작하지만, 변수 생성없이 일치 여부만 판별한다.

```c#
        object obj = 50f;

        if (obj is int)
        {
            Console.WriteLine("int다");
            Console.WriteLine(obj is int);
        }
        if (obj is float f)
        {
            Console.WriteLine("float다");
            Console.WriteLine(obj is int);
        }
        // 출력 값
        // float다
        // True
```

### 상수 패턴
{: .H3-font}

- 식이 특정 상수와 일치하는 검사
- 정수 리터럴과 문자열 리터럴 뿐 아니라 null, enum등 모든 상수와 매칭 가능

```c#
        object? obj = null;

        if (obj is null) // obj == null 과 같음
        {
            Console.WriteLine("null임");
        }
```

### 프로퍼티 패턴
{: .H3-font}

- 식의 속성이나 필드가 패턴과 일치하는지를 검사

```c#
    class B
    {
        public string name { get; set; }
        public int age { get; set; }    
    }
    static void Main(string[] args)
    {
        B b = new B { name = "Joon", age = 15 };
        if(b is B{ name : "Joon" , age : 15 })
        {
            Console.WriteLine($"B의 이름은 Joon이고 나이는 15입니다");
        }
        if (b is B { name: "yaho", age: 20 })
        {
            Console.WriteLine($"B의 이름은 yaho이고 나이는 20입니다");
        }
    }
```

- 위 코드에서는 class B가 프로퍼티 name과 age를 가지고있다.
- B의 객체인 b를 선언하고 프로퍼티 값을 이름은 Joon 나이는 15로 하였다.
- 첫 번째 if문의 조건식에서 프로퍼티 패턴이 `name = Joon, age = 15`로 일치하므로 if문이 실행된다.
- 두 번째 if문의 경우 조건식의 프로퍼티 패턴이 일치하지 않으므로 실행되지 않는다.

### 관계 패턴
{: .H3-font}

- >, >=, ==, !=, <, <= 와 같은 관계 연산자를 이용하여 입력받은 식을 상수와 비교

```c#
        int score = 76;
        string grade = score switch
        {
            < 60 => "F",  //60 미만의 경우 F
            >= 60 and < 70 => "D", // 60이상 70미만 D
            >= 70 and < 80 => "C", // 70이상 80미만 C
            >= 80 and < 90 => "B", // 80이상 90미만 B
            _ => "A" // 그 외에는 A
        };

        Console.WriteLine(grade); // C 를 출력
```

switch문이 아니라도 사용가능

```c#
        int score = 59;
        if(score is > 60) // score > 60
        {
            Console.WriteLine("F는 아니네");
        }
        else
        {
            Console.WriteLine("F네..."); /
        }
        // 출력 결과
        // F네...
```

### 논리 패턴
{: .H3-font}

- 패턴과 패턴을 패턴 논리 연산자`and(결합), or, not`을 조합해서 하나의 논리 패턴으로 만들 수 있음

```c#
    class B
    {
        public string name { get; set; }
        public int age { get; set; }
    }
    static void Main(string[] args)
    {

        Console.WriteLine(RamdaMethod(new B() { name = "Hi" , age = 15})); // 출력 결과 : "이름 Hi이거나 나이 30"
    }
    static string RamdaMethod(B b) => b switch
    {
        B { name: "Hi" } or B { age: 30 } => "이름 Hi이거나 나이 30",
        _ => "아무것도 일치하지 않음"
    };
```

- 위 코드에선 메소드 RamdaMethod에서 or를 통한 논리패턴이 사용되었다.
- 매개변수로 주어진 값 중 속성 name이 일치하기에 `이름 Hi이거나 나이30`이 반환되었다.

### 괄호 패턴
{: .H3-font}

- `소괄호()`로 패턴을 감쌈.
- 보통 논리 패턴으로 여러 패턴을 조합한 뒤 이를 새로운 패턴으로 만드는 경우 사용.

```c#
    class B
    {
        public int age { get; set; }
    }
    B b = new B() {age = 15};

    if(b.age is (int and > 10))
    {
        Console.WriteLine("나이는 int형이면서 10살");
    }
```

- 위 코드의 `(int and > 10)` 처럼 사용한다.

### 위치 패턴
{: .H3-font}

- 식의 결과를 분해(Deconstruct)하고, 분해된 값들이 내장된 복수의 패턴과 일치하는지를 검사.
- 내장되는 패턴에는 형식 패턴, 상수 패턴 등 모든 패턴 사용 가능.

[**Tuple 개념은 여기서 볼 수 있음.**](https://www.youtube.com/watch?v=QofCu147658)

```c#
var r = ("age", 30); // tuple

if ( r is ("age" , >25)) // tuple의 속성 값을 분해하여 각각 비교 
{
    Console.WriteLine("나이가 25보다 큽니다");
}
```
