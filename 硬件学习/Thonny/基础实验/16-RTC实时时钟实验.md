



```python
'''
实验名称：RTC实时时钟实验
接线说明：
         
实验现象：程序下载成功后，软件shell控制台间隔1S输出RTC实时时钟年月日时分秒星期
         
注意事项：

'''

#导入Pin模块
from machine import Pin
from machine import RTC
import time


#定义RTC控制对象
rtc=RTC() 

#定义星期
week=("星期一","星期二","星期三","星期四","星期五","星期六","星期天")

#程序入口
if __name__=="__main__":
    if rtc.datetime()[0]!=2022:
        rtc.datetime((2022,8,10,2,10,20,58,0))
    while True:
        date_time=rtc.datetime()
        print("%d-%d-%d \t %02d:%02d:%02d \t %s" %(date_time[0],date_time[1],date_time[2],
                                                   date_time[4],date_time[5],date_time[6],
                                                   week[date_time[3]]))
        time.sleep(1)


```

