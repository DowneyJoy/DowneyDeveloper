日志：`心跳超时: keepalive 30s，已47s未收到任何报文，判定离线`

> PubSubClient 的机制：**客户端必须在 keepalive 时间内，至少有一次收发报文（publish/subscribe/ 服务端下发消息），否则服务端判定离线** 你现在 30s 内没有任何 MQTT 消息交互，不会自动发心跳包，Unity broker 直接踢下线。 ✅ 解决办法：ESP32 定时主动发送心跳报文给 Unity 服务端，维持在线。

#### 修改方案

1. 新增心跳主题，Unity 侧订阅这个心跳主题
2. ESP32 每 20 秒自动发一条心跳包，小于 30s 的 keepalive，避免超时断开
3. 保留原有 `cmd/device01`、`data/device01` 逻辑不变
4. 同时保留按键、启停逻辑

### Arduino

```C++
#include <Arduino.h>
#include <Ethernet_Generic.h>
#include <PubSubClient.h>
#include <SPI.h>

#define USING_SPI2     true
#define SPI2_MOSI      23
#define SPI2_MISO      19
#define SPI2_SCK       18
#define SPI2_SS        5

#define W5500_RST    4
#define KEY_PIN      27

byte mac[] = {0xDE, 0xAD, 0xBE, 0xEF, 0xFE, 0xED};
IPAddress espStaticIp(192, 168, 0, 100);
IPAddress espGateway(192, 168, 1, 1);
IPAddress espSubnet(255, 255, 254, 0);
IPAddress espDns(192, 168, 1, 1);

const char* mqttServer = "192.168.0.71";
const uint16_t mqttPort = 1883;
const char* clientId = "ESP32_W5500_001";
const char* subTopic = "cmd/device01";
const char* pubTopic = "data/device01";
const char* heartbeatTopic = "heartbeat/device01";
const uint16_t MQTT_KEEPALIVE_SEC = 30;

EthernetClient ethClient;
PubSubClient mqttClient(ethClient);

bool enableUpload = false;
unsigned long lastHeartbeatTime = 0;
const unsigned long heartbeatInterval = 20000;
uint8_t reconnectFailCount = 0;

void mqttCallback(char* topic, byte* payload, unsigned int length)
{
  String recvMsg = "";
  for (int i = 0; i < length; i++)
  {
    recvMsg += (char)payload[i];
  }
  Serial.print("收到服务端指令：");
  Serial.println(recvMsg);

  if (recvMsg == "start")
  {
    enableUpload = true;
    Serial.println("✅ 上传已开启，按键可以上报数据");
  }
  else if (recvMsg == "stop")
  {
    enableUpload = false;
    Serial.println("❌ 上传已关闭，按键禁止上报");
  }
}

bool mqttFullReconnect()
{
  Serial.println("!!!执行完整重连：硬件复位W5500");

  // =====加长复位时序，解决W5500内部锁死=====
  pinMode(W5500_RST, OUTPUT);
  digitalWrite(W5500_RST, LOW);
  delay(200);   //拉低保持200ms
  digitalWrite(W5500_RST, HIGH);
  delay(500);   //释放后等待500ms，等待PHY完全初始化

  Ethernet.init(SPI2_SS);
  Ethernet.begin(mac, espStaticIp, espDns, espGateway, espSubnet);
  delay(400);

  Serial.print("W5500 IP after reset:");
  Serial.println(Ethernet.localIP());

  if (mqttClient.connect(clientId))
  {
    mqttClient.subscribe(subTopic);
    Serial.println("✅ MQTT重连成功");
    reconnectFailCount = 0;
    return true;
  }
  else
  {
    Serial.print("❌ MQTT重连失败 rc=");
    Serial.println(mqttClient.state());
    reconnectFailCount++;
    return false;
  }
}

void setup()
{
  Serial.begin(115200);
  delay(500);
  Serial.println("===== ESP32‑W5500 加固保活 =====");

  pinMode(W5500_RST, OUTPUT);
  digitalWrite(W5500_RST, LOW);
  delay(200);
  digitalWrite(W5500_RST, HIGH);
  delay(500);

  Ethernet.init(SPI2_SS);
  Ethernet.begin(mac, espStaticIp, espDns, espGateway, espSubnet);
  delay(400);
  Serial.print("✅ W5500 IP：");
  Serial.println(Ethernet.localIP());

  mqttClient.setServer(mqttServer, mqttPort);
  mqttClient.setCallback(mqttCallback);
  mqttClient.setKeepAlive(MQTT_KEEPALIVE_SEC);

  pinMode(KEY_PIN, INPUT_PULLUP);
}

void loop()
{
  mqttClient.loop();

  if (!mqttClient.connected())
  {
    mqttFullReconnect();

    //连续失败，拉长等待时间，防止疯狂循环复位，加重W5500锁死
    if(reconnectFailCount > 3)
    {
      Serial.println("⚠️多次重连失败，长延时等待...");
      delay(3000);
    }else{
      delay(1200);
    }
  }

  if(millis() - lastHeartbeatTime > heartbeatInterval)
  {
    lastHeartbeatTime = millis();
    if(mqttClient.connected())
    {
      bool ok = mqttClient.publish(heartbeatTopic, "alive");
      if(ok)
      {
        Serial.println("💓发送心跳alive");
        reconnectFailCount = 0;
      }
      else
      {
        Serial.println("⚠️心跳发送失败，网络僵死");
      }
    }
  }

  static unsigned long lastKeyTime = 0;
  const unsigned long keyInterval = 200;
  if(millis() - lastKeyTime > keyInterval)
  {
    if(digitalRead(KEY_PIN) == LOW)
    {
      lastKeyTime = millis();
      Serial.println("👉检测按键按下");
      if (enableUpload)
      {
        mqttClient.publish(pubTopic, "key_trigger_ok");
        Serial.println("✅按键上报完成");
      }
      else
      {
        Serial.println("❌上传关闭，不上报");
      }
    }
  }
}

```

