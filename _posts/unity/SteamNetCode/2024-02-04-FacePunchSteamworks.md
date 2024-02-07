---
layout: single
title:  "유니티 Netcode 정리"
categories: 
    - Unity
tag: [Unity, Netcode]
published: true

author_profile: true # 옆에뜨는 프로파일

search: true #검색해도 안뜨게함

date: 2024-02-04
last_modified_at: 2024-02-04

---

# Steamworks 

## 콜백함수
### OnLobbyCreated;

```cs
public static event Action<Result, Lobby> OnLobbyCreated;
```

로비가 만들어 질 때 호출되며 보통 방장이 실행. 매개변수로 `Result`와 `Lobby`를 넘겨줌.

`Result`는 로비 생성의 결과와 관련된 enum 형태의 정보이고,<br>
`Lobby`는 생성된 로비의 정보를 가지고 있는 구조체이다. 로비의 주인, 참여한 플레이어 수 등등 로비와 관련된 많은 정보들이 있음.

<details>
<summary>예시</summary>
<div markdown="1">


```cs
using Steamworks;
using Steamworks.Data;
using Unity.Netcode;

private void OnEnable()
{
    SteamMatchmaking.OnLobbyCreated += LobbyCreated;
}

private void LobbyCreated(Result result, Lobby lobby)
{
    if(result == Result.OK) // 로비 생성이 정상적으로 되었으면
    {
        lobby.SetPublic();          // 로비 공개방 전환
        lobby.SetJoinable(true);    // 참여 가능 설정

        NetworkManager.Singleton.StartHost();      // 넷코드 Host 시작
    }
}
```

</div>
</details>


### OnLobbyEntered;
```cs
public static event Action<Lobby> OnLobbyEntered;   
```
`로비에 참여시 호출`됨. 참여하는 클라이언트 뿐 아니라 lobby를 생성하는 방장역할도 로비에 생성후 참여하기 때문에
호출됨.

매개변수로 `Lobby`를 받음. `Lobby`는 현재 참여한 로비의 정보를 가지고 있음.


<details>
<summary>예시</summary>
<div markdown="1">


```cs
using Steamworks;
using Steamworks.Data;
using Unity.Netcode;
using Netcode.Transports.Facepunch;

private void OnEnable()
{
    SteamMatchmaking.OnLobbyEntered += LobbyEntered;  
}

private void LobbyEntered(Lobby lobby)
{
    // 방장일 경우 실행안함
    if (NetworkManager.Singleton.IsHost) return;

    // 스팀과 넷코드는 별개이므로 스팀 로비에 참여하여 연결되었다 해도 넷코드로 연결하려면 넷코드는 추가적인 정보가 필요함
    // FacepunchTransport의 targetSteamId 를 로비의 주인으로 함
    // targetSteamId를 통해 넷코드 연결을 함
    NetworkManager.Singleton.gameObject.GetComponent<FacepunchTransport>().targetSteamId = lobby.Owner.Id;
    NetworkManager.Singleton.StartClient();
}
```

</div>
</details>







