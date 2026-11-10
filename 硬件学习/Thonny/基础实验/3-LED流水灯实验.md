```python
'''
实验名称：LED流水灯实验
接线说明：LED模块-->ESP32 IO
         (D1-D8)-->(15,2,0,4,16,17,5,18)
         
实验现象：程序下载成功后，LED模块D1-D8指示灯依次点亮后依次熄灭
注意事项：

'''

#导入Pin模块
from machine import Pin
import time

led_pin=[15,2,0,4,16,17,5,18]  #定义LED控制引脚
leds=[]  #定义leds列表，保存LED管脚配置对象
for i in range(8):  #循环8次，0-7
    leds.append(Pin(led_pin[i],Pin.OUT))  #给leds列表添加对象
# leds=[Pin(led_pin[i],Pin.OUT) for i in range(0,8)]

    
#程序入口
if __name__=="__main__":
    #LED全熄灭
    for n in range(8):
        leds[n].value(0)
        
    while True:
        #LED逐个点亮
        for n in range(8):
            leds[n].value(1)
            time.sleep(0.05)
        #LED逐个熄灭
        for n in range(8):
            leds[n].value(0)
            time.sleep(0.05)



```

