

```c++
#include <Arduino.h>
#include <WiFi.h>
#include <PubSubClient.h>

// ==================== WiFi 配置 ====================
const char* wifiSsid     = "YGTH";      // 改成你的WiFi名称
const char* wifiPassword = "YGTHygth";  // 改成你的WiFi密码

// 如需静态IP，把 USE_STATIC_IP 改为 true 并按路由器网段修改
#define USE_STATIC_IP  false
IPAddress espStaticIp(192, 168, 0, 100);
IPAddress espGateway(192, 168, 1, 1);
IPAddress espSubnet(255, 255, 254, 0);
IPAddress espDns(192, 168, 1, 1);

// ==================== MQTT 配置 ====================
const char* mqttServer = "192.168.0.71";
const uint16_t mqttPort = 1883;
const char* clientId = "ESP32_WIFI_001";
const char* subTopic = "cmd/device01";
const char* pubTopic = "data/device01";
const char* heartbeatTopic = "heartbeat/device01";
const uint16_t MQTT_KEEPALIVE_SEC = 30;

#define KEY_PIN 27

WiFiClient wifiClient;
PubSubClient mqttClient(wifiClient);

bool enableUpload = false;
unsigned long lastHeartbeatTime = 0;
const unsigned long heartbeatInterval = 20000;
uint8_t reconnectFailCount = 0;

// ==================== WiFi 连接 ====================
bool wifiConnect()
{
  Serial.print("正在连接WiFi: ");
  Serial.println(wifiSsid);

  WiFi.mode(WIFI_STA);
  WiFi.setAutoReconnect(true);   // 让WiFi库自动重连
#if USE_STATIC_IP
  WiFi.config(espStaticIp, espGateway, espSubnet, espDns);
#endif
  WiFi.begin(wifiSsid, wifiPassword);

  unsigned long start = millis();
  while (WiFi.status() != WL_CONNECTED)
  {
    delay(300);
    Serial.print(".");
    // 10秒连不上就判定失败，返回false交给上层重试
    if (millis() - start > 10000)
    {
      Serial.println();
      return false;
    }
  }
  Serial.println();
  Serial.print("✅ WiFi连接成功，IP：");
  Serial.println(WiFi.localIP());
  Serial.print("信号强度 RSSI：");
  Serial.println(WiFi.RSSI());
  return true;
}

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

// ==================== MQTT 完整重连 ====================
bool mqttFullReconnect()
{
  // WiFi掉线时先恢复WiFi，相当于原来"复位W5500"的角色
  if (WiFi.status() != WL_CONNECTED)
  {
    Serial.println("!!!WiFi掉线，尝试重新连接...");
    WiFi.disconnect();
    delay(200);
    if (!wifiConnect())
    {
      reconnectFailCount++;
      return false;
    }
  }

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
  Serial.println("===== ESP32-WiFi 加固保活 =====");

  wifiConnect();

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

    // 连续失败，拉长等待时间，防止疯狂循环重连
    if (reconnectFailCount > 3)
    {
      Serial.println("⚠️多次重连失败，长延时等待...");
      delay(3000);
    }
    else
    {
      delay(1200);
    }
  }

  if (millis() - lastHeartbeatTime > heartbeatInterval)
  {
    lastHeartbeatTime = millis();
    if (mqttClient.connected())
    {
      bool ok = mqttClient.publish(heartbeatTopic, "alive");
      if (ok)
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
  if (millis() - lastKeyTime > keyInterval)
  {
    if (digitalRead(KEY_PIN) == LOW)
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

