C#

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public struct TestStruct
{
    public string Name;

    public string Output()
    {
        return Name;
    }
}

public class LuaCallStruct : MonoBehaviour
{
    void Start()
    {
        xLuaEnv.Instance.DoString("require('C2L/LuaCallStruct')");
    }

    void OnDestroy()
    {
        xLuaEnv.Instance.Free();
    }
}

```

Lua

```
-- 和对象调用保持一致
local obj = CS.TestStruct()

obj.Name = "admin"

print(obj.Name)

print(obj:Output())

```

