## 自定义报文协议

![image-20240202095523157](C:\Users\Tangning\AppData\Roaming\Typora\typora-user-images\image-20240202095523157.png)

| 魔数               | 长度字段         | 其他             | 消息内容         |
| ------------------ | ---------------- | ---------------- | ---------------- |
| 2                  | 4                | -4/0/10（偏移）  | 可变             |
| 长度字段开始的位置 | 长度字段本身长度 | 头和消息之间多远 | 最后从头丢弃多少 |

## 粘包和拆包

```C#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Net.Sockets;
using System.Text;
using System.Threading.Tasks;

namespace Network
{
    /// <summary>
    /// 这是Socket异步接受器，可以对接收的数据粘包与拆包
    /// 事件委托：
    /// Received：数据包接收完成事件，参数为接受的数据包
    /// ConnectFailed：接收异常事件
    /// </summary>
    public class LengthFieldDecoder
    {
        private Socket mSocket;

        private int lengthFieldOffset = 8; //第几个是body长度字段
        private int lengthFieldLength = 4; //长度字段占几个字节

        /// <summary>
        /// 长度字段和内容之间距离几个字节
        /// 负数代表向前偏移，body实际长度要减去这个绝对值
        /// </summary>
        private int lengthAdjustment = 0;
        private int initialBytesToStrip = 0; //结果数据中前几个字节不需要的字节数

        private byte[] mBuffer; //接收数据的缓存空间
        private int mOffset; //读取位置

        /// <summary>
        /// 一次性接收数据的最大字节，默认64K
        /// </summary>
        private int mSize = 64 * 1024;

        //成功收到消息的委托事件
        public event EventHandler<byte[]> DataReceived;
        //连接断开的委托事件
        public delegate void DisconnectedEventHandler();
        public event DisconnectedEventHandler Disconnected;

        public LengthFieldDecoder(Socket socket,int lengthFieldOffset,int lengthFieldLength)
        {
            mSocket = socket;
            this.lengthFieldOffset = lengthFieldOffset;
            this.lengthFieldLength = lengthFieldLength;
            mBuffer = new byte[mSize];
        }

        public LengthFieldDecoder(Socket socket,int maxBufferLength,int lengthFieldOffset,int lengthFieldLength,int lengthAdjustment,int initialBytesToStrip)
        {
            mSocket = socket;
            mSize = maxBufferLength;
            this.lengthFieldOffset = lengthFieldOffset;
            this.lengthFieldLength = lengthFieldLength;
            this.lengthAdjustment = lengthAdjustment;
            this.initialBytesToStrip = initialBytesToStrip;
            mBuffer = new byte[mSize];
        }

        public void BeginAsyncReceive()
        {
            //开始接收
            mSocket.BeginReceive(mBuffer, mOffset, mSize - mOffset, SocketFlags.None, new AsyncCallback(Receive), null);
        }

        private void Receive(IAsyncResult result)
        {
            int len = mSocket.EndReceive(result);
            // 0代表连接失败
            if (len == 0)
            {
                if(Disconnected != null)
                {
                    Disconnected();
                }
                return;
            }
            OnReceivceData(len);

            //继续接收数据
            BeginAsyncReceive();
        }

        private void OnReceivceData(int len)
        {
            int headLen = lengthFieldOffset + lengthFieldLength;
            int adj = lengthAdjustment; //body偏移量

            //size是待处理的数据长度，mOffset每次都从0开始
            //循环开始之前mOffset代表上次剩余长度
            int size = len;
            if (mOffset > 0)
            {
                size += mOffset;
                mOffset = 0;
            }
            //循环解析
            while (true)
            {
                int remain = size = mOffset;//剩余未处理的长度

                //如果未处理的数据超出限制
                if (remain > mSize)
                {
                    throw new IndexOutOfRangeException();
                }
                if(remain < headLen)
                {
                    //接收的数据不够一个完整的数据包，继续接收
                    Array.Copy(mBuffer, mOffset, mBuffer, 0, remain);
                    mOffset = remain;
                    return;
                }

                //获取包长度
                int bodyLen = BitConverter.ToInt32(mBuffer, mOffset + lengthFieldOffset);
                if (remain < headLen + adj + bodyLen)
                {
                    //接收的数据不够一个完整的包，继续接收
                    Array.Copy(mBuffer, mOffset, mBuffer, 0, remain);
                    mOffset = remain;
                    return;
                }

                //body的读取位置
                int bodyStart = mOffset = Math.Max(headLen, headLen + adj);
                //body的真实长度
                int bodyCount = Math.Min(bodyLen, bodyLen + adj);

                //获取包体
                int total = headLen + adj + bodyLen;//数据包总长度
                int count = total - initialBytesToStrip;
                byte[] data = new byte[count];
                Array.Copy(mBuffer, mOffset+initialBytesToStrip, data, 0, count);
                mOffset += total;

                //完成一个数据包
                if(DataReceived != null)
                {
                    DataReceived(this, data);
                }
            }
        }
    }
}

```

参数说明（长度字段解码器）

- lengthFieldOffset 长度字段的位置下标
- lengthFieldLength 长度字段本身的长度，只推荐1，2，4，8
- lengthAdjustment 偏移位，长度字节和内容中间隔了几个字节，可为负数
- initialBytesToStrip 表示获取一个完整的数据包之后，舍弃前面的几个字节



