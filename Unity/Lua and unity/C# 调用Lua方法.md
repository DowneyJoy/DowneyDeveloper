C#

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using XLua;

public delegate void Func1();
public delegate void Func2(string name);
public delegate string Func3();

[CSharpCallLua]//映射产生时，xLua提示添加
public delegate void Func4(out string name,out int id);

public class CsharpCallFunction : MonoBehaviour
{
    private void Start()
    {
        xLuaEnv.Instance.DoString("return require('L2C/CsharpCallFunction')");

        LuaTable g = xLuaEnv.Instance.Global;

        //lua的函数，会导出为C#的委托类型
        Func1 func1 = g.Get<Func1>("func1");
        func1();
        
        //向lua函数传递数据
        Func2 func2 = g.Get<Func2>("func2");
        func2("fancy");

        //接收lua函数的返回值
        Func3 func3 = g.Get<Func3>("func3");
        func3();

        //lua多返回值
        Func4 func4 = g.Get<Func4>("func4");
        string name;
        int id;
        func4(out name,out id);
    }

    private void OnDestroy()
    {
        xLuaEnv.Instance.Free();
    }
}

```

Lua

```
func1 = function()
	print("这是Lua中的func1")
end

func2 = function(name)
	print("这是Lua中的func2,参数是："..name)
end

func3 = function()
	return "这是Lua中的func3"
end

func1 = function()
	return "这是Lua中的func4",100
end

```

