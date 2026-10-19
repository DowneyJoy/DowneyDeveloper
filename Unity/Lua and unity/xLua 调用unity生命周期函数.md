C#

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using XLua;

public delegate void LifeCycle();

//将Lua与Lua核心表对应的结构体
[GCOptimize]
public struct LuaBootstrap
{
    public LifeCycle Start;
    public LifeCycle Update;
    public LifeCycle OnDestroy;
}

public class Bootstrap : MonoBehaviour
{
    //Lua的核心table
    private LuaBootstrap _Bootstrap;

    //调用起Lua代码
    void Start()
    {
        //防止切换场景时，脚本对象丢失
        DontDestroyOnLoad(gameObject);

        xLuaEnv.Instance.DoString("require('Bootstrap')");

        //将Lua的核心table，导入到C#端
        _Bootstrap = xLuaEnv.Instance.Global.Get<LuaBootstrap>("Bootstrap");
        if(_Bootstrap.Start != null)
            _Bootstrap.Start();
    }

    void Update()
    {
        if (_Bootstrap.Update != null)
            _Bootstrap.Update();
    }

    private void OnDisable()
    {
        if (_Bootstrap.OnDestroy != null)
            _Bootstrap.OnDestroy();

        //释放Lua环境前，需要将导出到C#的Lua回调函数进行释放
        _Bootstrap.Start = null;
        _Bootstrap.Update = null;
        _Bootstrap.OnDestroy = null;
    }
    //释放Lua的代码
    void OnDestroy()
    {

        xLuaEnv.Instance.Free();
    }
}

```

Lua

```
Bootstrap = {}

--核心table，存储所有的控制器
Bootstrap.Controllers = {}

Bootstrap.Start = function()
	Bootstrap.LoadPage("MainMenuController")
end

Bootstrap.Update = function()
	--遍历所有注册给核心的控制脚本
	for k, v in pairs(Bootstrap.Controllers)
	do
		if(v.Update ~= nil)
		then
			--调用控制器的Update
			v:Update()
		end
	end
end

Bootstrap.OnDestroy = function()
	print("Lua OnDestroy")
end

-- 提供一个核心table函数用于加载页面控制器
-- 参数是脚本的名称
Bootstrap.LoadPage = function(name)
	--加载Controller目录下对应的脚本
	local c = require("Controller/" .. name)
	--注册给核心的table，这样生命周期函数就会正常调用
	Bootstrap.Controllers[name] = c
	--加载页面时，同时运行Start
	c:Start()
end

require("Config")
require("ABManager")
require("Prefabs")
LuaJson = require("LuaJson")

--LuaJson.encode()
--LuaJson.decode()

```

