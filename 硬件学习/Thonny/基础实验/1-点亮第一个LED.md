```python
'''
实验名称：点亮第一个LED
接线说明：LED模块-->ESP32 IO
         D1-->15
         
实验现象：程序下载成功后，D1指示灯点亮
注意事项：

'''

#导入Pin模块
from machine import Pin

led1=Pin(15,Pin.OUT)#构建led1对象，GPIO15输出
led1.value(1)#使IO15输出高电平，点亮LED



```

