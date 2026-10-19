C#

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public delegate void EventLua();

public class TestEvent
{
    public static event EventLua Static;

    public static void StaticFunc()
    {
        Debug.Log("这是静态函数");
    }

    public static void CallStatic()
    {
        if(Static != null)
        {
            Static();
        }
    }

    public event EventLua Dynamic;

    public void CallDynamic()
    {
        if (Dynamic != null)
        {
            Dynamic();
        }
    }
}

public class LuaCallEvent : MonoBehaviour
{
    void Start()
    {
        xLuaEnv.Instance.DoString("require('C2L/LuaCallEvent')");
    }

    void OnDestroy()
    {
        xLuaEnv.Instance.Free();
    }
}

```

Lua

```
-- C#添加事件 TestEvent.Static += TestEvent.StaticFunc

-- Lua添加事件
CS.TestEvent.Static("+", CS.TestEvent.StaticFunc)
CS.TestEvent.CallStatic()
CS.TestEvent.Static("-", CS.TestEvent.StaticFunc)

-- 添加动态成员变量
local func = function()
	print("来自于Lua的回调函数")
end

local obj = CS.TestEvent()
obj:Dynamic("+", func)
obj:CallDynamic()
obj:Dynamic("-", func)

```

