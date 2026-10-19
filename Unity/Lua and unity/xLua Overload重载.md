C#

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class TestOverload
{
    public static void Test(int id)
    {
        Debug.Log("数字类型: " + id);
    }

    public static void Test(string name)
    {
        Debug.Log("字符串类型: " + name);
    }

    public static void Test(int id, string name)
    {
        Debug.Log("两个数值: " + id + ", " + name);
    }
}

public class LuaCallOverload : MonoBehaviour
{
    void Start()
    {
        xLuaEnv.Instance.DoString("require('C2L/LuaCallOverload')");
    }

    void OnDestroy()
    {
        xLuaEnv.Instance.Free();
    }
}

```

Lua

```
-- 数字重载函数
CS.TestOverload.Test(99)

-- 字符串重载函数
CS.TestOverload.Test("admin")

-- 不同参数的重载函数
CS.TestOverload.Test(100, "root")
```

