

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Net;
using System.Net.Sockets;
using System.Text;
using System.Threading.Tasks;

namespace GameServer.Network
{
    /// <summary>
    /// 负责监听TCP网络端口，异步接受Socket链接
    /// </summary>
    public class TcpSocketListener
    {

        private IPEndPoint endPoint;
        private Socket serverSocket;//服务端监听对象

        public event EventHandler<Socket> SocketConnected;//客户端接入事件

        public TcpSocketListener(string host,int port)
        {
            endPoint = new IPEndPoint(IPAddress.Parse(host), port);
        }

        public void Start()
        {
            if (!IsRunning)
            {
                serverSocket = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);
                serverSocket.Bind(endPoint);
                serverSocket.Listen();
                Console.WriteLine("开始监听端口：" + endPoint.Port);

                SocketAsyncEventArgs args = new SocketAsyncEventArgs();
                args.Completed += OnAccept;//有人连入
                serverSocket.AcceptAsync(args);
            }
        }

        private void OnAccept(object sender, SocketAsyncEventArgs e)
        {
            if(e.SocketError == SocketError.Success)
            {
                Socket client = e.AcceptSocket;
                if (client != null)
                {
                    SocketConnected?.Invoke(this, client);
                }
            }

            e.AcceptSocket = null;
            serverSocket.AcceptAsync(e);
        }

        public bool IsRunning
        {
            get
            {
                return serverSocket != null;
            }
        }

        public void Stop()
        {
            if (serverSocket == null)
                return;
            serverSocket.Close();
            serverSocket = null;
        }
    }
}

```

连接

```
using System;
using System.Net;
using System.Net.Sockets;
using System.Text;
using GameServer.Network;
using Network;

namespace GameServer
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello World!");
            TcpSocketListener listener = new TcpSocketListener("0.0.0.0", 32510);
            listener.SocketConnected += OnClientConnected;
            listener.Start();
            Console.ReadKey();
        }

        private static void OnClientConnected(object sender, Socket e)
        {
            var ip = e.RemoteEndPoint as IPEndPoint;
            Console.WriteLine("有客户端接入，对方IP及端口==="+ip.Address+":"+ip.Port);

            var lfd = new LengthFieldDecoder(e, 64 * 1024, 0, 4, 0, 4);
            lfd.DataReceived += Lfd_DataReceived;
            lfd.BeginAsyncReceive();
        }

        private static void Lfd_DataReceived(object sender, byte[] e)
        {
            string txt = Encoding.UTF8.GetString(e);
            Console.WriteLine(txt);
        }

    }
}

```

客户端连接

```
using System;
using System.Net;
using System.Net.Sockets;
using GameServer.Network;
namespace GameServer
{
    class Program
    {
        static void Main(string[] args)
        {
            var host = "127.0.0.1";
            int port = 32510;
            
            IPEndPoint ipEndPoint = new IPEndPoint(IPAddress.Parse(host),port);
            Socket socket = new Socket(AddressFamily.InterNetwork,SocketType.Stream,ProtocolType.Tcp);
            socket.Connect(ipEndPoint);
            
        }
        
        static void SendMessage(Socket socket,byte[] body)
        {
            int len = body.Length;
            byte[] lenBytes = BitConverter.GetBytes(len);
            socket.Send(lenBytes);
            socket.Send(body);
        }

    }
}
```

```
using Google.Protobuf;
using Proto.Message;
using System;
using System.Net;
using System.Net.Sockets;
using System.Text;

namespace NetClient
{
    class Program
    {
        static void Main(string[] args)
        {
            var host = "127.0.0.1";
            int port = 32510;

            IPEndPoint ipEndPoint = new IPEndPoint(IPAddress.Parse(host), port);
            Socket socket = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);
            socket.Connect(ipEndPoint);

            Console.WriteLine("成功连接到服务器");

            //protobuf 数据的序列化
            Vector3 v = new Vector3() { X = 100, Y = 200, Z = 300 };
            SendMessage(socket, v.ToByteArray());
        }

        static void SendMessage(Socket socket, byte[] body)
        {
            int len = body.Length;
            byte[] lenBytes = BitConverter.GetBytes(len);
            socket.Send(lenBytes);
            socket.Send(body);
        }
    }
}

```

