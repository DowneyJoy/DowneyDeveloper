C#

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public enum TestEnum
{
    LoL = 0,

    Dota2
}

public class LuaCallEnum : MonoBehaviour
{
    void Start()
    {
        xLuaEnv.Instance.DoString("require('C2L/LuaCallEnum')");
    }

    void OnDestroy()
    {
        xLuaEnv.Instance.Free();
    }
}

```

Lua

```
-- C# TestEnum.LoL
-- CS.命名空间.枚举名.枚举值
-- 枚举获得是userdata自定义数据类型，获得其他语言数据类型时，就是userdata
print(type(CS.TestEnum.LoL))
print(CS.TestEnum.Dota2)

-- 转换获得枚举值
print(CS.TestEnum.__CastFrom(0))
print(CS.TestEnum.__CastFrom("Dota2"))

```

