### 常用函数介绍

- pinMode(pin, mode)：用于配置数字引脚的输入或输出模式。pin是数字引脚的编号，mode是要设置的模式（输入或输出）。
- digitalWrite(pin, value)：用于在数字引脚上写入数字值（HIGH或LOW）。pin是数字引脚的编号，value是要写入的值。
- digitalRead(pin)：用于读取数字引脚上的数字值（HIGH或LOW）。pin是数字引脚的编号。
- analogRead(pin)：用于读取模拟数字引脚上的模拟值（0-1023）。pin是模拟引脚的编号。
- analogWrite(pin, value)：用于在支持PWM输出的数字引脚上输出模拟值（0-255）。pin是数字引脚的编号，value是要输出的值。
- delay(ms)：用于在程序中创建暂停（延迟）时间。ms是要延迟的毫秒数。
- Serial.begin(baud)：用于初始化串口通信，其中baud是波特率。
- Serial.println(data)：用于将数据打印到串口监视器。data可以是数字，字符串或其他数据类型。