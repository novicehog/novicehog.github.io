---
layout: single
title:  "[C#] 리플렉션"
categories: 
    - C Sharp
tag: [c#]
published: true

author_profile: true # 옆에뜨는 프로파일

date: 2024-04-13
last_modified_at: 2024-04-13
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
이 글은 한빛미디어의 **이것이 C#이다**를 보고 헷갈리는 부분 위주로 정리한 글입니다. 틀리거나 부족한 부분이 있을 수 있습니다.
{: .notice--warning}

# 리플렉션
리플렉션은 객체를 `X-Ray 사진`처럼 객체의 `형식(type) 정보를 들여다보는 기능`이다.
프로그램이 돌아가는 시점(.exe실행)에서 객체의 형식 이름부터, 프로퍼티 목록, 메소드 목록, 필드, 이벤트 목록까지도 열어볼 수 있다. 

열어보는걸로 그치지 않고 형식의 이름만 있다면 동적으로 인스턴스를 만들거나, 그 인스턴스의 메소드를 호출하는 것도, 심지어는 새로운 데이터 형식을 동적으로 만드는 것도 가능하다. 즉, `프로그래머에게 매우 다체로운 표현력을 선사한다.`


## Object.GetType() 메소드와 Type클래스
.NET에서는 `모든 형식을 들여다볼 수 있도록 장치를 설계`하였다.<br>
바로 모든 데이터 형식의 조상인 `Object형식에 GetType() 메소드를 만들어 놓았다.`<br>
여기서 사용하는 GetType()메소드는 `객체의 형식 정보(Type)를 반환`하는 기능을 한다.<br>

모든 데이터 형식이 GetType()메소드를 가지고 있기 때문에 `어떤 객체에 대해서든` 
이 메소드를 호출해서 `그 객체의 형식 정보를 얻어낼 수 있다.`

GetType()은 Type형식의 결과를 반환하는데, Type형식은 
.NET에서 사용하는 데이터 형식의 모든 정보를 담고있다.

GetType()과 Type형식을 사용하는 방법은 다음과 같다.

```cs
static void Main(string[] args)
{
    int a = 0;

    Type type = a.GetType();
    FieldInfo[] fields = type.GetFields();

    foreach (FieldInfo field in fields)
    {
        Console.WriteLine("Type:{0}, Name:{1}", field.FieldType.Name, field.Name);
    }
}
//Type:Int32, Name:MaxValue
//Type:Int32, Name:MinValue
```
<br>

자주 사용하는 메소드는 다음과 같다.

| 메소드 | 반환 형식 | 설명 |
| --- | --- | --- |
| GetConstructors() | ConstructorInfo[] | 해당 형식의 모든 생성자 목록의 반환함 |
| GetEvents() | EventInfo[] | 해당 형식의 이벤트 목록을 반환함 |
| GetFields() | FieldInfo[] | 해당 형식의 필드 목록을 반환함 |
| GetGenericArguments() | Type[] | 해당 형식의 형식 매개변수 목록을 반환함 |
| GetInterfaces() | Type[] | 해당 형식의 인터페이스 목록을 반환함 |
| GetMembers() | MemberInfo[] | 해당 형식의 멤버 목록을 반환함 |
| GetMethods() | MethodInfo[] | 헤당 형식의 메소드 목록을 반환함 |
| GetNestedTypes() | Type[] | 해당 형식의 내장 형식 목록을 반환함 |
| GetProperties() | PropertyInfo[] | 해당 형식의 프로퍼티 목록을 반환함 |


***
앞에 표에 있는 `GetFields()`나 `GetMethods()` 같은 메소드는 검색 옵션을 지정할 수 있다.
예를 들어 `public` 항목만, `비(Non) public` 형식만 조회하거나, `둘다 함께 조회`할 수 있다.
또한 `static` 항목만 조회하거나 `Instance` 항목만 조회할 수 있거나 `둘다 조회`할 수 있다.

이러한 검색 옵션은 System.Reflection.BindingFlags 열겨형을 이용해서 구성된다.

검색 옵션 예시 코드
```cs
internal class Program
{
    public class A
    {
        public int public_Instance_Field = 0;
        private int private_Instance_Field = 0;
        public static int public_Static_Field = 0;
        private static int private_Static_Field = 0;

        public void public_Instnace_Method(){}
        private void private_Instance_Method() { }
        public static void public_Static_Method() {}
        private static void private_Static_Method() { }
    }
    static void Main(string[] args)
    {
        A a = new A();

        Type type = a.GetType();

        foreach (FieldInfo field in type.GetFields(BindingFlags.Public | BindingFlags.Instance))
        {
            Console.WriteLine("public Instance Field : " + field.Name);
        }

        foreach (FieldInfo field in type.GetFields(BindingFlags.NonPublic | BindingFlags.Instance))
        {
            Console.WriteLine("private Instance Field : " + field.Name);
        }

        foreach (FieldInfo field in type.GetFields(BindingFlags.Public | BindingFlags.Static))
        {
            Console.WriteLine("public static Field : " + field.Name);
        }

        foreach (FieldInfo field in type.GetFields(BindingFlags.NonPublic | BindingFlags.Static))
        {
            Console.WriteLine("private static Field : " + field.Name);
        }

        // 메소드 -----------------------------------------------------------
        foreach (MethodInfo method in type.GetMethods(BindingFlags.Public | BindingFlags.Instance))
        {
            Console.WriteLine("public instnace Method : " + method.Name);
        }
        foreach (MethodInfo method in type.GetMethods(BindingFlags.NonPublic | BindingFlags.Instance))
        {
            Console.WriteLine("private instnace Method : " + method.Name);
        }
        foreach (MethodInfo method in type.GetMethods(BindingFlags.Public | BindingFlags.Static))
        {
            Console.WriteLine("public static Method : " + method.Name);
        }
        foreach (MethodInfo method in type.GetMethods(BindingFlags.NonPublic | BindingFlags.Static))
        {
            Console.WriteLine("private static Method : " + method.Name);
        }
    }
}
/* 출력결과
public Instance Field : public_Instance_Field
private Instance Field : private_Instance_Field

public static Field : public_Static_Field
private static Field : private_Static_Field

public instnace Method : public_Instnace_Method
public instnace Method : GetType
public instnace Method : ToString
public instnace Method : Equals
public instnace Method : GetHashCode
private instnace Method : private_Instance_Method
private instnace Method : MemberwiseClone
private instnace Method : Finalize

public static Method : public_Static_Method
private static Method : private_Static_Method
*/
```

추가로 검색 옵션에 아무런 매개변수도 주지 않으면 public인 모두(instance, static)를 찾는다.


## 리플렉션을 이용해서 객체 생성과 활용
### 객체 생성
이번에는 리플렉션을 이용해 특정 형식의 `인스턴스를 만들고` `데이터를 할당`하며 `메소드를 호출`하는 방법에 대해 알아봄.

리플렉션을 이용해 동적으로 인스턴스를 만들기 위해서는 `System.Activator` 클래스를 사용해야함.
인스턴스를 만들려는 형식의 Type 객체를 매개변수에 넘기면, Activator.CreateInstance() 메소드는 입력받은 형식의 인스턴스를 생성하여 반환한다.

```cs
object a = Activator.CreateInstance(typeof(int));
```
<br>

또한 일반화(Generic)을 지원하는 버전의 CreateInstance()메소드도 있다.  예를들어 List<int>의 인스턴스를
만들고 싶으면 다음과 같다.

```cs
List<int> list = Activator.CreateInstance<List<int>>();
```

### 프로퍼티 동적 할당
인스턴스만 생성만 동적으로 할 수 있는것이 아닌 객체의 `프로퍼티에 값을 할당`하는 것도 동적으로 할 수 있다.<br>
GetProperty()를 통해서 얻은 `PropertyInfo`클래스가 가지고 있는 `SetValue()`와 `GetValue()` 메소드를 통해 가능하다.

```cs
public class Profile
{
    public string Name { get; set; }
    public string Phone { get; set; }
}
static void Main(string[] args)
{
    Type type = typeof(Profile);
    // 인스턴스 생성
    Object profile = Activator.CreateInstance(type);

    // 프로퍼티 인포 가져오기
    PropertyInfo name = type.GetProperty("Name");
    PropertyInfo phone = type.GetProperty("Phone");

    // 값 할당
    name.SetValue(profile, "박찬호", null);
    phone.SetValue(profile, "123-1234", null);

    // 할당된 값 가져오기
    Console.WriteLine($"{name.GetValue(profile, null)} {phone.GetValue(profile, null)}");
}
// 출력 결과
// 박찬호 123-1234
```
<br>

PropertyInfo는 프로퍼티 뿐 아니라 [인덱서](https://novicehog.github.io/c%20sharp/ch10-array-collection/#%EC%9D%B8%EB%8D%B1%EC%84%9C-indexer)의 정보도 담을 수 있는데,
SetValue()나 GetValue() 메소드의 `마지막 인수`는 `인덱서의 인덱스`를 위해 사용된다.<br>
여기서는 인덱서가 필요 없으므로 null을 할당한다.

### 메소드 호출
메소드는 MethodInfo 클래스를 이용해서 호출할 수 있다.<br>
MethodInfo 클래스에는 `Invoke()`라는 메소드가 있는데 이 메소드를 이용하면 동적으로 메소드를 호출할 수 있다.

```cs
public class Profile
{
    public string Name { get; set; }
    public string Phone { get; set; }

    public void Print()
    {
        Console.WriteLine($"{Name}, {Phone}");
    }
}
static void Main(string[] args)
{
    Type type = typeof(Profile);
    // 인스턴스 생성
    Profile profile = Activator.CreateInstance(type) as Profile; 
    if(profile == null) { return; }
    profile.Name = "박찬호";
    profile.Phone = "123-456";

    // 메소드 찾기
    MethodInfo method = type.GetMethod("Print");

    // 메소드 호출
    method.Invoke(profile, null);
}
```