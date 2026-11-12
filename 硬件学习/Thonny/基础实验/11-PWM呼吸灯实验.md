### PWM

‌PWM 全称是脉冲宽度调制（Pulse Width Modulation），它是一种通过快速开关信号并调节“通电时间比例”（即占空比）来控制平均电压或功率的技术‌。

- 高电平占75%，低电平占25%：（3.75V/5V）
- 高电平占50%，低电平占50%：（2.5V/5V）
- 高电平占20%，低电平占80%：（1V/5V）

### 实验目的

通过调节PWM占空比，控制LED指示灯亮度变化，实现呼吸灯效果，即由暗变亮，再由亮变暗的循环过程。

### 构造函数

- pwm25=machine.PWM(machine.Pin(id),freq,duty)
  - 构建PWM对象。id：引脚编号；freq：频率值；duty：占空比；配置完后PWM自动生效。

### 使用方法

- pwm25.freq(freq)
  - 设置频率。freq：频率值在1-40MHz之间，freq为空时表示获取当前频率值。
- pwm25.duty(duty)
  - 设置占空比。duty：占空比在0-1023之间，duty为空时表示获取当前占空比值。
- pwm25.deinit()
  - 关闭PWM

```python
from machine import Pin,PWM

pwm0 = PWM(Pin (0))    		# create PWM object from a pin
freq = pwm0.freq()   		# get current frequency (default 5kHz)
pwm0.freq(1000)      		# set PWM frequency from 1Hz to 40MHz

duty = pwme.duty()			# get current duty cycle, range 0-1023 (default 512, 50%)
pwm0.duty(256)			    # set duty cycle from 0 to 1023 as a ratio duty/1023, (now 25%)

duty_u16 = pwm0.duty_u16()   # get current duty cycle, range 0-65535
pwm0.duty_u16(2**16*3//4)    # set duty cycle from 0 to 65535 as a ratio duty u16/65535, (now 75%)

duty_ns = pwme.duty_ns()	 # get current pulse width in ns
pwm0.duty_ns(250_000)		 # set pulse width in nanoseconds from 0 to 1_000_000_000/freq, (now 25%)

pwm0.deinit()				# turn off PWM on the pin

pwm2 = PWM(Pin(2), freq=20000, duty=512)	# create and configure in one go
print(pwm2) 	 						  # view PWM settings
```



```python
'''
实验名称：PWM呼吸灯实验
接线说明：LED模块-->ESP32 IO
         (D1)-->(15)
         
实验现象：程序下载成功后，D1指示灯呈现呼吸灯效果，由暗变亮，再由亮变暗
注意事项：

'''

#导入Pin模块
from machine import Pin
from machine import PWM
import time

#定义LED1控制对象
led1=PWM(Pin(15),freq=1000,duty=0)


#程序入口
if __name__=="__main__":
    duty_value=0
    fx=1
    while True:
        if fx==1:
            duty_value+=10
            if duty_value>1010:
                fx=0
        else:
            duty_value-=10
            if duty_value<10:
                fx=1
        led1.duty(duty_value)
        time.sleep_ms(10)


```



