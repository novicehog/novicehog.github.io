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
last_modified_at: 2024-05-23

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
다음으로는 State들간의 연결을 담당할 State Transition클래스이다.

```cs
public class StateTransition<EntityType>
{
    // Transition Command가 없음을 나타냄 (명령이 없다, nullable대신 사용하는 방법)
    public const int kNullCommand = int.MinValue;

    // Transition을 위한 조건 함수, 인자는 현재 State, 결과값은 전이 가능 여부
    private Func<State<EntityType>, bool> transitionCondition;

    // 현재 State에서 다시 현재State로 전이가 가능한지
    public bool CanTransitionToSelf { get; private set; }
    // 현재 State
    public State<EntityType> FromState { get; private set; }
    // 전이할 State
    public State<EntityType> ToState { get; private set; }
    // 전이 명령어
    public int TransitionCommand { get; private set; }


    /// 전이 가능 여부, 컨디션을 검사하여 컨디션이 없거나, 있다면 컨디션의 조건 함수를 실행하여 전이 가능 여부를 파악
    public bool IsTransferable => transitionCondition == null || transitionCondition.Invoke(FromState);

    public StateTransition(State<EntityType> fromState, State<EntityType> toState,
        int transitionCommand,
        Func<State<EntityType>, bool> transitionCondition,
        bool canTransitionToSelf)
    {
        Debug.Assert(transitionCommand != kNullCommand || transitionCondition != null, 
            "커맨드와 컨디션이 둘 다 null이 될 수 없음");

        FromState = fromState;
        ToState = toState;
        TransitionCommand = transitionCommand;
        this.transitionCondition = transitionCondition;
        CanTransitionToSelf = canTransitionToSelf;
    }
}
```
<br>

Transition는 특정한 조건을 만족하면 `특정 State -> 특정 State`로 또는 `아무 State -> 특정 State`로의 상태 전이가 일어난다.

먼저 알아두어야 할 것은 전이 조건을 판별할 때 2가지를 확인한다는 것이다.<br>
`전이가 가능한 조건의 Condition`과 `특정한 타이밍을 뜻하는 Command`가 있다.

Condition과 Command가 모두 있는 Transition의 경우에는 특정한 타이밍(내가 원할 때 Command 신호를 보냄)에 전이 조건이 만족해야한다.(condition의 조건문이 True를 반환해야함)
Command가 없을 경우엔 매 프레임 Condition을 체크.


크게 어려운 내용은 없으므로 주석으로 대체하고 생성자만 보겠다.

```cs
public StateTransition(State<EntityType> fromState, State<EntityType> toState,
    int transitionCommand,
    Func<State<EntityType>, bool> transitionCondition,
    bool canTransitionToSelf)
{
    Debug.Assert(transitionCommand != kNullCommand || transitionCondition != null, 
        "커맨드와 컨디션이 둘 다 null이 될 수 없음");

    FromState = fromState;
    ToState = toState;
    TransitionCommand = transitionCommand;
    this.transitionCondition = transitionCondition;
    CanTransitionToSelf = canTransitionToSelf;
}
```

생성자에선 시작 State, 전이할 State, Command, Condition, 스스로에게 전이 가능 여부를 인자로 받아 대입한다. <br>
이때 커맨드와 컨디션 둘다 존재하지 않는 경우는 의미가 없는 트랜지션이므로 Assert문으로 잘못됐음을 알린다.


### StateMachine
가장 중요하고 긴 StateMachine이다.

코드 자체가 길기 때문에 중요한 부분만을 서술하려고한다.

<details>
<summary>StateMachine 스크립트</summary>
<div markdown="1">       

