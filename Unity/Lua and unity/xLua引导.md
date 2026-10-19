学习目标

​	什么是热更新

​	为何要热更新

​	常见的Unity热更新插件有哪些

​	xLua的安装

​	xLua的基础API使用

 

课程知识点

什么是冷更新

​		开发者将测试好的代码，发布到应用商店的审核平台，平台方会进行稳定性及性能

​		测试。测试成功后，用户即可在AppStore看到应用的更新信息，用户点击应用更

​		新后，需要**先关闭应用**，**再进行更新**。

 

​	什么是热更新

​		广义：无需关闭应用，不停机状态下修复漏洞，更新资源等，重点是更新逻辑代码。

​		狭义定义（ iOS热更新）：无需将代码重新打包提交至AppStore，即可更新客户端

​		的执行代码，即不用下载app而自动更新程序。

​		现状：苹果禁止了C#的部分反射操作，禁止JIT（即时编译，程序运行时创建并运

​		行新代码），不允许逻辑热更新，只允许使用AssetBundle进行资源热更新。

​	

​		注意：2017年，苹果更新了热更新政策说明，上线后的项目，一旦发现使用热更

​		新，一样会以下架处理

 

​	为何要热更新

​		缩短用户获取新版应用的客户端的流程，改善用户体验

​		具体到iOS平台的应用上，有以下几个原因

​			AppStore的审核周期难控制

​			手机应用更新频繁

​			对于大型应用，更新成本太大

​		终极目标

​			不重新下载、不停机状态下完全变换一个应用的内容

 

​	每个平台如何做热更新

​		Android，PC（C#）

​			将执行代码预编译为AssemblyDLL

​			将代码作为TextAsset打包进AssetBundle

​			运行时调用AssemblyDLL代码

​			更新相应的AssetBundle即可实现热更新

​		iOS（Lua）

​			苹果官方禁止iOS下的程序热更新；JIT在iOS下无效

​			热更新方案：Unity + Lua插件

​			![img](file:///C:\Users\Tangning\AppData\Local\Temp\ksohtml8176\wps1.jpg)

 

​	常见的Unity热更新插件

​		sLua：最快的Lua插件

​		toLua：由uLua发展而来的，第三代Lua热更新方案

​		xLua：特性最先进的Lua插件

​		ILRuntime：纯C#实现的热更新插件

​		

![img](file:///C:\Users\Tangning\AppData\Local\Temp\ksohtml8176\wps2.jpg) 

 

xLua的安装

​		官网：https://github.com/Tencent/xLua

 

​	xLua解析器的获得

​	DoString函数执行Lua代码

​	Lua代码调用C#代码

​	C#加载Lua文件

​		![img](file:///C:\Users\Tangning\AppData\Local\Temp\ksohtml8176\wps3.jpg)

​		根据加载提示，发现可以将Lua文件放在StreamingAssets目录下

 

​		StreamingAssets被抛弃，所以移动到Resources下（以txt为结尾）

​		

​		自定义加载器LuaEnv.AddLoader()

​		![img](file:///C:\Users\Tangning\AppData\Local\Temp\ksohtml8176\wps4.jpg)

 

​		**接触****一个新的Lua项目时，****先要****弄懂Lua的加载器规则，只有这样，才能弄懂项目**

​		**的Lua执行流程**

​	xLua的单例运行环境

​		xLua解析器创建销毁

​		xLua加载器编写

​	xLua中Lua调用C#代码

​		C#实现的系统，因为Lua可以调用，所以完全可以换成lua实现，因为lua可以即时更改，即时运行，所以游戏的代码逻辑就可以随时修改

​		Lua调用Unity的各种Api，从而实现C#开发系统同样的效果

​	xLua中C#调用Lua代码

​		Unity是基于C#语言开发的，所有生命周期函数都是基于C#实现，xLua本身不存在Unity的相关生命周期函数。如果希望xLua能够拥有生命周期函数，那么我们可以实现C#作为Unity的原始调用，再使用C#调用Lua对应的方法

课后作业

 