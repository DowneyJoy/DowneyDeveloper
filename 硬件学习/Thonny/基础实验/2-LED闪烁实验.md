```python
'''
实验名称：LED闪烁实验
接线说明：LED模块-->ESP32 IO
         D1-->15
         
实验现象：程序下载成功后，D1指示灯闪烁
注意事项：

'''


from machine import Pin  #导入Pin模块
import time  #导入time模块

led1=Pin(15,Pin.OUT)#构建led1对象，GPIO15输出

#循环
while True:
    led1.value(1)#使IO15输出高电平，点亮LED
    time.sleep(0.5)#延时0.5秒
    led1.value(0)#使IO15输出低电平，熄灭LED
    time.sleep(0.5)


	
```

