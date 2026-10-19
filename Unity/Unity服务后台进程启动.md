# Unity服务后台进程启动

将Unity程序作为Windows服务运行，核心是使用服务包装器（Service Wrapper）工具，并以无头模式（Headless）启动程序。以下是使用 NSSM 的详细实现步骤。
## 核心原理与限制


Windows服务运行在`Session 0`中，这是一个没有图形界面的特殊会话，无法创建窗口或初始化`DirectX/OpenGL`等图形API。因此，Unity程序必须以无头模式启动，通过命令行参数<kbd>-batchmode -nographics</kbd>来实现。如果你的程序需要渲染画面，则无法作为标准服务运行

## 详细实现步骤

`NSSM（Non-Sucking Service Manager）`是一款免费工具，可将任意EXE封装为Windows服务，并支持自动重启和日志重定向。

### 第一步：构建无头Unity程序
在Unity中，将项目构建为Windows Standalone程序。

确保场景中没有任何需要渲染的组件（如Camera、Canvas），所有逻辑必须在Awake或Start中启动，不依赖任何视觉反馈。

构建后，你会得到一个 YourApp.exe 以及 YourApp_Data 文件夹。

### 第二步：下载并准备NSSM
从NSSM官网（nssm.cc）下载最新版（建议v2.24+）。

解压后，根据系统架构选择` win64\nssm.exe`，将其复制到任意方便的位置（如 C:\nssm\）。

### 第三步：创建Windows服务
以管理员身份打开命令提示符（`CMD`）或`PowerShell`，执行以下命令：

```bash
nssm install UnityService "D:\YourApp\YourApp.exe"
```
将 UnityService 替换为你的服务名称，路径替换为你的Unity程序路径。

### 第四步：配置服务参数（关键）
设置启动参数（无头模式）：

```bash
nssm set UnityService AppParameters "-batchmode -nographics -logFile D:\Logs\UnityService.log"
```
 - `batchmode`：以批处理模式运行，不显示任何窗口。
- `nographics`：不初始化图形设备，这是服务运行的关键。
- `logFile`：将日志输出到指定文件，方便排查问题。

设置工作目录：

```bash
nssm set UnityService AppDirectory "D:\YourApp"
```
配置日志重定向（推荐）：

```bash
nssm set UnityService AppStdout "D:\Logs\stdout.log"
nssm set UnityService AppStderr "D:\Logs\stderr.log"
```
NSSM支持将程序的标准输出和错误重定向到文件，便于调试。

设置启动类型：

```bash
nssm set UnityService Start SERVICE_AUTO_START
```
这会让服务随Windows启动而自动运行。

### 第五步：启动并验证服务

```bash
nssm start UnityService
```
检查服务状态：

```bash
nssm status UnityService
```
或打开Windows“服务”管理器（`services.msc`），找到 `UnityService`，确认其状态为“正在运行”。

### 第六步：管理服务
| 操作 | 命令                             |
| ---- | -------------------------------- |
| 启动 | nssm start UnityService          |
| 停止 | nssm stop UnityService           |
| 重启 | nssm restart UnityService        |
| 卸载 | nssm remove UnityService confirm |