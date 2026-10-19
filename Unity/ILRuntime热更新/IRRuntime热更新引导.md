ILRuntime官网：

http://ourpalm.github.io/ILRuntime/public/v1/guide/index.html

## 基础

### 原理

- ILRuntime借助Mono.Cecil库来读取DLL的PE信息，以及当中类型的所有信息，最终得到方法的IL汇编码，然后通过内置的IL解译执行虚拟机来执行DLL中的代码。 ILRuntime目标是读取热更新的dll，编译成IL，然后使用自己的JIT Compiler来执行热更dll中的代码,达到热更的目的。
- ILRuntime自己实现了一些类型和栈的操作，以此减少装箱拆箱的次数。

### 优势

- 无缝访问C#工程的现成代码，无需额外抽象脚本API
- 直接使用VS2015进行开发，ILRuntime的解译引擎支持.Net 4.6编译的DLL
- 执行效率是L#的10-20倍
- 选择性的CLR绑定使跨域调用更快速，绑定后跨域调用的性能能达到slua的2倍左右（从脚本调用GameObject之类的接口）
- 支持跨域继承
- 完整的泛型支持
- 拥有Visual Studio的调试插件，可以实现真机源码级调试。支持Visual Studio 2015 Update3 以及Visual Studio 2017和Visual Studio 2019
- 最新的2.0版引入的寄存器模式将数学运算性能进行了大幅优化

### C# vs Lua

- 用Lua热更需要对Lua和C#都比较熟悉，Lua优势在于足够成熟。
- C#直接使用更加便捷



Lua是一个已经非常成熟的解决方案，但是对于Unity项目而言，也有非常明显的缺点。就是如果使用Lua来进行逻辑开发，就势必要求团队当中的人员需要同时对Lua和C#都特别熟悉，或者将团队中的人员分成C#小组和Lua小组。不管哪一种方案，对于中小型团队都是非常痛苦的一件事情。

用C#来作为热更语言最大的优势就是项目可以用同一个语言来进行开发，对Unity项目而言，这种方式肯定是开发效率最高的。

Lua的优势在于解决方案足够成熟，之前的C++团队可能比起C#，更加习惯使用Lua来进行逻辑开发。此外借助luajit，在某些情况下的执行效率会非常不错，但是luajit现在维护情况也不容乐观，官方还是推荐使用公版Lua来开发。

#### 如果需要测试ILRuntime对比Lua的性能Benchmark，需要确认以下几点：

- ILRuntime加载的dll文件是Release模式编译的
- dll中对外部API的调用都进行了CLR绑定
- 确保没有勾选Development Build的情况下发布成正式真机运行包，而不是在Editor中直接运行
- 可以直接使用Demo工程中提供的性能测试进行对比

#### ILRuntime设计上为了在开发时提供更多的调试支持，在Unity Editor中运行会有很多额外的性能开销，

#### 因此在Unity Editor中直接测试并不能代表ILRuntime的实际运行性能。

最新2.0版本的ILRuntime，加入了寄存器模式，在10多项测试用例当中的性能，均已超过lua53版xlua，详细测试代码可参见ILRuntime的U3D Demo工程以及视频教程



### 环境配置

#### 热更新原理

![image-20240122120632578](C:\Users\Tangning\AppData\Roaming\Typora\typora-user-images\image-20240122120632578.png)



#### 项目配置步骤

- ###### step1:打开Window-->package Manager ---->安装ILRuntime（并导入sample或者导入github上的测试项目：https://github.com/Ourpalm/ILRuntimeU3D/）

- ###### step2：开启PlaerSetting unsafecode模式

- ###### step3：安装好推荐版本.net4.6 (4.7版本须在配置中修改)

- ###### step4：配置.net版本（demo中：HotFix_Project~/HotFix_Project.csproj文件内---><TargetFrameworkVersion>v4.7.1</TargetFrameworkVersion>）

  - ​	Hotfix-project：所有的业务逻辑都是基于这个工程来开发，然后使用这个项目编译成.dll
  - ​	Unity+ILRuntime来加载，解释执行我们的业务逻辑的.dll

- ###### step5：测试ILRuntime基本环境：打开测试scene：Helloword，点击运行查看Log面板是否打印成功（!!! InstanceClass.StaticFunTest()）

###### 

#### 总结：

1. ###### Unity部分ILRuntime虚拟机+引擎核心代码=框架代码（不能做热更新）；加载我们的.dll文件；ILRuntime解释执行里面的代码

2. ###### ILRuntime业务逻辑项目工程：.net项目可以调用UnityEngine.dll代码；Assembly-Csharp.dll（调用底层框架Csharp代码）



### 基于ILRuntime框架的基本原则

1. 我们的场景不放任何内容，场景是空的，（方便维护以及协作；美术模型策划有编辑资源的需要，自己创建场景）资源、视图、地图场景；角色美术制作角色模型预制体；策划：关卡编辑器（关卡+数据）；程序，策划，美术分离；运行的时候场景是空的，程序使用代码创建出来；
2. 运行的时候场景都是空的，没有必要切换场景，运行游戏的一个容器，运行的时候只有一个场景；
3. 代码new出来，代码加载资源；资源管理：基于AssetBundle资源管理，不使用Resources资源；ab包放在本地或者服务器，方便热更新，ab包放StreamingAssets开发的时候，你资源变动，你要打ab包；开发模式：AssetsDataBase模式；发布模式：ab包的模式；ResMgr：开始模式，发布模式
4. 业务逻辑代码全部写到ILRuntime的热更新项目里；
5. 尽量业务逻辑代码不继承Unity项目里面类（MonoBehaviour，自己Unity创建的C#类）



首先需要在项目的Packages/manifest.json中，添加ILRuntime的源信息，在这个文件的dependencies节点前增加以下代码



```
"scopedRegistries": [
  {
    "name": "ILRuntime",
    "url": "https://registry.npmjs.org",
    "scopes": [
      "com.ourpalm"
    ]
  }
],
```

部分Unity版本可以无法直接在列表中刷出ILRuntime，如果左边列表找不着，那就在项目的manifest.json中的dependencies段的开头，增加如下代码手动将ILRuntime添加进项目

```
"com.ourpalm.ilruntime": "1.6.0",
```



### 搭建基于ILRuntime的框架

搭建基于ILRuntime的框架，把逻辑代码和核心代码分离开来

Unity项目部分：启动代码+资源管理（ab包，ResMgr，ILRumtimeWrapper）+其他的管理（网络管理）

ILRuntime热更项目：.net,.dll，输出生成路径，热更项目我们的代码结构



### ILRuntime代码调用C#核心代码

直接调用就可以了，gameobject里面获取我们的组件实例，gameobject.Getcomponent<Transform>();

不要继承他

如果你自己写的代码在我们unity项目工程里面C#代码，可以直接调用



### 基于ILRuntime热更新

热更我们的dll；远程下载dll；

判断版本，更新；

下载完成后，从下载路径下加载

如果没有下载路径，你在从streamingAssets加载

dll二进制，也可以ab包一起打包
