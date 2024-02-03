---
layout: single
title:  "유니티 FacePunchSteamworks 멀티 플레이"
categories: 
    - Unity
tag: [Unity, FacePunchSteamworks]
published: true

author_profile: true # 옆에뜨는 프로파일

search: true #검색해도 안뜨게함

date: 2024-02-02
last_modified_at: 2024-02-02

---

# 사용방법 정리

## 설치 방법
### FacePunchSteamworks
스팀에서 제공해주는 steamworks는 사용하기가 힘들기 때문에 개선하여 만들어진 FacePunchSteamworks를 사용.

[다운로드 링크](https://github.com/Facepunch/Facepunch.Steamworks)

![image](https://github.com/novicehog/comments/assets/131991619/efe18374-93be-42fb-8fd1-425c1a9daadb)

### multiplayer-community-contributions
멀티플레이 관련 코드와 자료들을 모아놓은 곳.
이곳에 facepunchsteamworks와 호환되는 것을 사용.

[다운로드 링크](multiplayer-community-contributions)

![image](https://github.com/novicehog/comments/assets/131991619/1cbddb16-bc95-487d-86e4-72612a70492b)


### 설치 순서
두가지 모두 압축을 풀어줌

![압축풀기](https://github.com/novicehog/comments/assets/131991619/69d7ac4b-3784-48d9-977b-448e9538043e)

<br>

여기서 net46과 netstandard2.0은 필요 없으므로 삭제

![image](https://github.com/novicehog/comments/assets/131991619/b027014f-4580-4212-8691-8d97c77db7fd)

<br>

그 다음 multiplayer-community-contributions-main -> Transports 에서 `com.community.netcode.transport.facepunch`만 밖으로 꺼내고 나머지는 삭제

![image](https://github.com/novicehog/comments/assets/131991619/e5d392cf-4ca1-4b79-8c6d-6594e6673b32)

<br>


이제 Unity 폴더 안에있는 내용물을 com.community.netcode.transport.facepunch -> RunTime -> FacePunch 폴더에 옮겨줌

![image](https://github.com/novicehog/comments/assets/131991619/52384124-f239-436b-8e3f-c78176504918)


![image](https://github.com/novicehog/comments/assets/131991619/e181c213-36dc-4ab4-87f7-b65ddeac1f8e)

<br>

이제 폴더의 이름을 바꿔주고 유니티 프로젝트의 에셋폴더에 넣어줌

![이름바꾸기](https://github.com/novicehog/comments/assets/131991619/6591b087-f2ad-4dcd-ac41-b3f11f0f699a)

![에셋에넣기](https://github.com/novicehog/comments/assets/131991619/38f4cbf6-66f6-4805-8f93-844ed550b304)

<br>

이제 콘솔창에 오류가 발생하는데

![image](https://github.com/novicehog/comments/assets/131991619/cb408ef0-8a13-4c16-a9de-2ea7074168f5)

이것은 넷코드를 임포트해주면 해결됨.

Window -> Package Manager 에서 Unity Registry에서 Netcode for GameObject를 설치해주면 됨

![넷코드설치](https://github.com/novicehog/comments/assets/131991619/dac0137a-8e5e-401d-b1c5-aefdbf28dc9d)


## 기본 세팅
빈 객체 NetworkManager를 만들고 NetworkManager 스크립트를 추가

![networkManager](https://github.com/novicehog/comments/assets/131991619/2036d660-830c-4b4d-8240-4f277f5c929a)

<br>

NetworkManager 컴포넌트에서 select transport를 FacePunchTransfort로 해줌

![트랜스포트 설정](https://github.com/novicehog/comments/assets/131991619/3f82bb3d-d78d-4bd1-a25f-483ed1836368)

이제 게임을 실행해보면 steam에서 자동으로 Spacewar(개발자용 테스트 게임)이 실행되며 연동되는 모습을 볼 수 있음.

![스팀연동](https://github.com/novicehog/comments/assets/131991619/a8ba807a-f92b-4843-8ab6-79808c678fc8)



## Steamworks 

### 콜백함수
#### OnLobbyCreated;

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


#### OnLobbyEntered;
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



