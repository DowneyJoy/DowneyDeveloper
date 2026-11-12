



```python
'''
实验名称：WIFI控制led灯实验
接线说明：LED模块-->ESP32 IO
         (D1)-->(22)
         
实验现象：程序下载成功后，手机登入网页IP地址，点击ON控制led灯点亮，OFF控制led灯熄灭。
注意事项：更改ssid，passwor
'''
try:
    import usocket as socket
except:
    import socket
    
from machine import Pin
import network
import time  # 新增time模块，用于WiFi超时与闪烁

import esp
esp.osdebug(None)

import gc
gc.collect()

# 路由器WIFI账号和密码
ssid = "wuliu"                     #修改为热点名字
password = "88888888"              #修改为热点密码

# 【替换后的WiFi连接函数】
def wifi_connect():
    wlan = network.WLAN(network.STA_IF)  # STA模式
    wlan.active(True)  # 激活
    start_time = time.time()  # 记录时间做超时判断
    
    if not wlan.isconnected():
        print("conneting to network...")
        wlan.connect(ssid, password)  # 输入 WIFI 账号密码
        
        while not wlan.isconnected():
            # 这里复用原有22脚LED做连接闪烁提示
            led.value(1)
            time.sleep_ms(300)
            led.value(0)
            time.sleep_ms(300)
            
            # 超时判断,15 秒没连接成功判定为超时
            if time.time() - start_time > 15:
                print("WIFI Connect Timeout!")
                break
    
    else:
        led.value(0)
        print("network information:", wlan.ifconfig())

# 原有LED定义（引脚22不变）
led = Pin(22, Pin.OUT)
led.value(0)  # 初始低电平（熄灭）

# 执行WiFi连接
wifi_connect()

# 以下原有网页服务代码 完全未修改
def web_page():
    if led.value()==1:
        gpio_state="ON"      # 高电平（点亮）
    else: 
        gpio_state="OFF"     # 低电平（熄灭）
        
    html = """<html><head> <title>ESp32 LED control</title> <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="icon" href="data:,"> <style>html{font-family: Helvetica; display: inline-block; margin: 0px auto; text-align:center;}
    h1{color: #0F3376; padding: 2vh;}p{font-size: 1.5rem;}.button{display: inline-block; background-color:#e7bd3b; border: none;
    border-radius: 4px; color: white; padding: 16px 40px; text-decoration: none; font-size: 30px; margin:2px;cursor: pointer;}
    .button2{background-color:#4286f4;)</style></head><body><h1>ESP32 LED control</h1>
    <p>GPIO state:<strong>""" + gpio_state +"""</strong></p><p><p><a href="/?led=on"><button class="button button">0N</button></a></p>
    <p><a href="/?led=off"><button class="button button2">0FF</button></p></body></html>"""
    return html

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.bind(('',80))
s.listen(5)
while True:
    conn, addr = s.accept()
    print('Got a connection from %s'% str(addr))
    request = conn.recv(1024)
    request = str(request)
    print('Content = %s'% request)
    led_on = request.find('/?led=on')
    led_off = request.find('/?led=off')
    if led_on == 6:
        print('LED ON')
        led.value(1)
    if led_off == 6:
        print('LED OFF')
        led.value(0)
    response = web_page()
    conn.send('HTTP/1.1 200 0K\n')
    conn.send('Content-Type: text/html\n')
    conn.send('Connection: close\n\n')
    conn.sendall(response)
    conn.close()


```