#### Unity 侧配套修改

1. 在 Unity 的 MQTT Broker 里，订阅 `heartbeat/device01`
2. 收到 `alive` 代表设备在线；长时间收不到心跳判定离线

#### 额外补充（外接电源断线的叠加优化）

1. 共地一定要做好，外接电源 ESP32 GND ↔ W5500 GND 短接
2. 如果你不想新增心跳主题，也可以**把 keepalive 调大**（比如 60s），但是不推荐，断网之后 Unity 要很久才识别离线
3. PubSubClient 本身**不会自动发送 PINGREQ 心跳包**，这是这个库的经典坑，必须自己定时发消息维持连接

> 上传代码后，串口会每 20s 打印 `💓 发送心跳 alive`，Unity 日志里就能持续收到心跳，不会再因为心跳超时踢掉设备。



### Unity Server

```C#
using System;
using System.Collections.Concurrent;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Net;
using System.Net.NetworkInformation;
using System.Net.Sockets;
using System.Text;
using System.Threading;

/// <summary>
/// 极简 MQTT 3.1.1 Broker（服务端），仅依赖 System.Net.Sockets，无第三方库。
/// 面向本项目的设备端 main.ino（ESP32 + W5500，Arduino Ethernet2 + PubSubClient）：
///   - CONNECT/CONNACK（协议等级 3/4）、SUBSCRIBE/SUBACK、UNSUBSCRIBE/UNSUBACK
///   - PUBLISH QoS0（设备上报 data/device01）与 QoS1（回 PUBACK）
///   - PINGREQ/PINGRESP、DISCONNECT
///   - KeepAlive 心跳检测：1.5 倍 keepalive 内无任何报文即判定离线并断开
///   - 订阅过滤器支持 '+' 单层与 '#' 多层通配符
/// 后台线程产生的事件写入 Events 队列，由 Unity 主线程在 Update 中消费。
/// </summary>
public class MqttBrokerServer : IDisposable
{
    public enum EventType
    {
        Started,
        Stopped,
        Error,
        ClientConnected,
        ClientDisconnected,
        MessageReceived,
        Subscribed,
    }

    public struct BrokerEvent
    {
        public EventType Type;
        public string ClientId;   // MessageReceived 时为消息来源，null = 服务器自身发布
        public string Topic;
        public string Payload;
        public string Detail;     // Error 的附加信息
    }

    private TcpListener _listener;
    private Thread _acceptThread;
    private Thread _monitorThread;
    private readonly List<MqttClientConnection> _clients = new List<MqttClientConnection>();
    private readonly object _gate = new object();
    private volatile bool _running;

    /// <summary>后台线程产生的事件队列，主线程在 Update 中 TryDequeue 消费。</summary>
    public readonly ConcurrentQueue<BrokerEvent> Events = new ConcurrentQueue<BrokerEvent>();

    public int Port { get; private set; }
    public bool IsRunning { get { return _running; } }

    public bool Start(int port)
    {
        if (_running) return true;
        try
        {
            Port = port;
            _listener = new TcpListener(IPAddress.Any, port);
            _listener.Server.SetSocketOption(SocketOptionLevel.Socket, SocketOptionName.ReuseAddress, true);
            _listener.Start();
            _running = true;
            _acceptThread = new Thread(AcceptLoop) { IsBackground = true, Name = "MqttBrokerAccept" };
            _acceptThread.Start();
            _monitorThread = new Thread(MonitorLoop) { IsBackground = true, Name = "MqttBrokerMonitor" };
            _monitorThread.Start();
            Push(EventType.Started, detail: "Broker 已启动，监听 0.0.0.0:" + port);
            return true;
        }
        catch (Exception ex)
        {
            _running = false;
            Push(EventType.Error, detail: "Broker 启动失败（端口 " + port + "）: " + ex.Message);
            return false;
        }
    }

    public void Stop()
    {
        if (!_running) return;
        _running = false;
        try { if (_listener != null) _listener.Stop(); } catch { }
        MqttClientConnection[] snapshot;
        lock (_gate) { snapshot = _clients.ToArray(); _clients.Clear(); }
        foreach (var client in snapshot) client.Close();
        _listener = null;
        Push(EventType.Stopped);
    }

    public void Dispose() { Stop(); }

    private void AcceptLoop()
    {
        while (_running)
        {
            TcpClient tcp;
            try { tcp = _listener.AcceptTcpClient(); }
            catch (SocketException) { Thread.Sleep(50); continue; }
            catch { break; } // 监听器已关闭（ObjectDisposedException 等）
            var conn = new MqttClientConnection(this, tcp);
            lock (_gate) _clients.Add(conn);
            conn.Start();
        }
    }

    /// <summary>心跳看护线程：设备超过 1.5 倍 keepalive（加 2s 判定余量）未发来任何报文，
    /// 判定为离线并断开；keepalive=0 的连接不做心跳约束。
    /// 任何报文（PINGREQ、上报、订阅等）都视为一次心跳。</summary>
    private void MonitorLoop()
    {
        while (_running)
        {
            Thread.Sleep(2000);
            if (!_running) break;
            MqttClientConnection[] snapshot;
            lock (_gate) snapshot = _clients.ToArray();
            foreach (var client in snapshot)
            {
                if (!client.IsSessionEstablished) continue;
                int keepAlive = client.KeepAliveSeconds;
                if (keepAlive <= 0) continue;
                int idleMs = client.IdleMilliseconds;
                if (idleMs > keepAlive * 1500 + 2000)
                    client.Close("心跳超时：keepalive " + keepAlive + "s，已 " + (idleMs / 1000) +
                                 "s 未收到任何报文，判定离线");
            }
        }
    }

    /// <summary>服务器主动发布消息（如 start/stop 命令）。返回转发到的订阅者数量，未运行返回 -1。</summary>
    public int Publish(string topic, string payload)
    {
        if (!_running || string.IsNullOrEmpty(topic)) return -1;
        return RouteMessage(null, topic, Encoding.UTF8.GetBytes(payload ?? string.Empty));
    }

    internal int RouteMessage(MqttClientConnection source, string topic, byte[] payload)
    {
        Push(EventType.MessageReceived,
             source != null ? source.ClientId : null,
             topic,
             Encoding.UTF8.GetString(payload));

        MqttClientConnection[] snapshot;
        lock (_gate) snapshot = _clients.ToArray();   // 仅在拷贝期间持锁，发送在锁外进行
        int delivered = 0;
        foreach (var client in snapshot)
        {
            if (ReferenceEquals(client, source)) continue;   // 不回发给发布方
            if (!client.IsSessionEstablished) continue;
            if (!client.IsMatch(topic)) continue;
            if (client.SendPublish(topic, payload)) delivered++;
        }
        return delivered;
    }

    public int ConnectedCount
    {
        get { lock (_gate) return _clients.Count(c => c.IsSessionEstablished); }
    }

    public string[] GetConnectedClientIds()
    {
        lock (_gate)
        {
            return _clients.Where(c => c.IsSessionEstablished && !string.IsNullOrEmpty(c.ClientId))
                            .Select(c => c.ClientId)
                            .ToArray();
        }
    }

    /// <summary>设备在线状态快照（供 UI 显示心跳信息）。</summary>
    public struct ClientStatus
    {
        public string ClientId;
        public int KeepAliveSeconds; // CONNECT 声明的 keepalive，0 = 未启用
        public int IdleSeconds;      // 距最近一次收到该设备报文已过的秒数
    }

    public ClientStatus[] GetClientStatuses()
    {
        lock (_gate)
        {
            return _clients
                .Where(c => c.IsSessionEstablished && !string.IsNullOrEmpty(c.ClientId))
                .Select(c => new ClientStatus
                {
                    ClientId = c.ClientId,
                    KeepAliveSeconds = c.KeepAliveSeconds,
                    IdleSeconds = c.IdleMilliseconds / 1000,
                })
                .ToArray();
        }
    }

    internal void OnSessionEstablished(MqttClientConnection conn)
    {
        Push(EventType.ClientConnected, conn.ClientId);
    }

    internal void OnSubscribed(MqttClientConnection conn, string filter)
    {
        Push(EventType.Subscribed, conn.ClientId, filter);
    }

    internal void OnClosed(MqttClientConnection conn, string disconnectedClientId, string reason = null)
    {
        lock (_gate) _clients.Remove(conn);
        if (disconnectedClientId != null)
            Push(EventType.ClientDisconnected, disconnectedClientId, detail: reason);
    }

    private void Push(EventType type, string clientId = null, string topic = null,
                      string payload = null, string detail = null)
    {
        Events.Enqueue(new BrokerEvent
        {
            Type = type,
            ClientId = clientId,
            Topic = topic,
            Payload = payload,
            Detail = detail,
        });
    }

    /// <summary>获取本机局域网 IPv4（设备端 main.ino 的 mqttServer 应填写该地址）。
    /// 优先返回真实网卡的私有网段地址；过滤 VPN/代理 TUN 虚拟网段（198.18.0.0/15、100.64.0.0/10）。</summary>
    public static string GetLocalIPv4()
    {
        var candidates = GetLocalIPv4Candidates();
        if (candidates.Count > 0)
        {
            int space = candidates[0].IndexOf(' ');
            return space > 0 ? candidates[0].Substring(0, space) : candidates[0];
        }
        // 兜底：UDP Connect 探测默认路由出口（不实际发包）
        try
        {
            using (var socket = new Socket(AddressFamily.InterNetwork, SocketType.Dgram, ProtocolType.Udp))
            {
                socket.Connect("8.8.8.8", 65530);
                var endPoint = socket.LocalEndPoint as IPEndPoint;
                if (endPoint != null) return endPoint.Address.ToString();
            }
        }
        catch { }
        return "127.0.0.1";
    }

    /// <summary>列出可用于局域网通信的 IPv4 候选（格式 "IP (网卡名)"）。
    /// 过滤回环、APIPA(169.254)、VPN-TUN 常用保留段；私有网段（192.168/10/172.16-31）优先排序。</summary>
    public static List<string> GetLocalIPv4Candidates()
    {
        var filtered = new List<KeyValuePair<int, string>>();
        var unfiltered = new List<string>();
        try
        {
            foreach (var nic in NetworkInterface.GetAllNetworkInterfaces())
            {
                if (nic.OperationalStatus != OperationalStatus.Up ||
                    nic.NetworkInterfaceType == NetworkInterfaceType.Loopback) continue;
                bool hasGateway = false;
                try
                {
                    var gateways = nic.GetIPProperties().GatewayAddresses;
                    hasGateway = gateways != null && gateways.Count > 0;
                }
                catch { }
                foreach (var addr in nic.GetIPProperties().UnicastAddresses)
                {
                    if (addr.Address.AddressFamily != AddressFamily.InterNetwork) continue;
                    if (IPAddress.IsLoopback(addr.Address)) continue;
                    string entry = addr.Address + " (" + nic.Name + ")";
                    if (IsVirtualLanIp(addr.Address))
                        unfiltered.Add(entry); // 只有虚拟网卡时至少还能给出参考
                    else
                        // 有默认网关的真实网卡优先（VirtualBox/WSL 虚拟交换机没有网关）
                        filtered.Add(new KeyValuePair<int, string>(
                            (hasGateway ? 0 : 10) + LanPreferenceScore(addr.Address), entry));
                }
            }
        }
        catch { }
        var result = new List<string>();
        foreach (var item in filtered.OrderBy(x => x.Key)) result.Add(item.Value);
        if (result.Count == 0) result.AddRange(unfiltered);
        return result;
    }

    // 169.254 APIPA 链路本地；198.18.0.0/15 基准测试段（Clash 等代理 TUN 常用）；100.64.0.0/10 CGNAT（Tailscale 等）
    private static bool IsVirtualLanIp(IPAddress ip)
    {
        var b = ip.GetAddressBytes();
        if (b[0] == 169 && b[1] == 254) return true;
        if (b[0] == 198 && (b[1] & 0xFE) == 18) return true;
        if (b[0] == 100 && (b[1] & 0xC0) == 64) return true;
        return false;
    }

    // 排序偏好：192.168.x（本项目设备网段）最优先，其次 10.x、172.16-31，公网地址靠后
    private static int LanPreferenceScore(IPAddress ip)
    {
        var b = ip.GetAddressBytes();
        if (b[0] == 192 && b[1] == 168) return 0;
        if (b[0] == 10) return 1;
        if (b[0] == 172 && b[1] >= 16 && b[1] <= 31) return 2;
        return 3;
    }

    /// <summary>MQTT 主题过滤器匹配：'+' 匹配单层，'#' 匹配剩余全部层级。</summary>
    public static bool TopicFilterMatches(string filter, string topic)
    {
        if (string.IsNullOrEmpty(filter) || string.IsNullOrEmpty(topic)) return false;
        var f = filter.Split('/');
        var t = topic.Split('/');
        for (int i = 0; i < f.Length; i++)
        {
            if (f[i] == "#") return true;
            if (i >= t.Length) return false;
            if (f[i] == "+") continue;
            if (!string.Equals(f[i], t[i], StringComparison.Ordinal)) return false;
        }
        return f.Length == t.Length;
    }
}

/// <summary>单个设备连接：独立读取线程解析 MQTT 报文，线程安全写入。</summary>
internal sealed class MqttClientConnection
{
    private static readonly byte[] EmptyBytes = new byte[0];

    private readonly MqttBrokerServer _broker;
    private readonly TcpClient _tcp;
    private readonly NetworkStream _stream;
    private readonly List<string> _subscriptions = new List<string>();
    private readonly object _writeGate = new object();
    private readonly object _closeGate = new object();
    private bool _closed;
    private int _keepAliveSeconds;                                // CONNECT 声明的 keepalive（秒），0 = 未启用
    private volatile int _lastReceiveMs = Environment.TickCount;  // 最近一次收到报文的时间戳

    public string ClientId { get; private set; }
    public bool IsSessionEstablished { get; private set; }

    /// <summary>客户端 CONNECT 报文声明的 keepalive（秒），0 表示未启用心跳。</summary>
    public int KeepAliveSeconds { get { return _keepAliveSeconds; } }

    /// <summary>距最近一次收到报文已过的毫秒数（TickCount 差值，回绕安全）。</summary>
    public int IdleMilliseconds
    {
        get { return unchecked(Environment.TickCount - _lastReceiveMs); }
    }

    public MqttClientConnection(MqttBrokerServer broker, TcpClient tcp)
    {
        _broker = broker;
        _tcp = tcp;
        _stream = tcp.GetStream();
        try { _tcp.NoDelay = true; } catch { }
    }

    public void Start()
    {
        var thread = new Thread(ReadLoop) { IsBackground = true, Name = "MqttClientConn" };
        thread.Start();
    }

    public bool IsMatch(string topic)
    {
        lock (_subscriptions)
        {
            for (int i = 0; i < _subscriptions.Count; i++)
            {
                if (MqttBrokerServer.TopicFilterMatches(_subscriptions[i], topic)) return true;
            }
        }
        return false;
    }

    private void ReadLoop()
    {
        try
        {
            while (!_closed)
            {
                int header = _stream.ReadByte();
                if (header < 0) break;                       // 对端关闭
                _lastReceiveMs = Environment.TickCount;      // 任意报文都视为一次心跳
                int remainingLength;
                if (!TryReadRemainingLength(_stream, out remainingLength)) break;
                byte[] body = remainingLength > 0 ? ReadExactly(_stream, remainingLength) : EmptyBytes;
                HandlePacket(header, body);
            }
        }
        catch { }
        finally { Close(); }
    }

    private void HandlePacket(int header, byte[] body)
    {
        int packetType = (header >> 4) & 0x0F;
        switch (packetType)
        {
            case 1: HandleConnect(body); break;                  // CONNECT
            case 3: HandlePublish(header, body); break;          // PUBLISH
            case 8: HandleSubscribe(body); break;                // SUBSCRIBE
            case 10: HandleUnsubscribe(body); break;             // UNSUBSCRIBE
            case 12: SendRaw(new byte[] { 0xD0, 0x00 }); break;  // PINGREQ -> PINGRESP
            case 14: Close(); break;                            // DISCONNECT
            default: break;                                     // 其余类型忽略
        }
    }

    private void HandleConnect(byte[] body)
    {
        if (IsSessionEstablished) { Close(); return; } // 协议规定同一连接只允许一次 CONNECT

        int pos = 0;
        string protocolName = ReadMqttString(body, ref pos);   // "MQTT" 或 "MQIsdp"（未再校验）
        int protocolLevel = ReadByte(body, ref pos);           // 4 = MQTT 3.1.1, 3 = MQTT 3.1
        int connectFlags = ReadByte(body, ref pos);            // 仅解析推进，不依赖 flags
        int keepAlive = ReadUShort(body, ref pos);
        _keepAliveSeconds = keepAlive;                          // 心跳周期，供离线判定与在线状态显示
        string clientId = ReadMqttString(body, ref pos);

        if (protocolLevel != 3 && protocolLevel != 4)
        {
            SendRaw(new byte[] { 0x20, 0x02, 0x00, 0x01 });    // CONNACK: 不支持的协议版本
            Close();
            return;
        }
        if (string.IsNullOrEmpty(clientId))
            clientId = "anonymous-" + Environment.TickCount;

        ClientId = clientId;
        IsSessionEstablished = true;
        SendRaw(new byte[] { 0x20, 0x02, 0x00, 0x00 });         // CONNACK: 接受，无持久会话
        _broker.OnSessionEstablished(this);
    }

    private void HandlePublish(int header, byte[] body)
    {
        int pos = 0;
        string topic = ReadMqttString(body, ref pos);
        int qos = (header >> 1) & 0x03;
        if (qos > 0)
        {
            int packetId = ReadUShort(body, ref pos);
            if (qos == 1)
                SendRaw(new byte[] { 0x40, 0x02, (byte)(packetId >> 8), (byte)(packetId & 0xFF) }); // PUBACK
        }
        if (qos >= 2) return; // QoS2 超出本 Broker 能力，丢弃

        byte[] payload = new byte[body.Length - pos];
        Array.Copy(body, pos, payload, 0, payload.Length);
        if (!string.IsNullOrEmpty(topic))
            _broker.RouteMessage(this, topic, payload);
    }

    private void HandleSubscribe(byte[] body)
    {
        int pos = 0;
        int packetId = ReadUShort(body, ref pos);
        var granted = new List<byte>();
        while (pos < body.Length)
        {
            string filter = ReadMqttString(body, ref pos);
            int qos = ReadByte(body, ref pos); // 订阅 QoS 字节必须消费掉
            if (!string.IsNullOrEmpty(filter))
            {
                lock (_subscriptions)
                {
                    if (!_subscriptions.Contains(filter)) _subscriptions.Add(filter);
                }
                _broker.OnSubscribed(this, filter);
            }
            granted.Add(0); // 统一按 QoS0 授权
        }
        var ack = new byte[4 + granted.Count];
        ack[0] = 0x90;
        ack[1] = (byte)(2 + granted.Count); // SUBACK（单包订阅数 <126 时剩余长度单字节足够）
        ack[2] = (byte)(packetId >> 8);
        ack[3] = (byte)(packetId & 0xFF);
        for (int i = 0; i < granted.Count; i++) ack[4 + i] = granted[i];
        SendRaw(ack);
    }

    private void HandleUnsubscribe(byte[] body)
    {
        int pos = 0;
        int packetId = ReadUShort(body, ref pos);
        while (pos < body.Length)
        {
            string filter = ReadMqttString(body, ref pos);
            lock (_subscriptions) _subscriptions.Remove(filter);
        }
        SendRaw(new byte[] { 0xB0, 0x02, (byte)(packetId >> 8), (byte)(packetId & 0xFF) });
    }

    /// <summary>向该客户端转发一条 QoS0 PUBLISH。</summary>
    public bool SendPublish(string topic, byte[] payload)
    {
        if (_closed) return false;
        byte[] topicBytes = Encoding.UTF8.GetBytes(topic);
        var buffer = new MemoryStream();
        buffer.WriteByte(0x30); // PUBLISH, QoS0
        WriteRemainingLength(buffer, 2 + topicBytes.Length + payload.Length);
        buffer.WriteByte((byte)(topicBytes.Length >> 8));
        buffer.WriteByte((byte)(topicBytes.Length & 0xFF));
        buffer.Write(topicBytes, 0, topicBytes.Length);
        buffer.Write(payload, 0, payload.Length);
        return SendRaw(buffer.ToArray());
    }

    private bool SendRaw(byte[] data)
    {
        if (_closed) return false;
        try
        {
            lock (_writeGate) _stream.Write(data, 0, data.Length);
            return true;
        }
        catch { Close(); return false; }
    }

    public void Close(string reason = null)
    {
        string disconnectedId = null;
        lock (_closeGate)
        {
            if (_closed) return;
            _closed = true;
            if (IsSessionEstablished && !string.IsNullOrEmpty(ClientId))
                disconnectedId = ClientId;
            IsSessionEstablished = false;
        }
        try { _stream.Close(); } catch { }
        try { _tcp.Close(); } catch { }
        _broker.OnClosed(this, disconnectedId, reason);
    }

    // ---------- 报文解析辅助 ----------

    private static bool TryReadRemainingLength(NetworkStream stream, out int length)
    {
        length = 0;
        int multiplier = 1;
        for (int i = 0; i < 4; i++) // 剩余长度最多 4 字节（MQTT 变长整数）
        {
            int value = stream.ReadByte();
            if (value < 0) return false;
            length += (value & 0x7F) * multiplier;
            if ((value & 0x80) == 0) return true;
            multiplier *= 128;
        }
        return false;
    }

    private static byte[] ReadExactly(NetworkStream stream, int count)
    {
        var buffer = new byte[count];
        int offset = 0;
        while (offset < count)
        {
            int read = stream.Read(buffer, offset, count - offset);
            if (read <= 0) throw new EndOfStreamException();
            offset += read;
        }
        return buffer;
    }

    private static int ReadByte(byte[] buffer, ref int pos)
    {
        if (pos >= buffer.Length) throw new EndOfStreamException();
        return buffer[pos++];
    }

    private static int ReadUShort(byte[] buffer, ref int pos)
    {
        int high = ReadByte(buffer, ref pos);
        int low = ReadByte(buffer, ref pos);
        return (high << 8) | low;
    }

    private static string ReadMqttString(byte[] buffer, ref int pos)
    {
        int length = ReadUShort(buffer, ref pos);
        if (pos + length > buffer.Length) throw new EndOfStreamException();
        string value = Encoding.UTF8.GetString(buffer, pos, length);
        pos += length;
        return value;
    }

    private static void WriteRemainingLength(MemoryStream stream, int length)
    {
        do
        {
            int digit = length % 128;
            length /= 128;
            if (length > 0) digit |= 0x80;
            stream.WriteByte((byte)digit);
        } while (length > 0);
    }
}

```

