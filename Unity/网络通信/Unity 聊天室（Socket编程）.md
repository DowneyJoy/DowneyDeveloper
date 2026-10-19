## 基于C# Socket的最基础服务端通信流程：

1. 创建服务器端Socket对象
2. 绑定IP地址和端口
3. 设置最大监听客户端数量
4. 等待客户端连接
5. 收发信息
6. 释放连接



### 创建服务端Socket对象

使用socket需要引用命名空间 using System.Net.Sockets;

```
//创建服务端socket
Socket serverSocket = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);
```

直接new一个Socket就可以，参数分别是：

1. IP地址类型。AddressFamily.InterNetwork是IPv4类地址，AddressFamily.InterNetworkV6是IPv6类地址，这里用的是IPv4地址。

2. socket类型。TCP协议对应的Socket类型是流类型，所以这个参数用的是SocketType.Stream。

3. 协议类型。选择TCP协议。

   

### 绑定IP地址和端口

使用socket提供的Bind方法绑定IP地址和端口。

这里需要传入一个EndPoint类型的参数，我们先定义一个IPEndPoint对象（IPEndPoint是继承了EndPoint对象的，可以作为Bind方法的参数），需要引用命名空间using System.Net;
直接new一个IPEndPoint对象，传入IP地址和端口号，IPAddress.Parse(“127.0.0.1”)可以将IP地址字符串转化成IPAddress，使用起来比较方便，端口号自定义需大于1024且没被使用过的端口号。
将iPEndPoint传入Bind方法中即可绑定IP地址和端口号。
127.0.0.1 （保留地址，用于测试）这个IP地址是代表本机的IP地址，由于便于测试，服务端和客户端的程序都在同一台计算机上，所以可以用这个地址。

```c#
//绑定IP地址和端口
IPEndPoint iPEndPoint = new IPEndPoint(IPAddress.Parse("127.0.0.1"), 8080);
serverSocket.Bind(iPEndPoint);
```



### 设置最大监听客户端数量

使用socket提供的Accept方法，等待客户端连入，这个方法是一个阻塞方法，会阻塞线程，当有客户端连入后才会继续执行该方法下面的逻辑，客户端连入成功后，返回一个与客户端通信的socket，返回的socket后面用于与客户端相互通信。

```
//等待客户端链接
Socket clientSocket = serverSocket.Accept();
```



