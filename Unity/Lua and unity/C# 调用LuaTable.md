C#

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using XLua;

public delegate void OneStringParams(string name);
public delegate void TransSelf(LuaTable table);
//针对结构体调用后添加
public delegate string OneStringReturn(string name);
[CSharpCallLua]
public delegate void TransMy(LuaCore table);

//Lua映射table过来，需要有个结构题进行对应
//lua的table导出到C#的结构体，可以实现C#运行时无GC
[GCOptimize]
public struct LuaCore
{
    public int ID;
    public string Name;
    public bool IsWoman;

    public OneStringParams Func1;
    public OneStringReturn Func2;
    public TransMy Func3;
    public TransMy Func4;
}
public class CsharpCallTable : MonoBehaviour
{
    private void Start()
    {
        xLuaEnv.Instance.DoString("return require('L2C/CsharpCallTable')");

    }

    public void UseLuaTable()
    {
        LuaTable g = xLuaEnv.Instance.Global;

        //获取的是全局变量Core，因为它在lua中是表，所以取出的是luatable
        LuaTable core = g.Get<LuaTable>("Core");
        //获取Name
        //参数：table中索引名
        //类型：索引对应值的类型
        Debug.Log(core.Get<string>("Name"));

        core.Set<string, string>("Name", "admin");

        OneStringParams osp = core.Get<OneStringParams>("Func1");
        osp("admin");

        //相当于“：”调用
        TransSelf ts = core.Get<TransSelf>("Func4");
        ts(core);
    }

    public void UseStruct()
    {
        LuaTable g = xLuaEnv.Instance.Global;

        //将lua的table导出为core
        LuaCore core = g.Get<LuaCore>("Core");

        Debug.Log(core.Name);
        core.Func4(core);
    }
    private void OnDestroy()
    {
        xLuaEnv.Instance.Free();
    }
}

```

Lua

```
Core = {}

Core.ID = 100
Core.Name = "root"
Core.IsWoman = false

Core.Func1 = function ( name )
	print("这是Core表的Func1函数，接受到C#数据".. name)
end

Core.Func2 = function ()
	return "这是Core表的Func2函数"
end

Core.Func3 = function ( self )
	print("这是Core表的Func3函数，Core表的成员变量Name是".. self.Name)
end

function Core:Func4()
	print("这是Core表的Func4函数，Core表的成员变量Name是".. self.Name)
end
```

