**Thonny 只能识别 MicroPython 固件**，必须重刷 MicroPython 才能连上

### 步骤（Thonny 内置 esptool 直接刷，最简单）

1. 装好串口驱动（CP2102 / CH340，设备管理器看 COM 口），用**带数据的 USB 线**插上 ESP32
2. 打开 Thonny → 右下角解释器 → **Configure interpreter**
3. 解释器选：`MicroPython (ESP32)`，端口选你板子对应的 COM
4. 点下方 **Install or update MicroPython (esptool)**
5. 芯片家族选 ESP32（C3/S3 要对应选），勾选 **Erase all flash before installing**（必须勾，擦掉原来 Arduino 程序），选通用版本，点 Install
6. 烧录时如果超时识别不到：按住 **BOOT**，再按一下 RESET 松开，保持按住 BOOT 直到开始烧录，烧完再松开 BOOT
7. 烧录完成，板子自动重启，Shell 出现 `>>>` 提示符，就是识别成功

### 备选方案（ESP Flasher 工具）

下载 ESP32 Flash Download Tool，擦除 Flash，再写入对应 MicroPython bin 固件，地址：

- 原版 ESP32：`0x1000`
- ESP32-C3 / S3：`0x0`

### 常见坑

- 只充电线：识别不到串口
- 没擦 Flash：Arduino 残留，刷完还是连不上
- 芯片选错：C3/S3 不能用普通 ESP32 固件
- 驱动异常：设备管理器有黄色感叹号，重装 CH340/CP210x 驱动

## 方案 A：ESP Flash Download Tool（推荐，图形化，新手友好）

1. 先确认你的 ESP 型号：普通 ESP32 / ESP32-C3 / ESP32-S3

2. 去官网下载对应 

   ```
   .bin
   ```

    固件：

   - 普通 ESP32：`ESP32_GENERIC`
   - C3：`ESP32C3_GENERIC`
   - S3：`ESP32S3_GENERIC` 👉 官网：https://micropython.org/download/

3. 打开 

   ```
   ESP Flash Download Tool
   ```

   - 普通 ESP32：固件地址填 `0x1000`；C3/S3 填 `0x0`
   - 勾选 `ERASE` 擦除 Flash（把原来 Arduino 程序清干净）
   - 选择你的 COM 口，波特率 460800
   - 按住板子 BOOT 键，点 START，松开 BOOT 等待烧录完成

4. 烧完重启 ESP32，回到 Thonny，右下角解释器选 `MicroPython(ESP32)`，端口选对应 COM，直接就能连上，出现`>>>`

## 方案 B：命令行 esptool（如果你习惯命令）

1. 安装 esptool：

```
pip install esptool
```

1. 先擦除（替换 COMx 为你的端口，如 COM6）

```
esptool --chip esp32 --port COMx erase_flash
```

1. 烧录固件（普通 ESP32，把 bin 换成你下载好的文件名）

```
esptool --chip esp32 --port COMx write_flash -z 0x1000 ESP32_GENERIC-xxx.bin
```

> C3/S3 把`--chip esp32`改成`esp32c3`/`esp32s3`，地址改成`0x0`

## 方案 C：Thonny 里的自定义固件（部分版本支持）

在这个报错窗口，点左下角 **≡（三条横线）**，里面有选项：**Install from local file**，直接加载你已经下载好的 bin 固件，不用在线下载列表，直接在 Thonny 内完成烧录。

## 避坑提醒

1. 必须用**带数据传输的 USB 线**，只充电的线串口识别不到
2. 烧录时的 BOOT+RESET 操作：按住 BOOT，按一下 RESET 松开，保持 BOOT 按住直到烧录开始
3. 烧录完成后，一定要重启板子，再打开 Thonny 连接