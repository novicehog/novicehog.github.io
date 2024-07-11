---
layout: single
title:  "[C#] Socket 서버 만들기"
categories: 
    - C Sharp
tag: [c#, network]
published: true

author_profile: true # 옆에뜨는 프로파일

date: 2024-06-22
last_modified_at: 2024-06-24
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


## 리팩토링
[간단한 socket프로그래밍](https://novicehog.github.io/c%20sharp/Socket/)

저번에 socket을 이용해서 간단한 통신을 해보았는데, 이번에는 여러 문제점을 고친다.

1. 동기 방식을 비동기로 전환
2. 기능별로 클래스를 나눠서 가독성을 높임
3. 서버 엔진 코드와, 컨텐츠 부분의 코드를 분리

### 비동기 Socket 프로그래밍
전에 만든 socket프로그래밍은 Accept, recv, send, connect 등 통신이 일어날 때 무식하게 상대방을 기다리는 `blocking방식`이였다.<br>
이러한 방식은 직관적이여서 코드 흐름을 이해하기는 쉽지만 Blocking 함수가 나올 때 마다 상대방의 응답이 있기 전까지 `멈춰버리는 매우 큰 단점`이 있다.<br>
이 방식을 해결하기 위해 비동기적 socket프로그래밍 방식으로 코드의 수정이 필요하다.

- `동기`: 클라이언트가 서버에 또는 서버가 클라이언트에 요청을 보내면 다른 작업을 할 수 없음.
- `비동기`: 클라이언트가 서버에 또는 서버가 클라이언트에 요청을 보내고 다른 작업을 할 수 있다. 응답을 기다리지 않고 다른 작업을 수행할 수 있다.

Socket 비동기 함수 역시 지원되기 때문의 함수를 이용해 구현한다. 비동기 방식으로 구현할 경우
함수의 실행과, 응답 후 처리가 동시에 실행되지 않기 때문에, `이벤트를 이용해서 응답 후 처리 부분을 구현`해주어야 한다. <br>
그래서 모든 `Async 함수들은 인자값으로 SocketAsyncEventArgs 클래스를 입력받는다.`<br>
이 클래스를 통해 `응답이 왔을 때 실행되어야 할 코드를 우리가 지정`해 줄 수 있다.

### 기능별로 클래스 나누기
Accept를 담당하는 Listener클래스를 만들고 <br>
연결이 된 상태에서 여러 기능(Recv, Send,Disconnect)을 담당할 Session클래스,<br>
Connect를 담당하는 Connector클래스를 만든다.


### 코드 분리
완성된 ServerCore코드를 라이브러리화 시키고 실질적인 사용은 다른 곳 함.

### ServerCore 전체 코드
#### Listener (Accept 담당)
```cs
public class Listener
{
    Socket _listenSocket;
    Func<Session> _sessionFactory;

    public void Init(IPEndPoint endPoint, Func<Session> sessionFactory)
    {
        _listenSocket = new Socket(endPoint.AddressFamily, SocketType.Stream, ProtocolType.Tcp);
        _sessionFactory += sessionFactory;

        _listenSocket.Bind(endPoint);

        _listenSocket.Listen(10);

        // RegisterAccept 함수와 OnAcceptCompleted함수가 순환되며 반복되는 구조
        SocketAsyncEventArgs args = new SocketAsyncEventArgs();
        args.Completed += new EventHandler<SocketAsyncEventArgs>(OnAcceptCompleted);
        RegisterAccept(args);
    }


    
    void RegisterAccept(SocketAsyncEventArgs args)
    {
        args.AcceptSocket = null;

        bool pending = _listenSocket.AcceptAsync(args);

        // 딜레이없이 동기적으로 실행됨
        if (pending == false)
            OnAcceptCompleted(null, args);
            
    }

    void OnAcceptCompleted(object sender, SocketAsyncEventArgs args)
    {
        if(args.SocketError == SocketError.Success)
        {
            // 세션은 연결된 상태일 때 이용할 기능들이 있는 클래스
            Session session = _sessionFactory.Invoke();
            session.Start(args.AcceptSocket);
            session.OnConnected(args.AcceptSocket.RemoteEndPoint);
        }
        else
            Console.WriteLine(args.SocketError.ToString());

        RegisterAccept(args);
    }
}
```
#### Sessiong (연결될 때 송 수신을 담당)
```cs
public abstract class Session
{
    Socket _socket;         
    int _disconnected = 0;  // 0이면 연결중 1이면 이미 끊어진 상태

    object _lock = new object();
    // Send와 관련된 필드들
    // 이미 SendAsync가 실행 돼었을 때 Send가 추가로 호출될 경우 보낼 메세지을 잠깐 담아둘 곳
    Queue<byte[]> _sendQueue = new Queue<byte[]>();                 
    // Send에 담겨서 보내지는 메세지들 
    List<ArraySegment<byte>> _pendingList = new List<ArraySegment<byte>>(); 
    SocketAsyncEventArgs _sendArgs = new SocketAsyncEventArgs();    // Send전용 SocketAsyncEventArgs

    SocketAsyncEventArgs _recvArgs = new SocketAsyncEventArgs();    // Recv전용 SocketAsyncEventArgs

    // 자식 클래스에서 구현할 각 타이밍에 호출되는 추상 함수들
    public abstract void  OnConnected(EndPoint endPoint);       // 열결될 때 호출
    public abstract void  OnRecv(ArraySegment<byte> buffer);    // 메세지를 받을 때 호출
    public abstract void  OnSend(int numOfBytes);               // 보낼 때 호출
    public abstract void  OnDisconnected(EndPoint endPoint);    // 연결이 끊길 때 호출

    // 세션의 초기화와 Recv 함수가 시작됨
    public void Start(Socket socket)
    {
        _socket = socket;

        _recvArgs.Completed += new EventHandler<SocketAsyncEventArgs>(OnRecvCompleted);
        _recvArgs.SetBuffer(new byte[1024],0,1024);

        _sendArgs.Completed += new EventHandler<SocketAsyncEventArgs>(OnSendCompleted);

        RegisterRecv();
    }

    // 메세지를 보냄, 동시 접근되는걸 막기 위해 lock사용
    public void Send(byte[] sendBuff)
    {
        lock (_lock)
        {
            _sendQueue.Enqueue(sendBuff);
            // 이미 데이터가 전송중이지 않을 경우에만 RegisterSend실행
            if (_pendingList.Count == 0)
                RegisterSend();
        }
    }

    public void Disconnect()
    {
        // 만든 disconnected flag를 1(연결이 끊김)로함
        // 이미 1일 경우 끊어진 상태이므로 리턴을 함
        if (Interlocked.Exchange(ref _disconnected, 1) == 1)
            return;

        OnDisconnected(_socket.RemoteEndPoint);

        _socket.Shutdown(SocketShutdown.Both);
        _socket.Close();
    }

    #region 네트워크 통신

    void RegisterSend()
    {
        while (_sendQueue.Count > 0) 
        {
            byte[] buff = _sendQueue.Dequeue();
            _pendingList.Add(new ArraySegment<byte>(buff, 0, buff.Length));
        }

        // BufferList의 경우 직접Add하는것이 아닌 list를 만들어서 대입해야 오류가 없음
        _sendArgs.BufferList = _pendingList;

        bool pending = _socket.SendAsync(_sendArgs);
        if (pending == false)
            OnSendCompleted(null, _sendArgs);
    }

    void OnSendCompleted(object sender, SocketAsyncEventArgs args)
    {
        lock (_lock)
        {
            if (args.BytesTransferred > 0 && args.SocketError == SocketError.Success)
            {
                try
                {
                    _sendArgs.BufferList = null;
                    _pendingList.Clear();

                    OnSend(_sendArgs.BytesTransferred);

                    if (_sendQueue.Count > 0)
                    {
                        RegisterSend();
                    }
                    
                }
                catch (Exception e)
                {
                    Console.WriteLine($"OnSendCompleted Failed{e}");
                }
            }
            else
            {
                Disconnect();
            }
        }
    }

    void RegisterRecv()
    {
        bool pending = _socket.ReceiveAsync(_recvArgs);
        if (pending == false)
            OnRecvCompleted(null, _recvArgs);
    }

    void OnRecvCompleted(object sender, SocketAsyncEventArgs args)
    {
        if(args.BytesTransferred > 0 && args.SocketError == SocketError.Success)
        {
            try
            {
                OnRecv(new ArraySegment<byte>(args.Buffer, args.Offset, args.BytesTransferred));

                RegisterRecv();
            }
            catch (Exception e)
            {   
                Console.WriteLine($"OnRecvCompleted Failed{e}");
            }
        }
        else
        {

        }
    }
    #endregion
}
```

#### Connector (Connert 담당)

```cs
public class Connector
{
    Func<Session> _sessionFactory; // 만들어질 세션의 타입을 반환하는 함수
    public void Connect(IPEndPoint endPoint, Func<Session> sessionFactory)
    {
        Socket socket = new Socket(endPoint.AddressFamily, SocketType.Stream, ProtocolType.Tcp);
        _sessionFactory = sessionFactory;

        SocketAsyncEventArgs args = new SocketAsyncEventArgs();
        args.Completed += OnConnectCompleted;
        args.RemoteEndPoint = endPoint;
        args.UserToken = socket;

        RegisterConnect(args);
    }

    void RegisterConnect(SocketAsyncEventArgs args)
    {
        Socket socket = args.UserToken as Socket;
        if (socket == null)
            return;

        bool pending = socket.ConnectAsync(args);
        if (pending == false)
            OnConnectCompleted(null, args);
    }

    void OnConnectCompleted(object sender, SocketAsyncEventArgs args)
    {
        if (args.SocketError == SocketError.Success) 
        {
            // 연결에 성공하면 세션을 만들고 사용 
            Session session = _sessionFactory.Invoke();
            session.Start(args.ConnectSocket);
            session.OnConnected(args.RemoteEndPoint);
        }
        else
        {
            Console.WriteLine($"OnConnectCompleted Failed { args.SocketError}");
        }
    }
}
```


### Server 전체 코드
라이브러리화 된 ServerCore를 사용하는 코드

```cs
// 서버에 맞게 Sessiond을 상속받아 추상함수 구현
class GameSession : Session
{
    public override void OnConnected(EndPoint endPoint)
    {
        Console.WriteLine($"OnConnected: {endPoint}");

        byte[] sendBuff = Encoding.UTF8.GetBytes("Welcome to MMORPG SERVER !");
        Send(sendBuff);

        Thread.Sleep(1000);

        Disconnect();
    }

    public override void OnDisconnected(EndPoint endPoint)
    {
        Console.WriteLine($"OnDisconnected: {endPoint}");
    }

    public override void OnRecv(ArraySegment<byte> buffer)
    {
        string recvData = Encoding.UTF8.GetString(buffer.Array, buffer.Offset, buffer.Count); // 데이터를 인코딩
        Console.WriteLine($"[From Client] {recvData}");
    }

    public override void OnSend(int numOfBytes)
    {
        Console.WriteLine($"Transferred bytes: {numOfBytes}");
    }
}
internal class Program
{
    static Listener _listener = new Listener();
    static void Main(string[] args)
    {
        // DNS (Domain Name System)
        string host = Dns.GetHostName();
        IPHostEntry ipHost = Dns.GetHostEntry(host);
        IPAddress ipAddr = ipHost.AddressList[0];
        IPEndPoint endPoint = new IPEndPoint(ipAddr, 7777);

        // 서버를 열음
        _listener.Init(endPoint, () => { return new GameSession(); });

        Console.WriteLine("Listening...");
        while (true)
        {
        }
    }
}
```

### DummyClient 전체 코드
```cs
// 클라이언트에 맞게 Sessiond을 상속받아 추상함수 구현
class GameSession : Session
{
    public override void OnConnected(EndPoint endPoint)
    {
        Console.WriteLine($"OnConnected: {endPoint}");

        // 보낸다
        for (int i = 0; i < 5; i++)
        {
            byte[] sendBuff = Encoding.UTF8.GetBytes($"Hello World {i}");
            Send(sendBuff);
        }
    }

    public override void OnDisconnected(EndPoint endPoint)
    {
        Console.WriteLine($"OnDisconnected: {endPoint}");
    }

    public override void OnRecv(ArraySegment<byte> buffer)
    {
        string recvData = Encoding.UTF8.GetString(buffer.Array, buffer.Offset, buffer.Count); // 데이터를 인코딩
        Console.WriteLine($"[From Server] {recvData}");
    }

    public override void OnSend(int numOfBytes)
    {
        Console.WriteLine($"Transferred bytes: {numOfBytes}");
    }
}

internal class Program
{
    static void Main(string[] args)
    {
        // DNS (Domain Name System)
        string host = Dns.GetHostName();
        IPHostEntry ipHost = Dns.GetHostEntry(host);
        IPAddress ipAddr = ipHost.AddressList[0];
        IPEndPoint endPoint = new IPEndPoint(ipAddr, 7777);

        // 서버에 연결 
        Connector connector = new Connector();

        connector.Connect(endPoint, () => { return new GameSession(); });

        while (true)
        {
            Thread.Sleep(1000);
        }
    }
}
```