### Unity Server UI

```C#
using System;
using System.Collections.Generic;
using System.Text;
using UnityEngine;
using UnityEngine.UI;

/// <summary>
/// MQTT Broker 服务端控制台（对应设备端 main.ino：ESP32 + W5500，Arduino Ethernet2 + PubSubClient）：
/// - 启动内置 MQTT Broker（默认 1883 端口，实现见 MqttBrokerServer）
/// - 向命令主题（默认 cmd/device01，与设备端 main.ino 的订阅主题一致）发送 start（允许设备上报按键）/ stop（禁止上报）/ 自定义命令
/// - 显示设备连接状态、订阅事件、heartbeat/device01 心跳与 data/device01 按键上报日志
/// </summary>
public class MqttBrokerUI : MonoBehaviour
{
    [Header("Broker 设置")]
    public int port = 1883;
    public string cmdTopic = "cmd/device01";
    public string dataTopic = "data/device01";
    public string heartbeatTopic = "heartbeat/device01";   // 设备端每 20s 上报一次的心跳主题
    public int maxLogLines = 300;

    [Header("UI 引用")]
    public Text statusText;
    public Text logText;
    public ScrollRect logScroll;
    public Button startButton;
    public Button stopButton;
    public Button sendCustomButton;
    public Button clearButton;
    public InputField customInput;

    public MqttBrokerServer Broker { get; private set; }

    private readonly List<string> _logLines = new List<string>();
    private string _localIp;
    private bool _statusDirty = true;
    private float _nextStatusRefresh;

    void Awake()
    {
        Broker = new MqttBrokerServer();
        _localIp = MqttBrokerServer.GetLocalIPv4();
        EnsureChineseFont();
    }

    void Start()
    {
        Broker.Start(port);

        if (startButton != null) startButton.onClick.AddListener(SendStart);
        if (stopButton != null) stopButton.onClick.AddListener(SendStop);
        if (sendCustomButton != null) sendCustomButton.onClick.AddListener(OnSendCustomClicked);
        if (clearButton != null) clearButton.onClick.AddListener(ClearLog);
    }

    void Update()
    {
        bool hadEvents = false;
        MqttBrokerServer.BrokerEvent ev;
        while (Broker.Events.TryDequeue(out ev))
        {
            hadEvents = true;
            HandleEvent(ev);
        }
        if (hadEvents) _statusDirty = true;

        // 有设备在线时每秒刷新一次状态行，让心跳秒数实时走动
        if (Broker != null && Broker.IsRunning && Broker.ConnectedCount > 0 &&
            Time.unscaledTime >= _nextStatusRefresh)
        {
            _nextStatusRefresh = Time.unscaledTime + 1f;
            _statusDirty = true;
        }

        if (_statusDirty)
        {
            RefreshStatus();
            _statusDirty = false;
        }
    }

    void OnDestroy()
    {
        if (Broker != null) Broker.Stop();
    }

    // ---------- 对外操作 ----------

    /// <summary>向命令主题发布消息（start / stop / 自定义），返回转发到的设备数，失败返回 -1。</summary>
    public int PublishCommand(string payload)
    {
        if (Broker == null || !Broker.IsRunning)
        {
            AppendLog("[错误] Broker 未运行，无法发送: " + payload);
            return -1;
        }
        int delivered = Broker.Publish(cmdTopic, payload);
        AppendLog("[发送] " + cmdTopic + " → \"" + payload + "\"（转发给 " + delivered + " 台设备）");
        if (delivered == 0)
            AppendLog("[提示] 当前没有设备订阅 " + cmdTopic + "，设备连接并订阅后才能收到");
        return delivered;
    }

    public void SendStart() { PublishCommand("start"); }

    public void SendStop() { PublishCommand("stop"); }

    private void OnSendCustomClicked()
    {
        if (customInput == null) return;
        string text = customInput.text == null ? string.Empty : customInput.text.Trim();
        if (text.Length == 0)
        {
            AppendLog("[提示] 请先输入要发送的命令");
            return;
        }
        PublishCommand(text);
        customInput.text = string.Empty;
    }

    public void ClearLog()
    {
        _logLines.Clear();
        if (logText != null) logText.text = string.Empty;
    }

    // ---------- 内部 ----------

    private void HandleEvent(MqttBrokerServer.BrokerEvent ev)
    {
        switch (ev.Type)
        {
            case MqttBrokerServer.EventType.Started:
                AppendLog("Broker 已启动，监听 0.0.0.0:" + port);
                var ipCandidates = MqttBrokerServer.GetLocalIPv4Candidates();
                if (ipCandidates.Count > 0)
                    AppendLog("本机 IP 候选: " + string.Join("; ", ipCandidates.ToArray()));
                AppendLog("设备端 main.ino 请设置: IPAddress mqttServer 为本机 IP " + _localIp);
                break;
            case MqttBrokerServer.EventType.Stopped:
                AppendLog("Broker 已停止");
                break;
            case MqttBrokerServer.EventType.Error:
                AppendLog("[错误] " + ev.Detail);
                break;
            case MqttBrokerServer.EventType.ClientConnected:
                AppendLog("[连接] 设备已连接: " + ev.ClientId);
                break;
            case MqttBrokerServer.EventType.Subscribed:
                AppendLog("[订阅] " + ev.ClientId + " → " + ev.Topic);
                break;
            case MqttBrokerServer.EventType.ClientDisconnected:
                AppendLog(string.IsNullOrEmpty(ev.Detail)
                    ? "[断开] 设备已断开: " + ev.ClientId
                    : "[断开] 设备已断开: " + ev.ClientId + "（" + ev.Detail + "）");
                break;
            case MqttBrokerServer.EventType.MessageReceived:
                if (string.IsNullOrEmpty(ev.ClientId)) break; // 服务器自身发布的只记 "[发送]"，不在此重复记录
                if (ev.Topic == heartbeatTopic)
                    AppendLog("[心跳] " + ev.ClientId + ": " + ev.Payload);
                else
                    AppendLog("[收到] " + ev.Topic + " ← " + ev.ClientId + ": " + ev.Payload);
                break;
        }
    }

    private void RefreshStatus()
    {
        if (statusText == null) return;
        if (Broker != null && Broker.IsRunning)
        {
            var statuses = Broker.GetClientStatuses();
            var sb = new StringBuilder();
            sb.Append("● 运行中  ").Append(_localIp).Append(':').Append(port);
            sb.Append("  |  已连接设备: ");
            if (statuses.Length == 0)
                sb.Append("无");
            else
            {
                for (int i = 0; i < statuses.Length; i++)
                {
                    if (i > 0) sb.Append(", ");
                    sb.Append(statuses[i].ClientId).Append("（在线");
                    if (statuses[i].KeepAliveSeconds > 0)
                        sb.Append("，心跳 ").Append(statuses[i].IdleSeconds).Append("s 前");
                    else
                        sb.Append("，未启用心跳");
                    sb.Append('）');
                }
            }
            statusText.text = sb.ToString();
        }
        else
        {
            statusText.text = "○ 未运行";
        }
    }

    public void AppendLog(string message)
    {
        _logLines.Add(DateTime.Now.ToString("[HH:mm:ss] ") + message);
        int overflow = _logLines.Count - maxLogLines;
        if (overflow > 0) _logLines.RemoveRange(0, overflow);
        if (logText != null) logText.text = string.Join("\n", _logLines.ToArray());
        if (logScroll != null) logScroll.verticalNormalizedPosition = 0f; // 滚到底部
    }

    /// <summary>内置 LegacyRuntime/Arial 字体缺少中文字形时回退系统字体（微软雅黑），避免中文显示为方块。</summary>
    private void EnsureChineseFont()
    {
        try
        {
            var texts = FindObjectsOfType<Text>();
            if (texts == null || texts.Length == 0) return;
            var probe = texts[0];
            if (probe == null || probe.font == null || probe.font.HasCharacter('发')) return;
            var sysFont = Font.CreateDynamicFontFromOSFont(new[] { "Microsoft YaHei", "微软雅黑", "SimHei" }, 16);
            foreach (var t in texts)
            {
                if (t != null) t.font = sysFont;
            }
        }
        catch { /* 保持默认字体 */ }
    }
}
```