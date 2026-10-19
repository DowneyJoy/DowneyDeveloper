AssetBundle

课程重点

​	了解AssetBundle原理

​	了解创建AssetBundle的方法

​	学习加载AssetBundle内部资源

​	学习AssetBundle整合到项目中的方法

 

课程知识点

​	原理

​		Resources原理

​			项目中建立Resources目录，资源导入内部

​			![img](file:///C:\Users\Tangning\AppData\Local\Temp\ksohtml29756\wps1.jpg)

​			生成项目包

​			![img](file:///C:\Users\Tangning\AppData\Local\Temp\ksohtml29756\wps2.jpg)

​			资源文件存储路径

​			![img](file:///C:\Users\Tangning\AppData\Local\Temp\ksohtml29756\wps3.jpg)

​			结论：存储在Resources下的资源，最终会存储在游戏的主体包中，发送给用

户，手机系统上，如果需要做资源的更新，是无法使用Resources即时更新（不

需要关闭手机游戏，即可实现游戏更新）

 

崩坏3（应用商店重新下载）

王者荣耀（游戏内直接更新）

 

​		AssetBundle（简称AB包）

​			![img](file:///C:\Users\Tangning\AppData\Local\Temp\ksohtml29756\wps4.jpg)

​			AB包是**独立于****游戏主包存在的资源存储文件**，使用内部资源时，需要单独下

​			载和加载。

​		

​		AB包和Resources的区别

​			存储

​				Resources内部资源存储在游戏的发布包中

​				AB包存储在独立的文件中（AB包存储在非特殊目录下时，不在游戏的发

​				布包中）

​			加载

​				Resources内部资源使用Resources.Load()

​				AB包（可以理解为可下载的Resources）

​					*获得AB包文件（下载，解压streaming拷贝到可写目录）

​					加载步骤1：通过AB包文件路径，加载AB包文件

​					加载步骤2：通过名称（资源名称），加载内部资源

​	AssetBundle的定义

​		AssetBundle是把一些资源文件，场景文件或二进制文件以某种紧密的方式保存在

​		一起的一些文件。

​		AssetBundle内部不能包含C#脚本文件，AssetBundle可以配合Lua实现资源和游戏

​		逻辑代码的更新

​	

​	热更新

​		不关闭Unity应用的前提，实现游戏资源和代码逻辑的更新。

 

​	AB包的创建

​		资源导入项目

​			如果资源未来做更新（资源存储在AB包中），就不要将资源放置在Resources

目录下

​		资源配置

​			![img](file:///C:\Users\Tangning\AppData\Local\Temp\ksohtml29756\wps5.jpg)

​			选中需要添加AB包的资源，填写或选择已存在的AB包名称

 

​			**AB包配置修改后****或****AB内部的资源修改后，都需要****重新****生成AB包**

​			

​			AB包名称如果配置为这样的结构”ui/package”，ui会作为AB包存储的父目录，

​			package是AB包的名称

 

​			导入到AB包的资源，包内部是不存在目录的

 

​		生成AB包脚本编写（编辑器脚本）

 

​		AB包压缩方式（实际选择根据项目情况分析）

​			不压缩：AB包比较大，下载较慢，加载速度快（CPU不用运算解压缩）

​			LZMA算法压缩：**默认**压缩模式，文件尺寸居中，加载速度居中

​			LZ4算法压缩：5.3以后可用，压缩比例高，文件小，加载速度偏慢

​			![img](file:///C:\Users\Tangning\AppData\Local\Temp\ksohtml29756\wps6.jpg)

​		依赖关系

​			如果一个AB（名称A）包，使用到了另一个AB（名称B）包的资源，那么两

个AB包就产生了依赖关系。也就是A依赖于B。

​			![img](file:///C:\Users\Tangning\AppData\Local\Temp\ksohtml29756\wps7.jpg)

​			Package预制体，依赖于背包素材所在的AB包

​			主manifest文件中，存储了依赖关系

​			![img](file:///C:\Users\Tangning\AppData\Local\Temp\ksohtml29756\wps8.jpg)

 

​		编写顶部菜单栏按钮

​			AB包分平台，所以导出的API不同

​					Android

​					iOS

​					Windows

​					Mac

​				BuildPipeline.BuildAssetBundles

​					AB存储路径

​					导出选项

​						None：无任何选项

UncompressedAssetBundle：默认压缩模式为LZMA，开启这个选项就不会压缩AB包

CollectDependencies：默认开启，开启依赖记录机制

​	![img](file:///C:\Users\Tangning\AppData\Local\Temp\ksohtml29756\wps9.jpg)

​	主Manifest记录了依赖关系

DeterministicAssetBundle：将AssetBundle的哈希校验值，存储在ID中（默认开启）

ForceRebuildAssetBundle：强制重新导出所有的AB包

ChunkBasedCompression：使用LZ4算法压缩AB包

​					平台

​				导出后的目录结构

​					![img](file:///C:\Users\Tangning\AppData\Local\Temp\ksohtml29756\wps10.jpg)

​					和存储目录同名的文件和文件.manifest，是主AB包及主AB包配置

​					文件

 

​					**无****扩展名文件：AB包**

​					**文件****名.manifest文件：****AB****包对应的配置文件**

​	加载AB包内部数据

​		如果想处理依赖关系的加载，则必须加载主AB包，因为依赖关系的存储，都存储

​		在主AB包的配置文件中

​		第一步（加载依赖的AB包文件）

1.加载主AB包

2.根据主AB包的配置文件，获得我当前需要加载的AB所依赖的AB们

3.将所有的依赖AB们，加载进来

​		第二步（加载AB包文件）

​			AB包 = AssetBundle.LoadFromFile(AB包文件路径)

​			AssetBundle.LoadFromFileSync(AB包文件路径)

​		第三步（加载AB包内部资源）

​			资源对象 = AB包对象.LoadAsset<资源类型>(“资源名称”)

AB包对象.LoadAssetSync<资源类型>(“资源名称”)

 

​		注意！！！：AB包不能重复加载

​	

​	Unity性能分析工具

​		![img](file:///C:\Users\Tangning\AppData\Local\Temp\ksohtml29756\wps11.jpg)

​	

​		内存分析

​			![img](file:///C:\Users\Tangning\AppData\Local\Temp\ksohtml29756\wps12.jpg)

​		Resources.Load()加载后的资源，Unity会存储在内存中，所以即使场景中的对象被

删除，资源的内存占用依然存在，只用使用Resources.UnloadUnusedAssets()释放后，

没有被场景引用的资源内存才会被释放

 

![img](file:///C:\Users\Tangning\AppData\Local\Temp\ksohtml29756\wps13.jpg) 

课后作业

​	请实现一个AB管理器

内部采用字典实现AB包的加载，并防止同名AB包被加载

根据名称可以卸载AB包（卸载文件引用，完全卸载）

​		依赖关系的处理