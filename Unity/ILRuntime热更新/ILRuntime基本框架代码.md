UnitySingleton单例

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class UnitySingleton<T> : MonoBehaviour where T : Component
{
    private static T _instance = null;

    public static T Instance
    {
        get
        {
            if(_instance==null)
            {
                _instance = FindObjectOfType(typeof(T)) as T;
                if (_instance == null)
                {
                    GameObject obj = new GameObject();
                    _instance = (T)obj.AddComponent(typeof(T));
                    obj.hideFlags = HideFlags.DontSave;
                    obj.name = typeof(T).Name;
                }
            }
            return _instance;
        }
    }

    public virtual void Awake()
    {
        DontDestroyOnLoad(this.gameObject);
        if(_instance == null)
        {
            _instance = this as T;
        }
        else
        {
            GameObject.Destroy(this.gameObject);
        }
    }
}

```

ResMgr资源加载

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class ResMgr : UnitySingleton<ResMgr>
{
    public override void Awake()
    {
        base.Awake();
        //this.gameObject.AddComponent<AssetBundleManagger>();
    }

    public T GetAssetCache<T>(string name) where T : UnityEngine.Object
    {
#if UNITY_EDITOR
        //if(AssetBundleConfig.IsEditorMode)
        {
            //string path = AssetBundleUtility.PackagePathToAssetsPath(name);
            string path = "Assets/AssetsPackage" + name;
            UnityEngine.Object target = UnityEditor.AssetDatabase.LoadAssetAtPath<T>(path);
            return target as T;
        }
#endif
        //return AssetBundleManager.Instance.GetAssetCache(name) as T;
        return null;
    }
}

```

ILRuntimeWrapper热更单例

```
using UnityEngine;
using System.Collections;
using System.IO;
using ILRuntime.Runtime.Enviorment;

public class ILRuntimeWrapper : UnitySingleton<ILRuntimeWrapper>
{
    bool isGameStart = false;
    AppDomain appdomain;
    System.IO.MemoryStream fs;
    System.IO.MemoryStream p;
    public override void Awake()
    {
        base.Awake();
        this.isGameStart = false;
        this.appdomain = new ILRuntime.Runtime.Enviorment.AppDomain(); 
    }

    public void LoadHotFixAssembly(byte[] dll,byte[] pdb)
    {
        fs = new MemoryStream(dll);
        p = new MemoryStream(pdb);
        try
        {
            appdomain.LoadAssembly(fs, p, new ILRuntime.Mono.Cecil.Pdb.PdbReaderProvider());
        }
        catch
        {
            Debug.LogError("加载热更DLL失败，请确保已经编译过热更DLL");
            return;
        }

        this.InitializeILRuntime();
    }

    private void InitializeILRuntime()
    {
#if DEBUG && (UNITY_EDITOR || UNITY_ANDROID || UNITY_IPHONE)
        //由于Unity的Profiler接口只允许在主线程使用，为了避免出异常，需要告诉ILRuntime主线程的线程ID才能正确将函数运行耗时报告给Profiler
        appdomain.UnityMainThreadID = System.Threading.Thread.CurrentThread.ManagedThreadId;
#endif
        //这里做一些ILRuntime的注册，HelloWorld示例暂时没有需要注册的
    }

    public void EnterGame()
    {
        isGameStart = true;
        //进入到我们的热更的代码
        appdomain.Invoke("HotFix_Project.Main", "Init", null, null);

    }

    private void Update()
    {
        if (this.isGameStart)
        {
            appdomain.Invoke("HotFix_Project.Main", "Update", null, null);
        }
    }

    private void LateUpdate()
    {
        if (this.isGameStart)
        {
            appdomain.Invoke("HotFix_Project.Main", "LateUpdate", null, null);
        }
    }
    private void FixedUpdate()
    {
        if (this.isGameStart)
        {
            appdomain.Invoke("HotFix_Project.Main", "FixedUpdate", null, null);
        }
    }
    private void OnDestroy()
    {
        if (fs != null)
            fs.Close();
        if (p != null)
            p.Close();
        fs = null;
        p = null;
    }
}

```

GameLaunch游戏入口

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class GameLaunch : UnitySingleton<GameLaunch>
{
    public override void Awake()
    {
        base.Awake();

        //初始化游戏框架：资源管理，网络管理，日志管理
        this.gameObject.AddComponent<ResMgr>();
        this.gameObject.AddComponent<ILRuntimeWrapper>();
        //end

        this.StartCoroutine(this.CheckHotUpdate());
    }

    IEnumerator CheckHotUpdate()
    {
#if UNITY_ANDROID
        WWW www = new WWW(Application.streamingAssetsPath + "/Hotfix/HotFix_Project.dll");
#else
        WWW www = new WWW("file:///" + Application.streamingAssetsPath + "/Hotfix/HotFix_Project.dll");
#endif
        while (!www.isDone)
            yield return null;
        if (!string.IsNullOrEmpty(www.error))
            UnityEngine.Debug.LogError(www.error);
        byte[] dll = www.bytes;
        www.Dispose();

        //PDB文件是调试数据库，如需要在日志中显示报错的行号，则必须提供PDB文件，不过由于会额外耗用内存，正式发布时请将PDB去掉，下面LoadAssembly的时候pdb传null即可
#if UNITY_ANDROID
        www = new WWW(Application.streamingAssetsPath + "/Hotfix/HotFix_Project.pdb");
#else
        www = new WWW("file:///" + Application.streamingAssetsPath + "/Hotfix/HotFix_Project.pdb");
#endif
        while (!www.isDone)
            yield return null;
        if (!string.IsNullOrEmpty(www.error))
            UnityEngine.Debug.LogError(www.error);
        byte[] pdb = www.bytes;
        www.Dispose();

        ILRuntimeWrapper.Instance.LoadHotFixAssembly(dll, pdb);
        ILRuntimeWrapper.Instance.EnterGame();
        yield break;

    }

   
}

```

ILRuntime部分代码

main

```
using System;
using System.Collections.Generic;
using UnityEngine;

namespace HotFix_Project
{
    public class Main
    {
        public static void Init()
        {
            //初始化我们的 热更新项目的框架：自定义事件

            //进入游戏逻辑
            GameApp.Instance.init();
        }

        public static void Update()
        {

        }

        public static void LateUpdate()
        {

        }

        public static void FixedUpdate()
        {

        }
    }
}
```

GameApp逻辑代码

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

public class GameApp
{
    public static GameApp Instance = new GameApp();

    public void init()
    {
        this.EnterGameScene();
    }

    private void EnterGameScene()
    {
        //释放地图
    }
}
```

