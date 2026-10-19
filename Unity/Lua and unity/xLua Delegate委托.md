C#

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public delegate void DelegateLua();

public class TestDelegate
{
    public static DelegateLua Static;

    public DelegateLua Dynamic;

    public static void StaticFunc()
    {
        Debug.Log("C#静态成员函数");
    }
}

public class LuaCallDelegate : MonoBehaviour
{
    void Start()
    {
        xLuaEnv.Instance.DoString("require('C2L/LuaCallDelegate')");
    }

    void OnDestroy()
    {
        xLuaEnv.Instance.Free();
    }
}

```

Lua

```
-- C#给委托赋值
-- TestDelegate.Static = TestDelegate.StaticFunc
-- TestDelegate.Static += TestDelegate.StaticFunc
-- TestDelegate.Static -= TestDelegate.StaticFunc
-- TestDelegate.Static()

CS.TestDelegate.Static = CS.TestDelegate.StaticFunc
CS.TestDelegate.Static()
-- Lua中如果添加了函数到静态委托变量中后，再委托不再使用后，记得释放添加的委托函数
CS.TestDelegate.Static = nil

----------------------------------------------------

local func = function()
	print("这是Lua的函数")
end

-- 覆盖添加委托
CS.TestDelegate.Static = func
-- 加减操作前一定要确定已经添加过回调函数
CS.TestDelegate.Static = CS.TestDelegate.Static + func
CS.TestDelegate.Static = CS.TestDelegate.Static - func
-- 调用以前应确定委托有值
CS.TestDelegate.Static()

CS.TestDelegate.Static = nil

-----------------------------------------------------------

-- 调用前判定
--	if(CS.TestDelegate.Static ~= nil)
--	then
--		CS.TestDelegate.Static()
--	end

-- 根据委托判定赋值方法
--	if(CS.TestDelegate.Static == nil)
--	then
--		CS.TestDelegate.Static = func
--	else
--		CS.TestDelegate.Static = CS.TestDelegate.Static + func
--	end

-----------------------------------------------------------

local obj = CS.TestDelegate()
obj.Dynamic = func
obj.Dynamic()

obj.Dynamic = nil


```