```cs
using System;
using System.Collections;
using System.Collections.Generic;
using System.Linq;
using System.Net.NetworkInformation;
using UnityEngine;

public abstract class StateMachine<EntityType>
{
    #region 정의, 변수, 프로퍼티들
    // 대리자 Event 정의
    public delegate void StateChangeHandler(
        StateMachine<EntityType> stateMachine,
        State<EntityType> newState,
        State<EntityType> prevState,
        int layer);

    // State별로 여러개의 Transition을 가지고 있을 수 있기 때문에 State별로 한 번에 묶어서 관리하기 위한
    // 내장 클래스
    private class StateData
    {
        // State가 실행되는 Layer
        public int Layer { get; private set; }
        // State의 등록 순서
        public int Priority { get; private set; }
        // Data가 가진 State
        public State<EntityType> State { get; private set; }
        // State에서 다른 State로 이어진 Transition들
        public List<StateTransition<EntityType>> Transitions { get; private set; } = new();

        public StateData(int layer, int priority, State<EntityType> state) =>
            (Layer, Priority, State) = (layer, priority, state);
    }

    // Layer별 가지고 있는 StateDatas(=Layer Dictionary), Dictionary의 key는 Value인 StateData가 가진 State의 Type
    // 즉, State의 Type을 통해 해당 State가 가진 StateData를 찾아올 수 있음
    // 여기서 말하는 Type이란 State를 상속하여 만든 State 자식 클래스들을 의미
    // 예를 들어 State를 상속하는 RunState가 있다고 하면 
    // stateDatasByLayer[0][RunState.GetType()]를 하므로 써 0번 레이어의 RunState의 부가정보를 담고있는 StateData를 가져옴
    private readonly Dictionary<int, Dictionary<Type, StateData>> stateDatasByLayer = new();
    // Layer별 Any Transitions(조건만 만족하면 언제든지 ToState로 전이되는 Transition)들을 가지고 있음
    private readonly Dictionary<int, List<StateTransition<EntityType>>> anyTransitionsByLayer = new();

    // Layer별로 현재 실행중인 State의 StateData
    private readonly Dictionary<int, StateData> currentStateDatasByLayer = new();

    // StateMachine에 존재하는 Layer들, 중복X 자동정렬O 를 위해 SortedSet이용
    private readonly SortedSet<int> layers = new();

    // StateMachine의 소유주
    public EntityType Owner { get; private set; }

    public event StateChangeHandler onStateChanged;
    #endregion

    public void Setup(EntityType owner)
    {
        Debug.Assert(owner != null, $"StateMachine<{typeof(EntityType).Name}>:: Setup - owner는 null이 될 수 없음");

        Owner = owner;

        // 자식 StateMachine클래스에서 오버라이드해서 사용할 함수들
        AddStates();
        MakeTransitions();
        SetupLayers();
    }

    /// <summary>
    /// 설정된 상태들을 기준으로 State를 실행시킬 Layer들을 만들어주고,
    /// Layer별로 가장 먼저 등록된 상태들을 실행시킴
    /// </summary>
    public void SetupLayers()
    { 
        foreach ((int layer, var stateDatasByType) in stateDatasByLayer)
        {
            // State를 실행시킬 Layer를 만들어줌
            currentStateDatasByLayer[layer] = null;

            // 우선 순위가 가장 높은 StateData를 찾아옴
            var firstStateData = stateDatasByType.Values.First(x => x.Priority == 0);
            // 찾아온 StateData의 State를 현재 Layer의Current State로 설정해줌
            ChangeState(firstStateData);
        }
    }

    /// <summary>
    /// 현재 실행중인 CurrentStateData를 변경하는 함수
    ///  Enter함수와 Exit함수는 이곳에서 관리
    /// </summary>
    /// <param name="newStateData">전이할 상태</param>
    private void ChangeState(StateData newStateData)
    {
        // Layer에 맞는 현재 실행중인 CurrentStateData를 가져옴
        var prevState = currentStateDatasByLayer[newStateData.Layer];

        // 처음 상태가 시작될 땐 실행중인 상대가 없으므로 ?.접근연산자를 사용
        prevState?.State.Exit();

        // 인자로 받은 상태로 전이해줌
        currentStateDatasByLayer[newStateData.Layer] = newStateData;
        newStateData.State.Enter();

        onStateChanged?.Invoke(this, newStateData.State, prevState.State, newStateData.Layer);
    }

    /// <summary>
    /// State와 Layer를 통해 전이하는 ChangeState 오버로딩함수
    /// </summary>
    private void ChangeState(State<EntityType> newState, int layer)
    {
        var newStateData = stateDatasByLayer[layer][newState.GetType()];
        ChangeState(newStateData);
    }

    /// <summary>
    /// 전이 시도
    /// </summary>
    /// <returns>전이 성공 여부 반환</returns>
    private bool TryTransition(IReadOnlyList<StateTransition<EntityType>> transitions, int layer)
    {
        foreach (var transition in transitions)
        {
            // 여기서 continue는 전이하지 않겠다를 의미
            // 첫 조건 커맨드가 존재하면 true, 두 번째 조건 컨디션이 없거나 True 반환
            // 즉, 커맨드가 있다면 여기서 말고 커맨드가 왔을 때 따로 처리하므로 여기서는 처리하지 않고 continue로 넘어가고,
            // 커맨드가 없다면 전이조건을 확인해서 전이할 수 없을 경우 continue로 넘어감
            if (transition.TransitionCommand != StateTransition<EntityType>.kNullCommand || !transition.IsTransferable)
                continue;

            // 스스로에게 전이 불가능할 때 스스로에게 전이하려고 하면 continue
            if (!transition.CanTransitionToSelf && currentStateDatasByLayer[layer].State == transition.ToState)
                continue;

            // 모든 조건을 만족한다면 ToState로 전이
            ChangeState(transition.ToState, layer);
            return true;
        }
        // 전이 실패
        return false;
    }

    /// <summary>
    /// 이 함수를 매 프레임 호출하여 이 곳에서 Condition을 통한 State 전이와 State의 Update함수를 실행
    /// </summary>
    public void Update()
    {
        foreach (var layer in layers)
        {
            // 현재 State의 StateData를 가져옴
            var currentStateData = currentStateDatasByLayer[layer];

            // 현재 Layer의 AnyTrasitions들을 가져옴
            bool hasAnyTransitions = anyTransitionsByLayer.TryGetValue(layer, out var anyTransitions);

            // AnyTransition으로 먼저 전이 시도를 해보고 안된다면 현재 StateData의 Transition을 통해 전이를 시도함
            // 즉, AnyTransition이 일반적인 Transition보다 우선됨
            // 전이에 성공했다면 다음 레이어에서 똑같은 작업 반복
            if ((hasAnyTransitions && TryTransition(anyTransitions, layer)) ||
                TryTransition(currentStateData.Transitions, layer))
                continue;

            // 전이에 실패한다면 현재 State의 Update함수를 실행
            currentStateData.State.Update();
        }
    }



    public void AddStates<T>(int layer = 0) where T : State<EntityType>
    {
        // Set이므로 이미 존재한다면 추가하지 않음
        layers.Add(layer);

        // Generic타입이므로 Activator.CreateInstance<T>()를 통해서 객체를 생성함
        var newState = Activator.CreateInstance<T>();
        newState.Setup(this, Owner, layer);

        // 아직 stateDatasByLayer에 추가되지 않은 Layer라면 Layer를 생성
        if(!stateDatasByLayer.ContainsKey(layer))
        {
            // Layer의 StateDate 목록인 Dictionary<Type, StateData> 생성
            stateDatasByLayer[layer] = new();
            // Layer의 AnyTransitions 목록인 List<StateTransition<EntityType>> 생성
            anyTransitionsByLayer[layer] = new();
        }

        Debug.Assert(!stateDatasByLayer[layer].ContainsKey(typeof(T)),
            $"StateMachine::AddState<{typeof(T).Name}> - 이미 상태가 존재합니다.");


        var stateDatasByType = stateDatasByLayer[layer];
        // Dictionary<Type, StateData>에 저장함
        stateDatasByType[typeof(T)] = new StateData(layer, stateDatasByType.Count, newState);
    }


    /// <summary>
    /// Transition을 생성하는 함수
    /// </summary>
    public void MakeTransitions<FromStateType, ToStateType>(int transitionCommand,
        Func<State<EntityType>, bool> transitionCondition, int layer = 0) 
        where FromStateType : State<EntityType>
        where ToStateType : State<EntityType>
    {
        var stateDatas = stateDatasByLayer[layer];
        // StateDatas에서 FromStateType의 State를 가진 StateData를 찾아옴
        var fromStateData = stateDatas[typeof(FromStateType)];
        // StateDatas에서 ToStateType의 State를 가진 StateData를 찾아옴
        var toStateData = stateDatas[typeof(ToStateType)];

        // 인자와 찾아온 Data를 가지고 Transition생성
        // AnyTransition이 아닌 일반 Transition은 canTransitionToSelf 인자가 무조건 True
        var newTransition = new StateTransition<EntityType>(fromStateData.State, toStateData.State,
            transitionCommand, transitionCondition, true);

        // 생성한 Transtion을 FromStateData의 Transition으로 추가
        fromStateData.Transitions.Add(newTransition);
    }

    #region MakeTranstions
    // MakeTranstion함수의 Enum Command 버전
    // Enum형으로 받은 Commnad를 Int로 변환하여 위의 함수 호출
    public void MakeTransitions<FromStateType, ToStateType>(Enum transitionCommand,
        Func<State<EntityType>, bool> transitionCondition, int layer = 0)
        where FromStateType : State<EntityType>
        where ToStateType : State<EntityType>
        => MakeTransitions<FromStateType, ToStateType>(Convert.ToInt32(transitionCommand), transitionCondition, layer);

    // Command 인자가 없는 버전
    // null 커맨드를 넣어 최상단 MakeTranstion 호출
    public void MakeTransitions<FromStateType, ToStateType>(
        Func<State<EntityType>, bool> transitionCondition, int layer = 0)
        where FromStateType : State<EntityType>
        where ToStateType : State<EntityType>
        => MakeTransitions<FromStateType, ToStateType>(StateTransition<EntityType>.kNullCommand, transitionCondition, layer);

    // Condition이 없는 버전
    public void MakeTransitions<FromStateType, ToStateType>(int transitionCommand,
        int layer = 0)
        where FromStateType : State<EntityType>
        where ToStateType : State<EntityType>
        => MakeTransitions<FromStateType, ToStateType>(transitionCommand, null, layer);

    // Condition이 없는 Enum Command 버전
    public void MakeTransitions<FromStateType, ToStateType>(Enum transitionCommand,
        int layer = 0)
        where FromStateType : State<EntityType>
        where ToStateType : State<EntityType>
        => MakeTransitions<FromStateType, ToStateType>(transitionCommand, null, layer);

    #endregion
    #region MakeAnyTransitions
    /// <summary>
    /// AnyTransition을 만듬
    /// </summary>
    /// <param name="canTransitionToSelf">스스로에게 전이 가능한지 여부</param>
    public void MakeAnyTransitions<ToStateType>(int transitionCommand,
        Func<State<EntityType>, bool> transitionCondition, int layer = 0, bool canTransitionToSelf = false)
        where ToStateType : State<EntityType>
    {
        var stateDatasByType = stateDatasByLayer[layer];
        // StateDats에서 ToStateType에 해당하는 State를 가진 StateData를 찾아와서 State를 가져옴
        var state = stateDatasByType[typeof(ToStateType)].State;
        // Transition 생성, 언제든지 조건만 맞으면 전이할 것이므로 FromState는 존재하지 않음
        var newTransition = new StateTransition<EntityType>(null, state, transitionCommand, transitionCondition, canTransitionToSelf);
        // Layer의 AnyTransition으로 추가
        anyTransitionsByLayer[layer].Add(newTransition);
    }

    // Command Enum버전
    public void MakeAnyTransitions<ToStateType>(Enum transitionCommand,
        Func<State<EntityType>, bool> transitionCondition, int layer = 0, bool canTransitionToSelf = false)
        where ToStateType : State<EntityType>
        => MakeAnyTransitions<ToStateType>(Convert.ToInt32(transitionCommand),transitionCondition,layer,canTransitionToSelf);

    // Command 없는 버전
    public void MakeAnyTransitions<ToStateType>(
        Func<State<EntityType>, bool> transitionCondition, int layer = 0, bool canTransitionToSelf = false)
        where ToStateType : State<EntityType>
        => MakeAnyTransitions<ToStateType>(StateTransition<EntityType>.kNullCommand, transitionCondition, layer, canTransitionToSelf);

    // Condition 없으면서 Command Enum 버전
    public void MakeAnyTransitions<ToStateType>(int transitionCommand,
        int layer = 0, bool canTransitionToSelf = false)
        where ToStateType : State<EntityType>
        => MakeAnyTransitions<ToStateType>(transitionCommand, null, layer, canTransitionToSelf);

    // Condition 없으면서 Command Enum 버전
    public void MakeAnyTransitions<ToStateType>(Enum transitionCommand,
        int layer = 0, bool canTransitionToSelf = false)
        where ToStateType : State<EntityType>
        => MakeAnyTransitions<ToStateType>(transitionCommand, null, layer, canTransitionToSelf);


    #endregion
    #region ExecuteCommand
    /// <summary>
    /// Command를 실행하는 함수, AniTransition들을 검사하여 해당 Command를 가지며 Condition을 만족하는 Transition을 검색
    /// AniTransition에서 못찾으면 현재 State의 Transition들을 대상으로 똑같이 검사
    /// </summary>
    /// <returns>커맨드를 호출하여 상태 전이에 성공하면 True</returns>
    public bool ExecuteCommand(int transitionCommand, int layer)
    {
        // Layer에 해당하는 AnyTransition들을 검사하여 커맨드와 컨디션을 모두 만족하는 트랜지션을 찾음
        var transition = anyTransitionsByLayer[layer].Find(x =>
            x.TransitionCommand == transitionCommand && x.IsTransferable);

        // AnyTransition에서 못찾았다면 현재 실행중인 State의 Transition에서 검사함
        transition ??= currentStateDatasByLayer[layer].Transitions.Find(x =>
            x.TransitionCommand == transitionCommand && x.IsTransferable);

        // 적합한 Transition을 못찾았다면 false리턴
        if (transition == null)
            return false;

        // 찾았다면 상태 전환
        ChangeState(transition.ToState, layer);
        return true;
    }

    // Enum버전
    public bool ExecuteCommand(Enum transitionCommand, int layer) 
        => ExecuteCommand(Convert.ToInt32(transitionCommand),layer);

    /// <summary>
    /// 모든 레이어를 대상으로 Command 실행
    /// </summary>
    /// <param name="transitionCommand">하나의 Layer라도 전이에 성공하면 True</param>
    /// <returns></returns>

    public bool ExecuteCommand(int transitionCommand)
    {
        bool isSuccess = false;

        foreach (int layer in layers)
        {
            // 상태 전이에 성공하면 isSuccess를 True로 전환
            if (ExecuteCommand(transitionCommand, layer))
                isSuccess = true;
        }

        return isSuccess;
    }

    #endregion
    #region SendMessage
    /// <summary>
    /// 현재 실행중인 상태에 메세지를 보냄
    /// </summary>
    /// <param name="message">메세지</param>
    /// <param name="layer">메세지를 보낼 상태가 있는 레이어</param>
    /// <param name="extraData">메세지와 함께 보낼 데이터, 데이터는 어떠한 형태도(투플이든, 리스트든) 받을 수 있도록 object형</param>
    /// <returns>메세지를 올바르게 수신하였다면 True 반환</returns>
    public bool SendMessage(int message, int layer, object extraData = null)
        => currentStateDatasByLayer[layer].State.OnReceiveMessage(message, extraData);
    // Command Enum버전
    public bool SendMessage(Enum message, int layer, object extraData = null)
    => SendMessage(Convert.ToInt32(message),layer, extraData);

    // 모든 Layer의 현재 실행중인 State를 대상으로 SendMessage 함수 실행
    public bool SendMessage(int message, object extraData = null)
    {
        bool isSuccess = false;
        foreach(int layer in layers)
        {
            //f(currentStateDatasByLayer[layer].State.OnReceiveMessage(message,extraData))
            if (SendMessage(message, layer, extraData))
                isSuccess = true;
        }
        return isSuccess;
    }

    public bool SendMessage(Enum message, object extraData = null)
        => SendMessage(Convert.ToInt32(message), extraData);
    #endregion
    #region 그 외 State 관련 함수들
    /// <summary>
    /// 모든 Layer를 대상으로 현재 실행중인 State 중 T Type의 State가 있는지 확인
    /// 예를 들어 State를 상속받은 RunState가 있다고 한다면 RunState가 현재 실행중인 State인지 확인
    /// </summary>
    /// <typeparam name="T">검사할 타입</typeparam>
    /// <returns>검사한 타입이 실행중인 State라면 True 반환</returns>
    public bool IsInState<T>() where T : State<EntityType>
    {
        foreach ((_, StateData data) in currentStateDatasByLayer)
        {
            if (data.State.GetType() == typeof(T))
                return true;
        }
        return false;
    }

    // 특정 Layer를 대상으로 현재 실행중인 State가 T Type의 State인지 확인
    public bool IsInState<T>(int layer) where T : State<EntityType>
        => currentStateDatasByLayer[layer].State.GetType() == typeof(T);
    
    /// <summary>
    /// 특정 Layer의 현재 실행중인 State 반환
    /// </summary>
    /// <returns></returns>
    public State<EntityType> GetCurrentState(int layer = 0) => currentStateDatasByLayer[layer].State;

    /// <summary>
    /// 특정 Layer의 현재 실행중인 State의 Type 반환
    /// </summary>
    /// <param name="layer"></param>
    /// <returns></returns>
    public Type GetCurrentStateType(int layer = 0) => GetCurrentState(layer).GetType();
    #endregion


    // 자식 class에서 정의할 State 추가 함수
    // 이 함수에서 AddState<T> 함수를 사용해 State를 추가해주면 됨
    protected virtual void AddStates() { }

    // 자식 class에서 정의할 Transition 생성 함수
    // 이 함수에서 MakeTransition 함수를 사용해 Transition을 만들어주면 됨
    protected virtual void MakeTransitions() { }
}

```

