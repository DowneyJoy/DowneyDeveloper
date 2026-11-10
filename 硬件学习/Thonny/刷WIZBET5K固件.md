**MicroPython 官方发布的 ESP32 通用固件默认不带 WIZNET5K 驱动**，所以路线 A 需要你自己**编译一次固件**（把 W5500 支持编进去），之后就用官方 `network.WIZNET5K`，一劳永逸。Windows 上用 **Docker** 编译最干净，不用污染本机环境。

### 第 1 步：Docker 编译固件（约 30–60 分钟，一次搞定）

本机先装 **Docker Desktop**（[docker.com](https://www.docker.com/products/docker-desktop/)），然后：

```
# 克隆 MicroPython 源码（到 C:\Users\dh\micropython）
cd C:\Users\dh
git clone --recursive https://github.com/micropython/micropython.git
cd micropython

# 进入带 ESP-IDF 的编译容器（v5.2.2 对应 MicroPython 较新版本）
docker run -it --rm -v "C:\Users\dh\micropython:/micropython" -w /micropython espressif/idf:v5.2.2 bash
```

容器内依次执行（**关键：第 5 步打开 W5500 支持**）：

```
source /opt/esp/idf/export.sh
cd /micropython/mpy-cross && make -j4
cd /micropython/ports/esp32
make submodules
export MICROPY_PY_WIZNET5K=1          # ← 这一行是关键，编入 W5500 驱动
make BOARD=ESP32_GENERIC -j4
```

编译产物在：`C:\Users\dh\micropython\ports\esp32\build-ESP32_GENERIC\firmware.bin` （DevKitC 一般是 4MB flash，`ESP32_GENERIC` 默认即 4MB。若你的板子是 16MB 芯片，改用 `ESP32_GENERIC_S` 或先确认。）

### 第 2 步：烧录（用 esptool，串口 COM 号在设备管理器查）

```
pip install esptool
esptool.py --chip esp32 --port COM3 erase_flash
esptool.py --chip esp32 --port COM3 --baud 460800 write_flash -z 0x1000 firmware.bin
```

烧完重新插拔 USB，Thonny 右下角会重连到新固件。

**Docker 配置了 DaoCloud 公共镜像加速源，而它把 `espressif/idf` 列入黑名单 / 不在白名单**，拉取被拦截了。要换掉这个加速源。



## 解决：移除 DaoCloud 加速源，改直连 Docker Hub

1. 打开 **Docker Desktop** → 右上角齿轮 **Settings** → **Docker Engine** 选项卡。
2. 在中间的 JSON 里找到 `"registry-mirrors"` 这一项，把里面的 DaoCloud 地址**删掉**（或整项清成 `"registry-mirrors": []`）。
3. 点右下角 **Apply & Restart**，等 Docker 重启。

然后重跑：

```
cd F:\ESP32Pro\Resources\MicroPython\micropython
docker run -it --rm -v "F:\ESP32Pro\Resources\MicroPython\micropython:/micropython" -w /micropython espressif/idf:v5.2.2 bash
```

这次会**直连 Docker Hub 官方**拉取 `v5.2.2`。

## 如果直连 Docker Hub 拉不动（国内常见）

直连可能超时。那就换个允许这个镜像的源，把 `registry-mirrors` 改成下面任一个试试（改完同样 Apply & Restart）：

```
"registry-mirrors": ["https://docker.m.daocloud.io"]
```

或

```
"registry-mirrors": ["shturl.cc/BvpWgo2Y6z31VKx5i"]
```

## `lib/micropython-lib` 是个**空目录**

git 只登记了 commit（`660ddd8`）但文件没检出到工作树。需要强制检出。

### 容器里执行

```
cd /micropython
git submodule update --init --force --checkout lib/micropython-lib
ls lib/micropython-lib
```

`ls` 后应该能看到 `micropython`、`README.md` 等内容（不再为空）。

### 如果还是空的

删掉这个空目录，重新检出（这一步 git 会重新 checkout 文件，可能需要联网；若报 GitHub 连不上，回 Windows 宿主机执行同样的命令）：

```
rm -rf /micropython/lib/micropython-lib
cd /micropython
git submodule update --init lib/micropython-lib
ls lib/micropython-lib
```

宿主机版本（PowerShell，若容器网络不行）：

```
cd F:\ESP32Pro\Resources\MicroPython\micropython
Remove-Item -Recurse -Force lib\micropython-lib
git submodule update --init lib/micropython-lib
dir lib\micropython-lib
```

### 内容确认后，重编

```
cd /micropython/ports/esp32
rm -rf build-ESP32_GENERIC
make BOARD=ESP32_GENERIC submodules
export MICROPY_PY_WIZNET5K=1
make BOARD=ESP32_GENERIC -j4
```

**编译成功！** 固件已生成（application 1.69MB，总共 1.76MB，4MB flash 还剩很多空间）。

现在回到 **Windows 宿主机**烧录（容器里没有串口）。

## 1. 装 esptool（宿主机 PowerShell）

```
pip install esptool
```

## 2. 查串口号

设备管理器 →「端口 (COM 和 LPT)」，找到 `Silicon Labs CP210x` 或 `USB-SERIAL CH340` 对应的 **COMx**（比如 COM3）。

## 3. 擦除 + 烧录

先断电重连 USB，然后（把 `COM3` 换成你的实际串口）：

```
cd F:\ESP32Pro\Resources\MicroPython\micropython\ports\esp32\build-ESP32_GENERIC

# 擦除（会清掉板上旧文件，正好清理掉之前那些纯Python驱动 .py）
python -m esptool --chip esp32 -p COM3 erase_flash

# 烧录
python -m esptool --chip esp32 -p COM3 -b 460800 --before default_reset --after hard_reset write_flash --flash_mode dio --flash_size 4MB --flash_freq 40m 0x1000 bootloader\bootloader.bin 0x8000 partition_table\partition-table.bin 0x10000 micropython.bin
```

烧完看到 `Hash of data verified.` / `Hard resetting via RTS pin...` 就成功了。

## 4. 验证 W5500 驱动已编进固件

重新插拔 USB，打开 Thonny，在 REPL 里输入：

```
import network
print([x for x in dir(network) if 'WIZ' in x])
```

**期望输出 `['WIZNET5K']`**—— 有它，说明新固件带 W5500 驱动，路线 A 成功。