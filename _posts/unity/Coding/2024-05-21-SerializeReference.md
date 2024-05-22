---
layout: single
title:  "Finite State Machine(FSM) 유한 상태 머신"
categories: 
    - Unity
tag: [Unity]
published: true

author_profile: true # 옆에뜨는 프로파일

search: true #검색해도 안뜨게함

date: 2024-05-22
last_modified_at: 2024-05-22

---

이 글은 **`인프런`** [모듈식으로 개발하는 스킬 시스템](https://www.inflearn.com/course/%EC%9C%A0%EB%8B%88%ED%8B%B0-%EB%AA%A8%EB%93%88%EC%8B%9D-%EC%8A%A4%ED%82%AC-%EC%8B%9C%EC%8A%A4%ED%85%9C)를 보고 공부한 내용을 정리한 글입니다.
{: .notice--warning}

## Finite State Machine이란?
FSM, 유한 상태 기계란 `객체의 상태 제어`를 위해 유한한 개수의 상태(State)를 가지며, 그것들을 관리하는 추상 기계이다. <br>
흔히 게임에서는 간단한 AI를 구현하는데 사용된다.<br>
유니티에서 기본적으로 제공하는 Animation Controller는 앞으로 구현할 코드와 매우 유사하다. 그것과 같다고 생각하고 이해하면 도움이 된다.

FSM의 구현은 크게 세 가지 요소를 중점적으로 보면 된다.
- State : 사물이나 생물이 가질 수 있는 `어떤 추상적 상태`를 의미. ex(사람의 경우 밥먹는 중, 걷는 중, 신호등의 경우 노란불 빨간불 등)
- Transition : 하나의 State에서 다른 State로 전이하기 위한 `연결선`이자 `조건`. 즉, State와 State간의 연결을 의미한다. 
- StateMachine : State와 Transition을 `종합적으로 관리`하는 추상 기계

이제 실제 코드와 함께 보면서 유니티에서 어떤 식으로 구현되는지를 보자. <br>
이번에 구현되는 코드는 `Multi Layer`를 지원하는 `Multi LayeredStateMachine`이다 (유니티의 Animation Controller와 같음)
코드는 State, Transition, StateMachine 세 가지로 이루어진다.

### State
먼저 State의 코드는 다음과 같다.

```cs
// 제네릭으로 StateMachine의 소유자를 받음
public abstract class State<EntityType>
{
    public StateMachine<EntityType> Owner { get; private set; }
    public EntityType Entity { get; private set; }

    // StateMachine에 등록될 레이어
    public int Layer { get; private set; }

    public void Setup(StateMachine<EntityType> owner, EntityType entity, int layer)
    {
        Owner = owner;
        Entity = entity;
        Layer = layer;

        Setup();
    }

    // Awake 역할을 해줄 Setup함수
    protected virtual void Setup() { }

    // State가 시작될 때 실행될 함수
    public virtual void Enter() { }

    // State가 실행중일 때 매 프레임마다 실행될 함수
    public virtual void Update() { }

    // State가 끝날 때 실행할 함수
    public virtual void Exit() { }

    /// StateMachine을 통해 외부에서 메세지가 넘어왔을 때 처리하는 함수
    /// message : State에게 특정 작업을 하라고 명령하기 위해 개발자가 정한 신호 </param>
    /// data : 메세지와 함께 넘어오는 데이터, 데이터는 어떠한 형태도(투플이든, 리스트든) 받을 수 있도록 object형 </param>
    /// 반환값 : 메세지를 올바르게 수신하였다면 True를 반환하도록 오버라이드</returns>
    public virtual bool OnReceiveMessage(int message, object data) => false;
}
```
모든 State의 기본이 되는 State는 추상 클래스로 구현되며 `새로운 State를 정의`할 떄는 `이 클래스를 상속하여 정의`한다.<br>

State클래스는 제네릭으로 `StateMachine의 소유주의 타입` 정의한다.<br>
이는 State, Transition, StateMachine도 마찬가지이며 세 클래스의 타입을 일치시키기 위해서이다.

```cs
public abstract class State<EntityType> {...}
```
<br>

그 다음 변수로 이 State를 관리하는 StateMachine과 소유주, State가 속한 Layer를 선언한다.

```cs
public StateMachine<EntityType> Owner { get; private set; }
public EntityType Entity { get; private set; }

// StateMachine에 등록될 레이어
public int Layer { get; private set; }
```
<br>


Setup함수는 이 State가 등록될 때 호출될 함수이며 자식클래스에서도 Setup이 되는 타이밍에 따로 작업을 할 수 있도록 오버로딩한다.

```cs
public void Setup(StateMachine<EntityType> owner, EntityType entity, int layer)
{
    Owner = owner;
    Entity = entity;
    Layer = layer;

    Setup();
}

// Awake 역할을 해줄 Setup함수
protected virtual void Setup() { }
```
<br>

그 다음은 State에 진입, 유지(매 프레임), 끝날 때 호출될 함수들을 정의한다.

```cs
// State가 시작될 때 실행될 함수
public virtual void Enter() { }

// State가 실행중일 때 매 프레임마다 실행될 함수
public virtual void Update() { }

// State가 끝날 때 실행할 함수
public virtual void Exit() { }
```
<br>

마지막으로 OnReceiveMessage함수를 정의해주는데 이는 StatmeMachine에서 SendMessage라는 함수가 호출될 때 사용되며,
현재 State의 OnReceiveMessage함수를 실행시켜줄 것이다. 자세한건 주석으로 작성하였다.

```cs
/// StateMachine을 통해 외부에서 메세지가 넘어왔을 때 처리하는 함수
/// message : State에게 특정 작업을 하라고 명령하기 위해 개발자가 정한 신호 </param>
/// data : 메세지와 함께 넘어오는 데이터, 데이터는 어떠한 형태도(투플이든, 리스트든) 받을 수 있도록 object형 </param>
/// 반환값 : 메세지를 올바르게 수신하였다면 True를 반환하도록 오버라이드</returns>
public virtual bool OnReceiveMessage(int message, object data) => false;
```
<br>

이것으로 모든 State의 기반이 될 State 클래스는 완성되었다.


### StateTransition