</div>
</details>



먼저 StateDate 내장 클래스가 있다.
이는 State 자체만을 이용하기에는 제한되는 부분이 많아서 StateData라는 내장 클래스를 만들어 State들을 관리한다.
Transition의 경우 `특정 State -> 특정 State` 의 Transition의 경우에는 `StateData클래스에서 리스트로 저장`하여 가지고 있고,
`아무 State -> 특정 State` 의 `AnyTransition`의 경우에는 `StateMachine이 리스트로 가지고`있는다.

```cs
// State별로 여러개의 Transition을 가지고 있을 수 있기 때문에 State별로 한 번에 묶어서 관리하기 위한
// 내장 클래스
private class StateData
{
    // State가 실행되는 Layer
    public int Layer { get; private set; }
    // State의 등록 순서 (가장 먼저 등록된 State가 시작 State)
    public int Priority { get; private set; }
    // Data가 가진 State
    public State<EntityType> State { get; private set; }
    // State에서 다른 State로 이어진 Transition들
    public List<StateTransition<EntityType>> Transitions { get; private set; } = new();

    public StateData(int layer, int priority, State<EntityType> state) =>
        (Layer, Priority, State) = (layer, priority, state);
}
```
<br>


StateMachine에서 State와 Transition 관리에 필요한 자료구조들이 있다.