## C#异步接收API

1. AcceptAsync 
   1. 只有一个参数处理所有连接（资源利用更高）
   2. 无论循环接收多少次，都是重复利用同一个对象

2. BeginAccept+EndAccept  
   1. 每一次连接都会生成一个事件
   2. 每次循环都会创建新的异步结果对象

[System.Net.Sockets 命令空间官方文档](https://learn.microsoft.com/zh-cn/dotnet/api/system.net.sockets.socket?view=net-8.0&redirectedfrom=MSDN)



## Protobuf

Protobuf是Google公司提出的一种轻便高效的数据存储格式，常用于处理结构化数据，具有语言无关，平台无关，可扩展性特性，常用于通讯协议，服务端数据交换场景。

性能高效：与XML相比，protobuf更小（3-10倍），更快（20-100倍）、更为简单

语言无关、平台无关：protobuf支持Java、C++、Python、C#等多种语言，与平台无关

扩展性强、兼容性强：描述简单、更新消息结构时过程简单，且不会破坏原有程序。



序列化：把程序对象转化为字节数组或文字

反序列化：把存储的字节或者文字转化为程序对象。

message.proto

```protobuf
syntax = "proto3";

package Proto.Message;

message Vector3{
	int32 x = 1;
	int32 y = 2;
	int32 z = 3;
}

message Entity{
	int32 id =1;
	int32 speed = 2;
	Vector3 position = 3;
	Vector3 rotation = 4;
}
```

### 批处理工具

```bat
@echo off

set foo="%cd%\out\csharp"

if not exist %foo% (
	md %foo%
)

::遍历文件
for %%i in (proto/*.proto) do (
	%cd%\bin\protoc -I=proto --csharp_out=out/csharp %%i
	echo %%i Done
)
```

### 序列化反序列化

```
//protobuf 数据的序列化
Vector3 v = new Vector3() { X = 100, Y = 200, Z = 300 };
SendMessage(socket, v.ToByteArray());

//反序列化 e服务器接受的byte[]
Vector3 v = Vector3.Parser.ParseFrom(e);
```

## 消息分发器MessageRouter

队列，入队，多线程分发（线程池）

订阅，退订，触发订阅，消息频道

用户模块，角色模块，物品模块，地图模块，战斗模块

![image-20240204111454629](C:\Users\Tangning\AppData\Roaming\Typora\typora-user-images\image-20240204111454629.png)

## 网络服务NetService

```
using Network;
using Proto.Message;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Net;
using System.Net.Sockets;
using System.Text;
using System.Threading.Tasks;

namespace GameServer.Network
{
    public class NetService
    {
        TcpSocketListener listener = null;
        public NetService() { }

        public void Init(int port)
        {
            listener = new TcpSocketListener("0.0.0.0", port);
            listener.SocketConnected += OnClientConnected;
        }

        public void Start()
        {
            listener.Start();
        }

        private void OnClientConnected(object sender, Socket socket)
        {
            new NetConnection(socket,
                new NetConnection.DataReceivedCallback(OnDataReceived),
                new NetConnection.DisconnectedCallback(OnDisconnected));
            
        }

        private void OnDisconnected(NetConnection sender)
        {
            Console.WriteLine("连接断开");
        }

        private void OnDataReceived(NetConnection sender, byte[] data)
        {
            //string txt = Encoding.UTF8.GetString(data);
            //Console.WriteLine(txt);
            Vector3 v = Vector3.Parser.ParseFrom(data);
            Console.WriteLine("x=" + v.X + " y=" + v.Y + " z=" + v.Z);
        }
    }
}

```

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
            NetService netService = new NetService();
            netService.Init(32510);
            netService.Start();

            Console.ReadKey();
        }

    }
}

```

## 网络连接类NetConnection

发送消息，接收消息，关闭连接，断开通知

```
using Network;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Net.Sockets;
using System.Text;
using System.Threading.Tasks;

namespace GameServer.Network
{
    /// <summary>
    /// 客户端网络连接
    /// 发送消息，接收消息回调，关闭连接，断开通知回调
    /// </summary>
    public class NetConnection
    {
        public delegate void DataReceivedCallback(NetConnection sender, byte[] data);
        public delegate void DisconnectedCallback(NetConnection sender);

        public Socket socket;
        private DataReceivedCallback dataReceivedCallback;
        private DisconnectedCallback disconnectedCallback;

        public NetConnection(Socket socket,DataReceivedCallback cb1,DisconnectedCallback cb2)
        {
            this.socket = socket;
            this.dataReceivedCallback = cb1;
            this.disconnectedCallback = cb2;
            //创建解码器
            var lfd = new LengthFieldDecoder(socket, 64 * 1024, 0, 4, 0, 4);
            lfd.DataReceived += OnDataReceived;
            lfd.Disconnected += ()=> disconnectedCallback(this); 
            lfd.BeginAsyncReceive();
        }

        private void OnDataReceived(object sender, byte[] buffer)
        {
            dataReceivedCallback(this, buffer);
        }
        /// <summary>
        /// 主动关闭连接
        /// </summary>
        public void Close()
        {
            try
            {
                socket.Shutdown(SocketShutdown.Both);
            }
            catch { }
            disconnectedCallback(this);
            socket.Close();
            socket = null;
        }
    }
}

```

 
