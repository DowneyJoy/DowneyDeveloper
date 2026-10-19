C#

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

namespace HX
{
    public static class TestStatic
    {
        public static int ID = 99;

        public static string Name
        {
            get;
            set;
        }

        public static string Output()
        {
            return "static";
        }

        public static void Default(string str = "abc")
        {
            Debug.Log(str);
        }
    }
}

public class LuaCallStatic : MonoBehaviour
{
    void Start()
    {
        xLuaEnv.Instance.DoString("require('C2L/LuaCallStatic')");
    }

    private void OnDestroy()
    {
        xLuaEnv.Instance.Free();
    }
}

```

Lua

```
-- Lua调用静态类

-- 规则"CS.命名空间.类名.成员变量"
print(CS.HX.TestStatic.ID)

-- 给静态属性赋值
CS.HX.TestStatic.Name = "admin"
print(CS.HX.TestStatic.Name)

-- 静态成员方法调用
-- 规则"CS.命名空间.类名.方法名()"
print(CS.HX.TestStatic.Output())

-- 使用默认值
CS.HX.TestStatic.Default()
-- 使用Lua传递的值
CS.HX.TestStatic.Default("def")

```

