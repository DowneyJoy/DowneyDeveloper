### ADC

- ADC（analog to digital converter）即模数转换器，它可以将模拟信号转换为数字信号。

- ADC功能在ESP32引脚32-39上可用。

### 实验目的

使用ADC采集电位器0-3.3v电压，并输出在软件shell控制台显示

### 构造函数

- adc=machine.ADC(Pin(id))
  - 构建ADC对象。id:ESP32的ADC在32-39引脚上。

### 使用方法

- adc.read() 获取ADC值。测量精度是12位，返回0-4095(表示0-1V)。
- adc.atten(attenuation) 配置衰减器。配置衰减器能增加电压测量范围，但是以精度为代价的。
  - ADC.ATTN_0DB:0dB 衰减，最大输入电压为1.00v-这是默认配置;
  - ADC.ATTN_2_5DB:2.5dB衰减，最大输入电压约为1.34V;
  - ADC.ATTN_6DB:6dB 衰减，最大输入电压约为2.00V;
  - ADC.ATTN_11DB:11dB衰减，最大输入电压约为3.3v。

```python
from machine import ADC

adc = ADC(Pin(32))   		  #在ADC引脚上创建ADC对象
adc.read()					 #读取测量值，0-4095 表示电压从0.0v-1.0v
adc.atten(ADC.ATTN_11DB)	   #设置11dB 衰减输入(测量电压大致从0.0v-3.6v)
adc.width(ADC.WIDTH_9BIT)	   #设置9位 精度输出(返回值 0-511)
adc.read()					 #获取重新配置后的测量值

```

 

```python
'''
实验名称：ADC实验
接线说明：ADC电位器-->ESP32 IO
         ADC-->(34)
         
实验现象：程序下载成功后，会在软件shell控制台上输出ADC检测电压值，调节电位器可改变检测电压
         
注意事项：

'''

#导入Pin模块
from machine import Pin
from machine import ADC
from machine import Timer


#定义ADC控制对象
adc=ADC(Pin(34))
adc.atten(ADC.ATTN_11DB)  #开启衰减，量程增大到3.3V

#定时器0中断函数
def time0_irq(time0):
    adc_vol=3.3*adc.read()/4095
    print("ADC检测电压：%.2fV" %adc_vol)
    
#程序入口
if __name__=="__main__":
    time0=Timer(0)  #创建time0定时器对象
    time0.init(period=500,mode=Timer.PERIODIC,callback=time0_irq)
    while True:
        pass


```