```cs
// Layer별 가지고 있는 StateDatas(=Layer Dictionary), Dictionary의 key는 Value인 StateData가 가진 State의 Type
// 즉, State의 Type을 통해 해당 State가 가진 StateData를 찾아올 수 있음
// 여기서 말하는 Type이란 State를 상속하여 만든 State 자식 클래스들을 의미
// 예를 들어 State를 상속하는 RunState가 있다고 하면 
// stateDatasByLayer[0][RunState.GetType()]를 하므로 써 0번 레이어의 RunState의 부가정보를 담고있는 StateData를 가져옴
private readonly Dictionary<int, Dictionary<Type, StateData>> stateDatasByLayer = new();
// Layer별 Any Transitions(조건만 만족하면 언제든지 ToState로 전이되는 Transition)들을 가지고 있음
private readonly Dictionary<int, List<StateTransition<EntityType>>> anyTransitionsByLayer = new();

// Layer별로 현재 실행중인 State의 StateData
private readonly Dictionary<int, StateData> currentStateDatasByLayer = new();

// StateMachine에 존재하는 Layer들, 중복X 자동정렬O 를 위해 SortedSet이용
private readonly SortedSet<int> layers = new();
```

<br>



매 프레임 전이 조건을 확인하는 TryTransition 함수는 다음과 같다. <br>
Command가 있을 경우에는 이곳이 아닌 ExcecuteCommand가 호출되는 곳에서 전이 조건을 확인하기 때문에 <br>
TryTransition에서는 `Command가 없는 경우의 Condition만을 검사`한다.

