Unity中 Lua调用命令

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using XLua;

public class DoString : MonoBehaviour
{
    void Start()
    {
        //LuaCallCSharpCode();
        LuaReturnData();
    }

    //使用Lua调用C#代码
    public void LuaCallCSharpCode()
    {
        LuaEnv env = new LuaEnv();
        //Lua调用C#代码（CS.命名空间.类名.方法名(参数)）
        env.DoString("CS.UnityEngine.Debug.Log('from lua')");
        env.Dispose();
    }

    //Lua返回值给C#
    public void LuaReturnData()
    {
        LuaEnv env = new LuaEnv();
        object[] data = env.DoString("return 100, true");

        Debug.Log("Lua的第一个返回值：" + data[0].ToString());
        Debug.Log("Lua的第二个返回值：" + data[1].ToString());

        env.Dispose();
    }
}

```

Lua加载器

```c#
using System.Collections;
using System.Collections.Generic;
using System.IO;
using UnityEngine;
using XLua;

//Lua是脚本语言，编写代码脚本是实现功能最重要的方式
public class Loader : MonoBehaviour
{
    void Start()
    {
        MyLoader();
    }

    public void SystemLoader()
    {
        LuaEnv env = new LuaEnv();
        //对应test.lua
        //内置加载器会扫描预制的目录，查找是否存在test.lua
        //xLua存在默认加载器，StreamingAssets目录下可以加载文件
        env.DoString("require('test')");
        env.Dispose();
    }

    public void MyLoader()
    {
        LuaEnv env = new LuaEnv();
        //将我定义的加载器，加入到xLua的解析环境中
        env.AddLoader(ProjectLoader);
        env.DoString("require('test1')");

        env.Dispose();
    }

    //自定义加载器
    //自定义加载器，会先于系统内置加载器执行，当自定加载器加载到文件后，后续的加载器则不会继续执行
    //当Lua代码执行require()函数时，自定义加载器会尝试获得文件的内容
    //参数：被加载Lua文件的路径
    //如果需要加载的文件不存在，记得返回null
    public byte[] ProjectLoader(ref string filepath)
    {
        //filepath来自于Lua的require("文件名")
        //构造路径，才能将require加载的文件指向到我们想放Lua的路径下去
        //路径可以任意定制（能不能将Lua代码放入AB包）
        string path = Application.dataPath;
        path = path.Substring(0, path.Length - 7) + "/DataPath/Lua/" + filepath + ".lua";
        
        if(File.Exists(path))
        {
            //将Lua文件读取为字节数组
            //xLua的解析环境，会执行我们自定义加载器返回的Lua代码
            return File.ReadAllBytes(path);
        }
        else
        {
            return null;
        }
    }
}

```

LuaEnv控制器单例

```
using System.Collections;
using System.Collections.Generic;
using System.IO;
using UnityEngine;
using XLua;

public class xLuaEnv
{
    #region Singleton

    private static xLuaEnv _Instance = null;

    //单例的核心实现
    public static xLuaEnv Instance
    {
        get
        {
            if (_Instance == null)
            {
                _Instance = new xLuaEnv();
            }

            return _Instance;
        }
    }

    #endregion

    #region Create LuaEnv

    private LuaEnv _Env;

    //创建单例的时候，Lua运行环境，会一起被创建
    private xLuaEnv()
    {
        _Env = new LuaEnv();
        _Env.AddLoader(_ProjectLoader);
    }

    #endregion

    #region Loader

    //创建自定义Lua加载器，这样就可以任意订制项目的Lua脚本的存储位置
    private byte[] _ProjectLoader(ref string filepath)
    {
        string path = Application.dataPath;
        path = path.Substring(0, path.Length - 7) + "/DataPath/Lua/" + filepath + ".lua";

        if (File.Exists(path))
        {
            return File.ReadAllBytes(path);
        }
        else
        {
            return null;
        }
    }

    #endregion

    #region Free LuaEnv

    public void Free()
    {
        //释放LuaEnv，同时也释放单例对象，这样下次调单例对象，会再次产生Lua运行环境
        _Env.Dispose();
        _Instance = null;
    }

    #endregion

    #region Run Lua

    public object[] DoString(string code)
    {
        return _Env.DoString(code);
    }

    #endregion
}

```

单例测试

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class TestSingleton : MonoBehaviour
{
    void Start()
    {
        xLuaEnv.Instance.DoString("require('test1')");
    }

    void OnDestroy()
    {
        xLuaEnv.Instance.Free();
    }
}

```

