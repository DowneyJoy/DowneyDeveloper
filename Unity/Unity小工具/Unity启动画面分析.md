在Unity不同版本中，控制启动画面（SplashScreen）的方式存在显著兼容性差异，主要分为三个技术演进阶段：
## Unity 5.x
### 无程序化控制API
 - 启动画面只能通过Player Settings静态配置
 - 启用路径：Edit > Project Settings > Player > Splash Image
 - PC/Mobile平台需要企业版许可证才能完全移除Unity Logo
## Unity 2017.3~2019.x
### 引入SplashScreen类

```csharp
// 首次支持动态控制 (Unity >= 2017.3)
SplashScreen.Stop(SplashScreen.StopBehavior.StopImmediate);
```
### StopBehavior枚举值：
| 选项          | 行为     | 适用平台  |
| ------------- | -------- | --------- |
| StopImmediate | 立即消失 | 全平台    |
| StopFadeOut   | 平滑淡出 | 除WebGL外 |

### 等待机制优化

```csharp
// 检测启动画面状态
while (!SplashScreen.isFinished) 
{
    yield return null; // 协同等待
}

```
### WebGL特殊处理
 - 需等待页面加载完成才能跳过

```csharp
#if UNITY_WEBGL
Application.focusChanged += (focused) => {
    if (focused) SplashScreen.Stop();
};
#endif

```
## Unity 2020.x+
### 时序控制强化
RuntimeInitializeOnLoadMethod新参数：

```csharp
[RuntimeInitializeOnLoadMethod(RuntimeInitializeLoadType.BeforeSplashScreen)]
static void AutoSkip() => SplashScreen.Stop();
```
关键加载时机对比：
| 加载类型           | 执行时机       |
| ------------------ | -------------- |
| BeforeSplashScreen | 在渲染第一帧前 |
| AfterSplashScreen  | 启动画面结束后 |

### 多画面序列管理
 - 支持串联多个启动画面：
	Unity Logo → 自定义Logo1 → 自定义Logo2

 - 每个画面独立设置：
	 - 最小显示时长Duration
	 - 过渡动画（淡入时长FadeIn  >= 200ms|淡出时长FadeOut <=500ms）
	 - 序列时长 = Duration + FadeIn + FadeOut
	 - 序列总时长 = Σ(每个画面的序列时长)

## 直接跳过Unity Logo
 - 放入非Editor文件夹下
 - 使用 [Preserve] 特性：通过注解告知 Unity链接器保留特定代码
	 - 程序集：在任意 C# 文件的命名空间前添加，保留整个程序集的所有使用过的类型；
	 - 类 / 类型：保留类型及其默认构造函数；
	 - 方法 / 属性 / 字段 / 事件 / 委托：保留自身及关联的类型（如方法参数类型、属性值类型）；
 - 如果场景资源过大，会有几秒画面黑屏，建议开启SplashScene
```csharp
#if !UNITY_EDITOR // 仅在非编辑器环境生效
using UnityEngine;
using UnityEngine.Rendering;
using UnityEngine.Scripting;

[Preserve]
public static class SkipUnityLogo
{
    [RuntimeInitializeOnLoadMethod(RuntimeInitializeLoadType.BeforeSplashScreen)]
    private static void Skip()
    {
#if UNITY_WEBGL
        Application.focusChanged += OnFocusChanged;
#else
        System.Threading.Tasks.Task.Run(AsyncSkip);
#endif
    }

    // WebGL平台：窗口获得焦点时跳过
#if UNITY_WEBGL
    private static void OnFocusChanged(bool isFocused)
    {
        if (!isFocused) return;
        Application.focusChanged -= OnFocusChanged;
        SplashScreen.Stop(SplashScreen.StopBehavior.StopImmediate);
    }
#endif

    // 其他平台：异步线程跳过
    private static void AsyncSkip()
    {
        SplashScreen.Stop(SplashScreen.StopBehavior.StopImmediate);
    }
}
#endif

```
## 条件触发跳过
 - 禁用Unity内置的SplashScreen
 - 自定义一个SplashScreen