```cs
private bool TryTransition(IReadOnlyList<StateTransition<EntityType>> transitions, int layer)
{
    foreach (var transition in transitions)
    {
        // 여기서 continue는 전이하지 않겠다를 의미
        // 첫 조건 커맨드가 존재하면 true, 두 번째 조건 컨디션이 없거나 True 반환
        // 즉, 커맨드가 있다면 여기서 말고 커맨드가 왔을 때 따로 처리하므로 여기서는 처리하지 않고 continue로 넘어가고,
        // 커맨드가 없다면 전이조건을 확인해서 전이할 수 없을 경우 continue로 넘어감
        if (transition.TransitionCommand != StateTransition<EntityType>.kNullCommand || !transition.IsTransferable)
            continue;

        // 스스로에게 전이 불가능할 때 스스로에게 전이하려고 하면 continue
        if (!transition.CanTransitionToSelf && currentStateDatasByLayer[layer].State == transition.ToState)
            continue;

        // 모든 조건을 만족한다면 ToState로 전이
        ChangeState(transition.ToState, layer);
        return true;
    }
    // 전이 실패
    return false;
}
```
<br>

command가 있는 경우 전이조건을 검사하는 함수는 다음과 같다.
```cs
public bool ExecuteCommand(int transitionCommand, int layer)
{
    // Layer에 해당하는 AnyTransition들을 검사하여 커맨드와 컨디션을 모두 만족하는 트랜지션을 찾음
    var transition = anyTransitionsByLayer[layer].Find(x =>
        x.TransitionCommand == transitionCommand && x.IsTransferable);

    // AnyTransition에서 못찾았다면 현재 실행중인 State의 Transition에서 검사함
    transition ??= currentStateDatasByLayer[layer].Transitions.Find(x =>
        x.TransitionCommand == transitionCommand && x.IsTransferable);

    // 적합한 Transition을 못찾았다면 false리턴
    if (transition == null)
        return false;

    // 찾았다면 상태 전환
    ChangeState(transition.ToState, layer);
    return true;
}
```
<br>

