C#

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using XLua;

public class TestExtend
{
    public void Output()
    {
        Debug.Log("类本身带的方法");
    }
}

//类扩展，需要给扩展方法编写的静态类添加[LuaCallCSharp]，否则Lua无法调用到
[LuaCallCSharp]
public static class MyExtend
{
    public static void Show(this TestExtend obj)
    {
        Debug.Log("类扩展实现的方法");
    }
}

public class LuaCallExtend : MonoBehaviour
{
    private void Start()
    {
        xLuaEnv.Instance.DoString("require('C2L/LuaCallExtend')");
    }

    private void OnDestroy()
    {
        xLuaEnv.Instance.Free();
    }
}

```

Lua

```
-- 获取对象
local obj = CS.TestExtend()

obj:Output()
obj:Show()

```