### 输入任意键
```csharp
public class ConditionalSplashSkip : MonoBehaviour
{
    [SerializeField] private float minShowTime = 2.0f; // 最少显示时长
    [SerializeField] private SplashScreen.StopBehavior stopMode = SplashScreen.StopBehavior.StopFadeOut;
    IEnumerator Start()
    {
        float timer = 0;
        bool skipped = false;
        SplashScreen.Begin();

        while (!SplashScreen.isFinished)
        {
            timer += Time.deltaTime;
            
            // 条件1：强制显示最短时间
            // 条件2：用户按任意键跳过
            if (timer > minShowTime && Input.anyKeyDown)
            {
                SplashScreen.Stop(stopMode);
                skipped = true;
                break;
            }
            yield return null;
        }
        
        if (!skipped) Debug.Log("启动画面完整播放");
    }
}

```
### 场景异步加载完成
```csharp
IEnumerator Start()
{
    SplashScreen.Begin();
    var asyncLoad = SceneManager.LoadSceneAsync("MainScene");
    asyncLoad.allowSceneActivation = false;  // 禁止自动切换
    
    while (!asyncLoad.isDone || !SplashScreen.isFinished)
    {
        // 资源加载90%且用户未强制跳过时等待序列结束
        if (asyncLoad.progress >= 0.9f && !Input.anyKeyDown) {
            await SplashScreenCompletion();
        }
        yield return null;
    }
    asyncLoad.allowSceneActivation = true;  // 激活场景
}

IEnumerator await SplashScreenCompletion() {
    while (!SplashScreen.isFinished) yield return null;
}

```
## 自定义的启动序列替代Unity内置的SplashScreen
 - 禁用Unity内置的SplashScreen

 - 自己实现一个多画面序列（例如使用一个场景，里面包含多个画面图片的切换动画）

 - 在这个自定义序列的场景中，可以同时进行资源加载并显示进度条
```csharp
using UnityEngine;
using UnityEngine.SceneManagement;
using UnityEngine.UI;

public class CustomSplashScreen : MonoBehaviour
{
    [SerializeField] private Sprite[] splashImages; // 序列画面图片数组
    [SerializeField] private Image displayImage;    // 用于显示图片的Image组件
    [SerializeField] private Slider progressBar;    // 进度条
    [SerializeField] private Text progressText;     // 进度文本（可选）
    [SerializeField] private float perImageDuration = 2f; // 每张图片显示时间

    private int currentIndex = 0;
    private float timer = 0f;
    private AsyncOperation loadOperation;

    void Start()
    {
        // 开始加载主场景，但不激活
        loadOperation = SceneManager.LoadSceneAsync("MainScene");
        loadOperation.allowSceneActivation = false;

        // 显示第一张图片
        if (splashImages.Length > 0)
            displayImage.sprite = splashImages[0];
    }

    void Update()
    {
        // 更新当前图片的显示时间
        timer += Time.deltaTime;

        if (timer >= perImageDuration)
        {
            // 切换下一张图片
            currentIndex++;
            if (currentIndex < splashImages.Length)
            {
                displayImage.sprite = splashImages[currentIndex];
                timer = 0f;
            }
            // 如果已经是最后一张图片，则不再切换，等待加载完成（或可以循环显示）
        }

        // 更新加载进度
        // 注意：场景加载的进度在allowSceneActivation=false时最多到0.9
        float progress = loadOperation.progress;
        if (progress >= 0.9f)
        {
            // 此时加载已经完成，等待我们激活
            progressBar.value = 1.0f;
            if (progressText != null)
                progressText.text = "100%";
            // 这里可以添加一个延时或等待用户点击，然后激活场景
            // 或者直接激活：loadOperation.allowSceneActivation = true;
            // 但通常我们会等待序列播放完毕，或者用户点击跳过
        }
        else
        {
            progressBar.value = progress;
            if (progressText != null)
                progressText.text = string.Format("{0:F0}%", progress * 100);
        }

        // 检查是否可以激活场景：当加载已经完成（达到0.9）且当前序列播放完毕（或用户跳过）时
        if (progress >= 0.9f && (currentIndex >= splashImages.Length || Input.anyKeyDown))
        {
            loadOperation.allowSceneActivation = true;
        }
    }
}

```

```C#
Using System.Threading.Tasks;
using UnityEngine;
using UnityEngine.Rendering;
using UnityEngine.Scripting;

[Preserve]//此特性用于防止在打包的时候这个脚本没有被打包进程序
public class SkipSplashImage
{
	//此特性用于在启动画面显示之前执行这个方法
	[RuntimeInitializeOnLoadMethod(RuntimeInitializeLoadType.BeforeSplashScreen)]
	private static void Run()
	{
		SplashScreen.Stop(SplashScreen.StopBehavior.StopImmediate);
	}
}
```

