xLuaEnv扩展Global

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

    //返回Lua环境的全局变量
    public LuaTable Global
    {
        get
        {
            return _Env.Global;
        }
    }
    #endregion
}

```

C#

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using XLua;

public class CsharpCallVariable : MonoBehaviour
{
    private void Start()
    {
        xLuaEnv.Instance.DoString("return require('L2C/CsharpCallVariable')");

        //LuaEnv提供了一个成员变量Global，它可以用于C#获取lua的全局变量
        //Global的数据类型是C#实现的LuaTable，LuaTable是xLua实现的C#和lua中表对应的数据结构
        //xlua会将lua中的全局变量一table的方式全部存储到Global中

        //通过运行环境，导出全局变量，类型是luaTable
        //luatable是C#的数据对象，用于和lua中的全局变量存储的table对应

        LuaTable g = xLuaEnv.Instance.Global;

        //从lua中，将全局变量提取出来
        //参数：lua中全局变量的名称
        //类型：lua中全局变量的名称对应的类型
        //返回值：变量的值
        int num = g.Get<int>("num");
        float rate = g.Get<float>("rate");
        bool isWoman = g.Get<bool>("isWoman");
        string name = g.Get<string>("name");
    }

    private void OnDestroy()
    {
        xLuaEnv.Instance.Free();
    }
}

```

Lua

```
-- 隐性做了 {num = 100, rate=99.99,isWoman=false, name="admin"}

num=100

rate=99.99

isWoman=false

name="admin"
```

