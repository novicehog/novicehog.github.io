---
layout: single
title:  "C# Chapter 8 인터페이스와 추상클래스"
categories: 
    - C Sharp
tag: [c#]
published: true

author_profile: true # 옆에뜨는 프로파일

date: 2023-06-09
last_modified_at: 2023-06-09
---
이 글은 한빛미디어의 **이것이 C#이다**를 보고 헷갈리는 부분 위주로 정리한 글입니다.
{: .notice--warning}

# 인터페이스
{: .H1-font}

## 인터페이스 사용
{: .H2-font}

인터페이스는 약속같은 것이다.

인터페이스에서 선언된 메소드들은  인터페이스를 상속한 클래스에서 반드시 구현되어야 한다.

구현을 할떄는 반드시 public으로 구현되어야한다.

```csharp
    interface IInterface
    {
        void Method(); // 이 메소드를 상속받는 클래스는 꼭 오버로드해야함
    }
    class InterfaceClass : IInterface
    {
        public void Method(){} // 이 부분을 작성하지않으면 에러 발생
    }
```

인터페이스는 실체가 없어 객체 생성이 불가능하지만

참조변수를 선언하는 것은 가능하다.

```csharp
	static void Main(string[] args)
        {
            IInterface ITError = new IInterface; // 오류
            IInterface IT; // 이것은 가능
        }
```

인터페이스도 엄연한 상속관계를 가지므로 인터페이스의 자식들을 넣어주는 업캐스팅이 가능하다.

```csharp
        static void Main(string[] args)
        {
            IInterface IT = new InterfaceClass();
        }
```

이 경우 인터페이스의 참조변수를 통해 실체없는 메소드를 실행하면

자식 클래스의 구현된 메소드가 실행된다.

```csharp
    interface IInterface
    {
        void Method();
    }
    class InterfaceClass : IInterface
    {
        public void Method()
        {
            Console.WriteLine("구현된 메소드입니다");
        }
    }

    internal class Program
    {
        static void Main(string[] args)
        {
            IInterface IT = new InterfaceClass();
						IT.Method(); // 출력 결과 구현된 메소드입니다
        }
    }
```

이를 응용하여 인터페이스 참조변수를 함수의 매개변수로 받아 

넘겨주는 인자값에 따라 다른 함수가 실행되게 할 수 있다.

```csharp
    interface IFoodFighter // 인터페이스
    {
        void SayFavoriteFood();
    }
    class Minho : IFoodFighter
    {
        public void SayFavoriteFood()
        {
            Console.WriteLine("저는 라면을 좋아합니다.");
        }
    }
    class Sungho : IFoodFighter
    {
        public void SayFavoriteFood()
        {
            Console.WriteLine("저는 피자를 좋아합니다.");
        }
    }
    class Interviewer
    {
        public static void InterView(IFoodFighter human) // 인터페이스 참조변수 사용
        {
            human.SayFavoriteFood();
        }
    }

    internal class Program
    {
        static void Main(string[] args)
        {
            Minho minho = new Minho();
            Sungho sungho = new Sungho();
            Interviewer.InterView(minho); // 저는 라면을 좋아합니다.
            Interviewer.InterView(sungho); // 저는 피자를 좋아합니다.
        }
    }
```

## 인터페이스를 상속하는 인터페이스
{: .H2-font}

새로운 기능을 추가한 인터페이스를 만들고싶을때 사용.

보통은 기존 인터페이스를 수정하겠지만 다음과 같은 경우에는

인터페이스를 상속한 인터페이스를 사용하는 수 밖에 없음.

- **상속하려는 인터페이스가 소스 코드가 아닌 어셈블리로만 제공되는 경우: 
즉 .NET SDK에서 제공하는 인터페이스들이 그 예다.**
- **상속하려는 인터페이스의 소스코드를 가지고 있어도 이미 인터페이스를 상속하는 클래스들이 존재하는 경우:** 이 경우에는 기존에 인터페이스를 상속한 클래스들을 전부 다 같이 바꿔줘야한다.

```csharp
    interface A
    {
        void MethodA();
    }
    interface B : A
    {
        void MethodB();
    }
    class C : B // A 와 B의 메소드를 둘다 구현해야함
    {
        public void MethodA()
        {
            // A 인터페이스 구현
        }
        public void MethodB()
        {
            // B 인터페이스 구현
        }
    }
```

## 여러 인터페이스, 한꺼번에 상속하기
{: .H2-font}

클래스의 경우 다중 상속이 불가능하지만

인터페이스는 다중 상속이 가능하다.

```csharp
    interface A
    {
        void MethodA();
    }
    interface B : A
    {
        void MethodB();
    }
    class C : A , B
    {
        public void MethodA()
        {
            // A 인터페이스 구현
        }
        public void MethodB()
        {
            // B 인터페이스 구현
        }
    }
```

## 인터페이스의 기본 구현 메소드
{: .H2-font}

인터페이스 메소드를 구현해놓으면 해당 메소드는 

반드시 구현해야 되는 메소드가 아니게 된다.

```csharp
    interface II
    {
        void Method() // 기본구현 메서드
        {
            Console.WriteLine("기본구현 메소드입니다.");
        }
    }
    class C : II
    {

    } // 구현하지 않았지만 아무 에러도 발생하지 않음.
```

이 경우 인터페이스를 상속 받더라도 클래스에서 구현하지 않으면

클래스 객체로 메소드에 접근할 수 없다.

```csharp
		static void Main(string[] args)
        {
            C c = new C();
            c.Method(); // 에러 발생
        }
```

즉, 인터페이스에 새로운 메소드를 추가할 때 기본적인 구현체를 갖도록 해서 기존에 있는 파생 클래스에서의 컴파일 에러를 막을 수 있고, 구현을 하지 않았을 경우 업캐스팅을 통해서만 접근이 가능하게하여 인터페이스에 추가된 메소드가 엉뚱하게 호출될 일도 없다.

# 추상 클래스
{: .H1-font}

클래스와 인터페이스의 중간 느낌

클래스처럼 멤버 변수를 물려줄 수 있고 클래스의 접근성 원칙도 지키지만

가상 메소드를 통해 인터페이스 처럼 메소드 구현을 강제시킬 수 있고

인터페이스 처럼 객체를 생성할 수 없다.

```csharp
    abstract class A
    {
        public int a = 5;
        public abstract void Method();
    }
    class C : A
    {
        public override void Method() // 무조건 오버라이딩 해야함
        {
            a = 1; // a 멤버 변수를 사용할 수 있음
        }
	  }
		internal class Program
    {
        static void Main(string[] args)
        {
            A a = new A(); // 에러 발생 추상메서드는 객체생성 불가

						C c = new C();
            c.Method();
            Console.WriteLine(c.a); // 1출력
        }
    }
```