조건이 올바르다면 ChangeState함수를 실행한다.
State에서 만들었던 Enter, Exit등의 함수는 이곳에서 호출된다.

```cs
private void ChangeState(StateData newStateData)
{
    // Layer에 맞는 현재 실행중인 CurrentStateData를 가져옴
    var prevState = currentStateDatasByLayer[newStateData.Layer];

    // 처음 상태가 시작될 땐 실행중인 상대가 없으므로 ?.접근연산자를 사용
    prevState?.State.Exit();

    // 인자로 받은 상태로 전이해줌
    currentStateDatasByLayer[newStateData.Layer] = newStateData;
    newStateData.State.Enter();

    onStateChanged?.Invoke(this, newStateData.State, prevState.State, newStateData.Layer);
}

/// State와 Layer를 통해 전이하는 ChangeState 오버로딩함수
private void ChangeState(State<EntityType> newState, int layer)
{
    var newStateData = stateDatasByLayer[layer][newState.GetType()];
    ChangeState(newStateData);
}
```
<br>


마지막으로 Update문에서 전이조건을 확인하고 전이조건을 만족하지 못할 때, 현재 State의 Update를 호출한다.

```cs
/// 이 함수를 매 프레임 호출하여 이 곳에서 Condition을 통한 State 전이와 State의 Update함수를 실행
public void Update()
{
    foreach (var layer in layers)
    {
        // 현재 State의 StateData를 가져옴
        var currentStateData = currentStateDatasByLayer[layer];

        // 현재 Layer의 AnyTrasitions들을 가져옴
        bool hasAnyTransitions = anyTransitionsByLayer.TryGetValue(layer, out var anyTransitions);

        // AnyTransition으로 먼저 전이 시도를 해보고 안된다면 현재 StateData의 Transition을 통해 전이를 시도함
        // 즉, AnyTransition이 일반적인 Transition보다 우선됨
        // 전이에 성공했다면 다음 레이어에서 똑같은 작업 반복
        if ((hasAnyTransitions && TryTransition(anyTransitions, layer)) ||
            TryTransition(currentStateData.Transitions, layer))
            continue;

        // 전이에 실패한다면 현재 State의 Update함수를 실행
        currentStateData.State.Update();
    }
}
```


