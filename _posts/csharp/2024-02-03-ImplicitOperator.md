---
layout: single
title:  "[C#] Implicit Operator 암시적 형변환"
categories: 
    - C Sharp
tag: [c#]
published: true

author_profile: true # 옆에뜨는 프로파일

date: 2024-02-02
last_modified_at: 2024-02-02
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

# implicit operator

## 예시
간단한 구조체와 float형이 자동으로 변환할 수 있게 해주는 코드이다.

```cs
public struct Millimeter
{
    public float Value;

    Millimeter(float num)
    {
        this.Value = num;
    }


    // Millimeter 구조체형에서 float형으로 암시적 형변환을 구현한다.
    public static implicit operator float(Millimeter num)
    {
        return num.Value * 1000;
    }

    // float형에서 Millimeter 구조체로 암시적 형변환을 구현한다.
    public static implicit operator Millimeter(float num)
    {
        return new Millimeter(num / 1000.0f);
    }
}

class Program
{
    static void Main(string[] args)
    {
        // 암시적 형변환을 구현하여 float형을 바로 대입할 수 있음
        Millimeter myMil = 10;

        // 0.01 출력
        Console.WriteLine(myMil.Value);

    }
}
```