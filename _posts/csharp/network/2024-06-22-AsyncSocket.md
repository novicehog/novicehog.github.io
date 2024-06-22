---
layout: single
title:  "[C#] AsyncSocket"
categories: 
    - C Sharp
tag: [c#, network]
published: true

author_profile: true # 옆에뜨는 프로파일

date: 2024-06-22
last_modified_at: 2024-06-22
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
이 글은 Rookiss님의  **[C#과 유니티로 만드는 MMORPG 게임 개발 시리즈] Part4: 게임 서버**를 보고 헷갈리는 부분 위주로 정리한 글입니다. 틀리거나 부족한 부분이 있을 수 있습니다.
{: .notice--warning}

## 비동기 Socket 프로그래밍
[비동기x socket프로그래밍](https://novicehog.github.io/c%20sharp/Socket/)

전에 만든 socket프로그래밍은 Accept, recv, send 등 통신이 일어날 때 무식하게 상대방을 기다리는 `blocking방식`이였다.<br>
이러한 방식은 직관적이여서 코드 흐름을 이해하기는 쉽지만 Blocking 함수가 나올 때 마다 상대방의 응답이 있기 전까지 `멈춰버리는 매우 큰 단점`이 있다.<br>
이 방식을 해결하기 위해 비동기적 socket프로그래밍 방식으로 코드의 수정이 필요하다.

- `동기`: 클라이언트가 서버에 또는 서버가 클라이언트에 요청을 보내면 다른 작업을 할 수 없음.
- `비동기`: 클라이언트가 서버에 또는 서버가 클라이언트에 요청을 보내고 다른 작업을 할 수 있다. 응답을 기다리지 않고 다른 작업을 수행할 수 있다.


### 비동기 Accept 
Accept의 비동기적 처리를 담당하는 listener 클래스를 새로 정의해서 리팩토링하며 구현한다.

리스너 클래스

```cs
internal class Listener
{
    Socket _listenSocket;
    Action<Socket> _onAcceptHandler;

    // 서버를 열 주소, Accept 응답 시 실행할 Action을 인자로 받음
    public void Init(IPEndPoint endPoint, Action<Socket> onAcceptHandler)
    {
        _listenSocket = new Socket(endPoint.AddressFamily, SocketType.Stream, ProtocolType.Tcp);
        _onAcceptHandler += onAcceptHandler;

        // 서버 열기
        _listenSocket.Bind(endPoint);
        _listenSocket.Listen(10);

        // 소켓 비동기 이벤트
        SocketAsyncEventArgs args = new SocketAsyncEventArgs();
        // 비동기 응답이 오면 OnAcceptCompleted 함수를 실행하도록 함
        args.Completed += new EventHandler<SocketAsyncEventArgs>(OnAcceptCompleted);
        RegisterAccept(args);
    }

    void RegisterAccept(SocketAsyncEventArgs args)
    {
        // 전에 등록된 소켓 정보를 초기화
        args.AcceptSocket = null;

        // 비동기적 Accept실행, 비동기적 응답이면 true반환
        bool pending = _listenSocket.AcceptAsync(args);

        // 딜레이없이 동기적으로 응답이 왔다면 직접 OnAcceptCompleted 함수를 실행해줌
        if (pending == false)
            OnAcceptCompleted(null, args);   
    }

    void OnAcceptCompleted(object sender, SocketAsyncEventArgs args)
    {
        // Socket에 문제가 없다면 응답 처리 함수를 실행
        if(args.SocketError == SocketError.Success)
        {
            _onAcceptHandler.Invoke(args.AcceptSocket);
        }
        else
            Console.WriteLine(args.SocketError.ToString());

        RegisterAccept(args);
    }
}
```

<br>

변경된 메인 코드

```cs
internal class Program
{
    static Listener _listener = new Listener();

    // 응답 처리 부분을 따로 함수로 만듬
    static void OnAcceptHandler(Socket clientSocket)
    {
        try
        {
            // 받는다
            byte[] recvBuff = new byte[1024];           // 저장 공간
            int recvBytes = clientSocket.Receive(recvBuff); // 데이터를 받음
            string recvData = Encoding.UTF8.GetString(recvBuff, 0, recvBytes); // 데이터를 인코딩
            Console.WriteLine($"[From Client] {recvData}");

            // 보낸다
            byte[] sendBuff = Encoding.UTF8.GetBytes("Welcome to MMORPG SERVER !");
            clientSocket.Send(sendBuff);

            // 쫓아내기
            clientSocket.Shutdown(SocketShutdown.Both);
            clientSocket.Close();
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.ToString());
        }
    }
    static void Main(string[] args, Action<Socket> onAceeptHandler)
    {
        // DNS (Domain Name System)
        string host = Dns.GetHostName();
        IPHostEntry ipHost = Dns.GetHostEntry(host);
        IPAddress ipAddr = ipHost.AddressList[0];
        IPEndPoint endPoint = new IPEndPoint(ipAddr, 7777);

        // listner 클래스 init
        _listener.Init(endPoint, OnAcceptHandler);

        // 프로그램이 끝나지 않도록 무한 루프
        while (true)
        {
        }
    }
}
```