나머지 코드들은 State추가나 Transition 추가를 담당하는 함수, SendMessage함수 등의 여러 오버로딩 들이다.

### 사용 예시

걷기와 뛰기의 두가지 상태를 만들어줬다.

```cs
public class WalkState : State<Entity>
{
    public override void Enter()
    {
        Debug.Log("<color=green>걷기</color> 상태 진입");
    }
    public override void Update()
    {
        Debug.Log("<color=green>걷는</color> 중");
    }
    public override void Exit()
    {
        Debug.Log("<color=green>걷기</color> 상태 탈출");
    }
}
```


```cs
public class RunState : State<Entity>
{
    public override void Enter()
    {
        Debug.Log("<color=red>뛰기</color> 시작!");
    }

    public override void Update()
    {
        Debug.Log("<color=red>뛰는</color> 중");
    }

    public override void Exit()
    {
        Debug.Log("<color=red>뛰기</color> 탈출!");
    }
}
```
<br>

상태머신 또한 다음과 같이 간단하게 추가해주었다.

```cs
public class EntityStateMachine : StateMachine<Entity>
{
    protected override void AddStates()
    {
        AddStates<WalkState>();
        AddStates<RunState>();
    }

    protected override void MakeTransitions()
    {
        MakeTransitions<WalkState, RunState>(state => Owner.speed >= 5);
        MakeTransitions<RunState, WalkState>(state => Owner.speed < 5);
    }
}
```


<br>

이러면 이제 Entity라는 클래스를 만들고 간단하게 speed 변수만 shift키를 누름에 따라 바꿔주도록 하였다.

```cs
public class Entity : MonoBehaviour
{
    EntityStateMachine entityStateMachine = new EntityStateMachine();

    public float speed = 0;
    void Start()
    {
        entityStateMachine.Setup(this);
    }

    // Update is called once per frame
    void Update()
    {
        if(Input.GetKey(KeyCode.LeftShift)) 
            { speed = 10; }
        else
            { speed = 3; }

        entityStateMachine.Update();
    }
}
```

<br>

실행해보면 Shift키를 누르고 있을 경우 Entity의 speed가 바뀌게 되고 그로인해 상태가 변환되어 각 상태들의 Enter, Update, Exit함수가 실행됨을 볼 수 있다.

![state머신 테스트](https://github.com/novicehog/comments/assets/131991619/038e2e8c-ea72-4023-8a6c-749c1e548091)