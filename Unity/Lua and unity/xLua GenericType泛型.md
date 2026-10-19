C#

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class TestGenericType
{
    public void Output<T>(T data)
    {
        Debug.Log("泛型方法：" + data.ToString());
    }

    public void Output(float data)
    {
        Output<float>(data);
    }

    public void Output(string data)
    {
        Output<string>(data);
    }
}

public class LuaCallGenericType : MonoBehaviour
{
    void Start()
    {
        //gameObject.AddComponent<BoxCollider>();
        //gameObject.AddComponent(typeof(BoxCollider));

        xLuaEnv.Instance.DoString("require('C2L/LuaCallGenericType')");
    }

    void OnDestroy()
    {
        xLuaEnv.Instance.Free();
    }
}

```

Lua

```
local obj = CS.TestGenericType()

obj:Output(99)
obj:Output("admin")

local go = CS.UnityEngine.GameObject("LuaCreate")
-- xLua实现了typeof关键字，所以可以用类型API替代Unity内置的泛型方法
go:AddComponent(typeof(CS.UnityEngine.BoxCollider))

```

