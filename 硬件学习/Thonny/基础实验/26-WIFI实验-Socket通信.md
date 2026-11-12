WIFI实验-Socket通信

```python
'''
实验名称：WIFI实验-Socket通信
接线说明：LED模块-->ESP32 IO
         (D1)-->(15)
         
实验现象：程序下载成功后，软件shell控制台输出当前IP、子网掩码、网关的地址信息，在网络调试助手上
         连接成功后，可数据收发。
         
注意事项：ESP32 WIFI作为客户端连接路由器热点，然后电脑也连接路由器，此时可在电脑端使用网络调试助手，
         设置：协议类型：TCP Server，本机主机地址：电脑端IP地址，本机主机端口：10000

'''

#导入Pin模块
from machine import Pin
import time
import network
import usocket

#定义LED控制对象
led1=Pin(15,Pin.OUT)

#路由器WIFI账号和密码
ssid="wuliu"
password="88888888"

#服务器地址和端口
dest_ip="192.168.0.38"
dest_port=10000

#WIFI连接
def wifi_connect():
    wlan=network.WLAN(network.STA_IF)  #STA模式
    wlan.active(True)  #激活
    start_time=time.time()  #记录时间做超时判断
    
    if not wlan.isconnected():
        print("conneting to network...")
        wlan.connect(ssid,password)  #输入WIFI账号和密码
        
        while not wlan.isconnected():
            led1.value(1)
            time.sleep_ms(300)
            led1.value(0)
            time.sleep_ms(300)
            
            #超时判断,15 秒没连接成功判定为超时
            if time.time()-start_time>15:
                print("WIFI Connect Timeout!")
                break
        return False
    
    else:
        led1.value(0)
        print("network information:", wlan.ifconfig())
        return True


#程序入口
if __name__=="__main__":
    
    if wifi_connect():
        socket=usocket.socket()  #创建socket连接
        addr=(dest_ip,dest_port)  #服务器IP地址和端口
        socket.connect(addr)
        socket.send("Hello")
        
        while True:
            text=socket.recv(128)  #单次最多接收128字节
            if text==None:
                pass
            else:
                print(text)
                socket.send("I get:"+text.decode("utf-8"))
            time.sleep_ms(300)
        


```

