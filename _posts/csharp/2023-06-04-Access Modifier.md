---
layout: single
title:  "C# 접근 한정자"
categories: 
    - C Sharp
tag: [c#]
published: true


author_profile: true # 옆에뜨는 프로파일

date: 2023-06-04
last_modified_at: 2023-06-04
---
이 글은 한빛미디어의 **이것이 C#이다**를 보고 헷갈리는 부분 위주로 정리한 글입니다.
{: .notice--warning}

# 접근 한정자


| 접근 한정자 | 설명 |
| --- | --- |
| public | 클래스 내부/외부 모든 곳에서 접근할 수 있음 |
| protected | 클래스의 외부에서 접근할 수 없지만, 파생 클래스에서는 접근할 수 있음. |
| private | 클래스의 내부에서만 접근할 수 있음.  |
| internal | 같은 어셈블리에 있는 코드에서만 public으로 접근할 수 있음. 다른 어셈블리에 있는 코드에서는 private과 같은 수준의 접근성을 가짐 |
| protected internal | 같은 어셈블리에 있는 코드에서만 protected으로 접근할 수 있음. 다른 어셈블리에 있는 코드에서는 private과 같은 수준의 접근성을 가짐 |
| private protected | 같은 어셈블리에 있는 클래스에서 상속받은 클래스 내부에서만 접근할 수 있음 |


아무런 한정자도 지정해주지 않으면 private로 기본 설정 됨.