这是一个基于 **Unity (C#) + 纯前端网页 (HTML/JS)** 的完整局域网图片传输方案。该方案彻底摆脱了微信小程序的域名限制、审核流程和网络环境限制，只需手机与电脑处于同一 Wi-Fi 下，扫码即可实现原图上传与实时展示。

------

### 📌 一、 方案概述与架构

**核心目标**：手机端通过浏览器（扫码）选择图片，通过局域网 HTTP 协议上传至 Unity 程序，Unity 完成接收、落盘存储并在 UI 上自适应展示。

**架构流程**：

1. **服务端（Unity）**：内嵌 `HttpListener` 开启轻量级 HTTP 服务，提供网页访问、接收上传请求、解析图片数据、多线程更新 UI。
2. **客户端（手机浏览器）**：扫码打开 Unity 提供的 Web 页面，进行相册选择/相机拍照，通过前端 Canvas 修正图片方向并压缩，使用 AJAX 发送 `multipart/form-data` 请求。
3. **通信媒介**：纯局域网 Wi-Fi（建议在 Unity 端生成包含本机 IP 地址的二维码供手机扫描）。

------

### ⚙️ 二、 核心模块实现原理

#### 1. Unity 服务端 (`LanImageReceiver.cs`)

- **HTTP 服务构建**：利用 `System.Net.HttpListener` 监听 `http://+:8080/`（需管理员权限），在独立后台线程 `ListenLoop` 中处理请求，避免阻塞 Unity 主线程。
- **路由分发**：
  - `GET /`：读取并返回 `StreamingAssets/index.html` 中的网页代码。
  - `POST /upload`：接收前端传来的文件流。
- **原生 Multipart 解析**：不依赖任何第三方包，通过字节级索引查找 `boundary`，精准剥离出 `name="image"` 的文件数据。
- **文件存储**：将提取出的字节流保存为 `.jpg` 至 `Application.streamingAssetsPath/UploadedImages/` 目录下。
- **线程安全 UI 更新**：由于 HTTP 请求在子线程，将图片字节放入 `ConcurrentQueue<byte[]>` 队列。在 Unity 主线程的 `Update()` 中出队，转为 `Texture2D` 并赋给 UI。
- **自适应展示**：动态挂载 `AspectRatioFitter` 组件，并强制 `RectTransform` 铺满全屏。根据图片宽高比，利用 `FitInParent` 模式，实现**横图横向居中展示，竖图竖向居中展示**，绝不拉伸变形。

```c#
using System;
using System.Collections.Concurrent;
using System.IO;
using System.Net;
using System.Text;
using System.Threading;
using UnityEngine;
using UnityEngine.UI;

public class LanImageReceiver : MonoBehaviour
{
    [Header("服务设置")]
    [Tooltip("服务监听的端口号")]
    public int port = 8080;
    [Tooltip("图片保存的文件夹（位于 StreamingAssets 下）")]
    public string saveFolderName = "UploadedImages";

    [Header("UI 引用")]
    [Tooltip("用于展示接收到的图片的 RawImage")]
    public RawImage displayImage;

    private HttpListener _listener;
    private Thread _listenerThread;
    private volatile bool _isRunning;

    // 用于主线程更新 UI 的线程安全队列
    private ConcurrentQueue<byte[]> _imageQueue = new ConcurrentQueue<byte[]>();
    private Texture2D _currentTexture;
    private string _htmlContent = "";

    void Start()
    {
        LoadHtmlContent();
        StartServer();
    }

    void Update()
    {
        if (_imageQueue.TryDequeue(out byte[] imageBytes))
        {
            UpdateImageUI(imageBytes);
        }
    }

    // 读取网页文件
    private void LoadHtmlContent()
    {
        string htmlPath = Path.Combine(Application.streamingAssetsPath, "index.html");
        if (File.Exists(htmlPath))
        {
            _htmlContent = File.ReadAllText(htmlPath);
            Debug.Log("成功加载 index.html");
        }
        else
        {
            Debug.LogWarning("未找到 index.html，将返回默认提示网页");
            _htmlContent = "<h1 style='text-align:center; margin-top:50px;'>Unity 服务已启动<br><br><span style='color:red;'>但未找到 index.html，请将其放入 StreamingAssets 目录下</span></h1>";
        }
    }

    // 主线程更新 UI 并自适应横竖屏
    private void UpdateImageUI(byte[] imageBytes)
    {
        if (displayImage == null) return;

        Texture2D newTex = new Texture2D(2, 2);
        if (newTex.LoadImage(imageBytes))
        {
            if (_currentTexture != null) Destroy(_currentTexture);
            _currentTexture = newTex;
            displayImage.texture = _currentTexture;

            // 强制将 RawImage 的锚点铺满全屏
            var rt = displayImage.rectTransform;
            rt.anchorMin = Vector2.zero; 
            rt.anchorMax = Vector2.one;  
            rt.offsetMin = Vector2.zero;
            rt.offsetMax = Vector2.zero;
            rt.localPosition = Vector3.zero; 

            // 使用 AspectRatioFitter 自动适配比例（横图横向展示，竖图竖向展示）
            var fitter = displayImage.GetComponent<AspectRatioFitter>();
            if (fitter == null)
                fitter = displayImage.gameObject.AddComponent<AspectRatioFitter>();

            fitter.aspectRatio = (float)newTex.width / newTex.height;
            // FitInParent 保证图片完整显示并居中，不拉伸变形
            fitter.aspectMode = AspectRatioFitter.AspectMode.FitInParent;

            Debug.Log($"UI 已更新展示新图片 (分辨率: {newTex.width}x{newTex.height})");
        }
    }

    private void StartServer()
    {
        _listener = new HttpListener();
        string prefix = $"http://+:{port}/";
        _listener.Prefixes.Add(prefix);

        try
        {
            _listener.Start();
            _isRunning = true;
            _listenerThread = new Thread(ListenLoop) { IsBackground = true };
            _listenerThread.Start();
            
            string ip = GetLocalIPAddress();
            Debug.Log($"<color=green>局域网服务已启动</color>");
            Debug.Log($"请用手机浏览器访问: <b>http://{ip}:{port}/</b>");
        }
        catch (Exception e)
        {
            Debug.LogError($"启动服务失败: {e.Message}。\n(注意：Windows 系统监听 '+' 需要以管理员身份运行 Unity)");
        }
    }

    private void ListenLoop()
    {
        while (_isRunning && _listener.IsListening)
        {
            try
            {
                HttpListenerContext context = _listener.GetContext();
                ThreadPool.QueueUserWorkItem(o => HandleRequest(context));
            }
            catch (HttpListenerException) when (!_isRunning) { break; }
            catch (Exception e) { Debug.LogError($"监听循环异常: {e}"); }
        }
    }

    private void HandleRequest(HttpListenerContext context)
    {
        HttpListenerRequest request = context.Request;
        HttpListenerResponse response = context.Response;

        // CORS 跨域头
        response.Headers.Add("Access-Control-Allow-Origin", "*");
        response.Headers.Add("Access-Control-Allow-Methods", "GET, POST, OPTIONS");
        response.Headers.Add("Access-Control-Allow-Headers", "Content-Type");

        if (request.HttpMethod == "OPTIONS")
        {
            response.StatusCode = 200;
            response.Close();
            return;
        }

        try
        {
            // 1. 处理 GET 请求，返回网页
            if (request.HttpMethod == "GET" && (request.Url.AbsolutePath == "/" || request.Url.AbsolutePath.Equals("/index.html", StringComparison.OrdinalIgnoreCase)))
            {
                byte[] buffer = Encoding.UTF8.GetBytes(_htmlContent);
                response.StatusCode = 200;
                response.ContentType = "text/html; charset=utf-8";
                response.ContentLength64 = buffer.Length;
                response.OutputStream.Write(buffer, 0, buffer.Length);
                return;
            }

            // 2. 处理 POST 请求，接收图片
            if (request.HttpMethod == "POST" && request.Url.AbsolutePath.Equals("/upload", StringComparison.OrdinalIgnoreCase))
            {
                byte[] bodyData;
                using (MemoryStream ms = new MemoryStream())
                {
                    request.InputStream.CopyTo(ms);
                    bodyData = ms.ToArray();
                }

                string contentType = request.ContentType;
                if (string.IsNullOrEmpty(contentType) || !contentType.Contains("boundary="))
                {
                    Respond(response, 400, "缺少 multipart boundary");
                    return;
                }

                string boundary = "--" + contentType.Substring(contentType.IndexOf("boundary=") + 9);
                byte[] boundaryBytes = Encoding.UTF8.GetBytes(boundary);

                // 提取图片数据 (对应 html 中的 name="image")
                byte[] imageBytes = ExtractFileBytes(bodyData, boundaryBytes, "image");
                if (imageBytes == null || imageBytes.Length == 0)
                {
                    Respond(response, 400, "未找到图片数据");
                    return;
                }

                // 保存图片
                string saveDir = Path.Combine(Application.streamingAssetsPath, saveFolderName);
                if (!Directory.Exists(saveDir)) Directory.CreateDirectory(saveDir);
                
                string fileName = $"img_{DateTime.Now:yyyyMMdd_HHmmss_fff}.jpg";
                string filePath = Path.Combine(saveDir, fileName);
                
                File.WriteAllBytes(filePath, imageBytes);
                Debug.Log($"图片已保存: {filePath} (大小: {imageBytes.Length} 字节)");

                // 加入队列，通知主线程渲染
                _imageQueue.Enqueue(imageBytes);

                string jsonResponse = $"{{\"code\":0,\"msg\":\"success\",\"file\":\"{fileName}\"}}";
                Respond(response, 200, jsonResponse, "application/json");
                return;
            }

            Respond(response, 404, "Not Found");
        }
        catch (Exception e)
        {
            Debug.LogError($"处理请求异常: {e.Message}");
            try { Respond(response, 500, "Internal Server Error"); } catch { }
        }
        finally
        {
            try { response.Close(); } catch { }
        }
    }

    // 解析 Multipart 数据
    private byte[] ExtractFileBytes(byte[] data, byte[] boundary, string targetFieldName)
    {
        int start = IndexOf(data, boundary, 0);
        while (start != -1)
        {
            start += boundary.Length;
            if (start + 1 < data.Length && data[start] == '\r' && data[start + 1] == '\n')
                start += 2;

            int end = IndexOf(data, boundary, start);
            if (end == -1) break;

            int partLength = end - start - 2;
            if (partLength > 0)
            {
                byte[] partData = new byte[partLength];
                Array.Copy(data, start, partData, 0, partLength);

                int headerEnd = IndexOf(partData, Encoding.UTF8.GetBytes("\r\n\r\n"), 0);
                if (headerEnd != -1)
                {
                    string headerText = Encoding.UTF8.GetString(partData, 0, headerEnd);
                    if (headerText.Contains($"name=\"{targetFieldName}\"") || headerText.Contains($"name={targetFieldName}"))
                    {
                        int fileStart = headerEnd + 4;
                        int fileLength = partData.Length - fileStart;
                        byte[] fileBytes = new byte[fileLength];
                        Array.Copy(partData, fileStart, fileBytes, 0, fileLength);
                        return fileBytes;
                    }
                }
            }
            start = end;
        }
        return null;
    }

    private int IndexOf(byte[] data, byte[] pattern, int startIndex)
    {
        for (int i = startIndex; i <= data.Length - pattern.Length; i++)
        {
            bool match = true;
            for (int j = 0; j < pattern.Length; j++)
            {
                if (data[i + j] != pattern[j]) { match = false; break; }
            }
            if (match) return i;
        }
        return -1;
    }

    private void Respond(HttpListenerResponse response, int statusCode, string message, string contentType = "text/plain")
    {
        response.StatusCode = statusCode;
        response.ContentType = contentType;
        byte[] buffer = Encoding.UTF8.GetBytes(message);
        response.ContentLength64 = buffer.Length;
        response.OutputStream.Write(buffer, 0, buffer.Length);
    }

    private string GetLocalIPAddress()
    {
        var host = Dns.GetHostEntry(Dns.GetHostName());
        foreach (var ip in host.AddressList)
        {
            if (ip.AddressFamily == System.Net.Sockets.AddressFamily.InterNetwork)
                return ip.ToString();
        }
        return "127.0.0.1";
    }

    void OnDestroy()
    {
        _isRunning = false;
        if (_listener != null && _listener.IsListening)
        {
            _listener.Stop();
            _listener.Close();
        }
        if (_listenerThread != null && _listenerThread.IsAlive)
        {
            _listenerThread.Join(500);
        }
        if (_currentTexture != null) Destroy(_currentTexture);
        Debug.Log("局域网服务已停止");
    }
}
```



#### 2. 网页端客户端 (`index.html`)

- **零依赖单文件**：纯 HTML + CSS + 原生 JS，无需额外搭建 Nginx/Apache，直接由 Unity 提供访问。

- **EXIF 方向修正与压缩**：手机拍照（尤其是 iOS）经常带有 EXIF 旋转信息。前端利用 `<canvas>` 重新绘制图像，不仅抹平了旋转信息（确保传给 Unity 的竖图像素真正是竖的），还顺带进行了前端压缩（质量 0.8），极大提升局域网传输速度。

- **流畅的交互逻辑**：

  - 去除了 `capture` 属性，允许用户自由选择“拍照”或“从相册选择”。
  - 上传成功后，按钮变为“重新选择并上传”，点击会重置状态并直接唤起系统文件选择器。
  - 使用 `XMLHttpRequest` 监听 `upload.onprogress` 事件，实现直观的上传进度条。

  ```js
  <!DOCTYPE html>
  <html lang="zh-CN">
  <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
      <title>局域网图片上传</title>
      <style>
          body { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif; background-color: #f5f7fa; margin: 0; padding: 20px; display: flex; flex-direction: column; align-items: center; }
          .card { background: #fff; border-radius: 12px; box-shadow: 0 4px 12px rgba(0,0,0,0.1); padding: 24px; width: 100%; max-width: 400px; box-sizing: border-box; text-align: center; }
          h2 { margin-top: 0; color: #333; font-size: 20px; }
          .upload-area { border: 2px dashed #ccc; border-radius: 8px; padding: 30px 20px; margin-bottom: 20px; cursor: pointer; position: relative; overflow: hidden; background: #fafafa; }
          .upload-area:active { background-color: #f0f0f0; }
          /* 允许打开相册和相机 */
          input[type="file"] { position: absolute; top: 0; left: 0; width: 100%; height: 100%; opacity: 0; cursor: pointer; }
          .preview-img { max-width: 100%; max-height: 300px; border-radius: 6px; display: none; margin: 0 auto; object-fit: contain; }
          button { background-color: #07c160; color: white; border: none; border-radius: 6px; padding: 14px 20px; font-size: 16px; width: 100%; cursor: pointer; transition: background 0.3s; }
          button:disabled { background-color: #9cd9b4; cursor: not-allowed; }
          .progress-bar { width: 100%; background-color: #e0e0e0; border-radius: 4px; margin-top: 20px; display: none; overflow: hidden; }
          .progress-inner { width: 0%; height: 8px; background-color: #07c160; transition: width 0.2s; }
          .status-msg { margin-top: 15px; font-size: 14px; font-weight: 500; }
          .success { color: #07c160; }
          .error { color: #fa5151; }
      </style>
  </head>
  <body>
  
  <div class="card">
      <h2>扫码上传图片</h2>
      
      <div class="upload-area" id="dropZone">
          <!-- 注意：没有 capture 属性，这样手机点击可以自由选择相册或拍照 -->
          <input type="file" id="fileInput" accept="image/*">
          <div id="uploadHint">
              <svg width="40" height="40" viewBox="0 0 24 24" fill="none" stroke="#999" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"></path><polyline points="17 8 12 3 7 8"></polyline><line x1="12" y1="3" x2="12" y2="15"></line></svg>
              <p style="color: #666; margin: 10px 0 0 0; font-size: 14px;">点击选择相册或拍照</p>
          </div>
          <img id="preview" class="preview-img" alt="预览图">
      </div>
  
      <button id="uploadBtn" disabled>立即上传</button>
      
      <div class="progress-bar" id="progressBar">
          <div class="progress-inner" id="progressInner"></div>
      </div>
      
      <div class="status-msg" id="statusMsg"></div>
  </div>
  
  <script>
      const fileInput = document.getElementById('fileInput');
      const preview = document.getElementById('preview');
      const uploadHint = document.getElementById('uploadHint');
      const uploadBtn = document.getElementById('uploadBtn');
      const progressBar = document.getElementById('progressBar');
      const progressInner = document.getElementById('progressInner');
      const statusMsg = document.getElementById('statusMsg');
  
      let selectedFile = null;
  
      // 重置界面
      function resetUI() {
          selectedFile = null;
          fileInput.value = '';
          preview.style.display = 'none';
          uploadHint.style.display = 'block';
          uploadBtn.textContent = '立即上传';
          uploadBtn.disabled = true;
          progressBar.style.display = 'none';
          progressInner.style.width = '0%';
          statusMsg.textContent = '';
          statusMsg.className = 'status-msg';
      }
  
      // 1. 选择图片，使用 Canvas 纠正 EXIF 旋转并压缩
      fileInput.addEventListener('change', function(e) {
          const file = e.target.files[0];
          if (!file) return;
  
          statusMsg.textContent = '正在处理图片...';
          statusMsg.className = 'status-msg';
  
          const reader = new FileReader();
          reader.onload = function(event) {
              const img = new Image();
              img.onload = function() {
                  const canvas = document.createElement('canvas');
                  canvas.width = img.width;
                  canvas.height = img.height;
                  
                  const ctx = canvas.getContext('2d');
                  ctx.drawImage(img, 0, 0, img.width, img.height);
  
                  // 转换为 Blob，质量 0.8 进行压缩，加快局域网传输
                  canvas.toBlob(function(blob) {
                      selectedFile = new File([blob], file.name || "upload.jpg", { type: "image/jpeg" });
                      
                      // 显示修正方向后的预览图
                      preview.src = canvas.toDataURL('image/jpeg', 0.8);
                      preview.style.display = 'block';
                      uploadHint.style.display = 'none';
                      
                      uploadBtn.disabled = false;
                      uploadBtn.textContent = '立即上传';
                      statusMsg.textContent = '';
                  }, 'image/jpeg', 0.8);
              };
              img.src = event.target.result;
          }
          reader.readAsDataURL(file);
      });
  
      // 2. 上传逻辑
      uploadBtn.addEventListener('click', function() {
          // 如果已经是成功上传后的状态，点击则重置并重新选择
          if (this.textContent === '重新选择并上传') {
              resetUI();
              fileInput.click(); 
              return;
          }
  
          if (!selectedFile) return;
  
          const formData = new FormData();
          // ⚠️ 必须与 Unity 中解析的字段名 "image" 一致
          formData.append('image', selectedFile);
  
          const xhr = new XMLHttpRequest();
          xhr.open('POST', '/upload', true);
  
          uploadBtn.disabled = true;
          uploadBtn.textContent = '上传中...';
          progressBar.style.display = 'block';
          statusMsg.textContent = '';
          statusMsg.className = 'status-msg';
          progressInner.style.width = '0%';
  
          xhr.upload.onprogress = function(e) {
              if (e.lengthComputable) {
                  const percent = Math.round((e.loaded / e.total) * 100);
                  progressInner.style.width = percent + '%';
              }
          };
  
          xhr.onload = function() {
              if (xhr.status >= 200 && xhr.status < 300) {
                  statusMsg.textContent = '✅ 上传成功！';
                  statusMsg.className = 'status-msg success';
                  uploadBtn.disabled = false; 
                  uploadBtn.textContent = '重新选择并上传';
              } else {
                  statusMsg.textContent = '❌ 服务器返回错误: HTTP ' + xhr.status;
                  statusMsg.className = 'status-msg error';
                  uploadBtn.disabled = false;
                  uploadBtn.textContent = '重试上传';
              }
          };
  
          xhr.onerror = function() {
              uploadBtn.disabled = false;
              uploadBtn.textContent = '重试上传';
              statusMsg.textContent = '❌ 网络请求失败，请检查是否断开连接';
              statusMsg.className = 'status-msg error';
          };
  
          xhr.send(formData);
      });
  </script>
  </body>
  </html>
  ```

  #### 🚀 第三部分：运行步骤（关键必看）

  - **配置场景**：

    - 在 Unity 中新建一个 `Raw Image`（在 Hierarchy 窗口右键 -> UI -> Raw Image）。

    - 将其宽高设为 0（无所谓，代码会自动铺满）。

    - 将 `LanImageReceiver` 脚本挂载到场景里的空物体上。

    - 把 `Raw Image` 拖给脚本的 `Display Image` 槽位。

  - **以管理员身份运行**：
    - **Windows 系统必须在管理员权限下运行**。请关闭 Unity，然后右键 Unity 图标选择“以管理员身份运行”。

  - **获取 IP**：
    - 运行游戏后，看 Unity 的 Console 面板，会打印出类似：`请用手机浏览器访问: http://192.168.1.100:8080/`。

  - **手机扫码/访问**：

    - 确保手机和电脑连在**同一个 Wi-Fi** 下。

    - 用手机浏览器输入控制台打印的地址（或把这个地址生成二维码用手机扫）。

  - **上传体验**：

    - 点击页面的选框，选择相册里的横图或竖图。

    - 点击“立即上传”，Unity 屏幕会瞬间展示该图。**横图会横向撑满居中，竖图会竖向撑满居中**，绝不拉伸。

    - 图片会同时保存在电脑硬盘的 `Assets/StreamingAssets/UploadedImages/` 目录下。

------

### 🛡️ 三、 关键避坑与技术细节

1. **Windows 管理员权限**：Unity 使用 `+` 通配符监听局域网所有网卡时，Windows 系统会严格拦截。**必须右键以管理员身份运行 Unity 或打包后的 EXE**。
2. **CORS 跨域处理**：服务端必须在响应头中带上 `Access-Control-Allow-Origin: *`，并处理 `OPTIONS` 预检请求，防止部分手机浏览器拦截请求。
3. **防火墙入站规则**：如果手机一直转圈无法访问，请检查 Windows 防火墙是否拦截了 8080 端口，或临时关闭防火墙测试。
4. **移动端只读限制**：虽然方案将图片存至 `StreamingAssets`，这在 PC 端没问题。但如果未来你把 Unity 端打包到 Android/iOS，`StreamingAssets` 是只读的，必须改用 `Application.persistentDataPath`。
5. **IP 地址生成**：Unity 启动后应调用 `Dns.GetHostEntry` 获取局域网 IPv4，并利用本地二维码生成库（如 `QRCoder`）将 `http://IP:8080/` 绘制在 Unity 屏幕上供扫码。

------

### 🚀 四、 完整使用流程

1. **准备文件**：将修复好的 HTML 代码保存为 `index.html`，放入 Unity 项目的 `Assets/StreamingAssets/` 目录下。
2. **挂载脚本**：将 `LanImageReceiver.cs` 挂载到场景物体上，并在 Inspector 中绑定好用于显示的 `RawImage`。
3. **启动服务**：以**管理员身份**运行 Unity Play 模式。控制台会打印：`网页访问地址: http://192.168.x.x:8080/`。
4. **手机扫码/访问**：手机连接同一 Wi-Fi，扫描 Unity 生成的二维码（或直接在浏览器输入该 IP 地址）。
5. **图片上传**：在网页中点击“选择相册或拍照” -> 选图预览 -> 点击“立即上传”。
6. **实时展示**：网页显示进度条跑满，Unity 端瞬间渲染出图片，并在本地 `StreamingAssets/UploadedImages/` 中找到原图文件。

------

### 💎 方案总结

这个方案用**极低的开发成本**（仅一个 C# 脚本 + 一个 HTML 文件）实现了跨平台的局域网文件传输系统。它巧妙规避了微信小程序的网络限制，处理了移动端拍照的 EXIF 旋转痛点，并通过原生多线程解决了 Unity UI 阻塞问题。非常适合用于**展会互动、局域网多屏互动、摄影棚实时传图**等场景。