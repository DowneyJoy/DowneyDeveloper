C#

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class TestOutRef
{
    public static string Func1()
    {
        return "Func1";
    }

    public static string Func2(string str1, out string str2)
    {
        str2 = "Func2 out";
        return "Func2";
    }

    public static string Func3(string str1, ref string str2)
    {
        str2 = "Func3 Ref";
        return "Func3";
    }

    public static string Func4(ref string str1, string str2)
    {
        str1 = "Func4 Ref";
        return "Func4";
    }
}

public class LuaCallOutRef : MonoBehaviour
{
    void Start()
    {
        xLuaEnv.Instance.DoString("require('C2L/LuaCallOutRef')");
    }

    void OnDestroy()
    {
        xLuaEnv.Instance.Free();
    }
}


```

Lua

```
local r1 = CS.TestOutRef.Func1()
print(r1)

-- C# out返回的变量，会赋值给Lua的第二个接受返回值变量
local out2
local r2, out1 = CS.TestOutRef.Func2("admin", out2)
print(r2, out1, out2)

-- C# ref返回的变量，会赋值给Lua的第二个接受返回值变量
local ref2
local r3, ref1 = CS.TestOutRef.Func3("root", ref2)
print(r3, ref1, ref2)

-- 即使out ref作为第一个参数，其结果依然会以Lua的多个返回值进行返回
local ref4
local r4, ref3 = CS.TestOutRef.Func4(ref4, "test")
print(r4, ref3, ref4)


```

