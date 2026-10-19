学习目标

​	了解Unity编辑器的开发思路

​	检视器属性控制

​	检视器扩展显示开发

​	顶部菜单栏开发

​	学习弹窗开发

​	学习场景编辑器开发

 

课程知识点

​	插件了解

​		DOTween：UI动画

​		NGUI：场景UI制作

​		PlayMaker：图形化编写游戏逻辑工具

​	查看dll内部代码

​		百度：C# dll 反编译

​	什么是编辑器开发

​		对编辑器实现功能扩展，一般会使用它开发项目工具或实现Unity插件

​	C#特性

​		![img](file:///C:\Users\Tangning\AppData\Local\Temp\ksohtml2440\wps1.jpg)

​		用于在C#运行时，传递程序中各种元素（类，结构体，变量，方法，枚举，组件）

​		的行为信息的声明标签。一个声明标签是通过**放置****在****它所在应用元素的前面**的方

​		括号“[]”中来描述。

​	特殊目录

​		回顾

​			Plugins：需要跨语言调用的代码逻辑代码存储目录，手机SDK接入

​			Resources：存储跟随游戏包的资源目录

​			StreamingAssets：只读，存储跟随游戏包的资源目录

​		编辑器目录：Editor

​			制作多目录合并

​			项目中建立的Editor目录，编辑器相关的逻辑和资源会放在其内部，相关内容，

​			在打包生成时，不会一起生成到项目中，玩家也不会使用到编辑器相关的内容。

​			Editor目录下的脚本，无法挂载在场景对象下

​	命名空间

​		Unity代码逻辑命名空间：UnityEngine

​		Unity编辑器命名空间：UnityEditor，此命名空间，不要出现在游戏被发布的逻辑代码中，会导致项目打包失败！

​		![img](file:///C:\Users\Tangning\AppData\Local\Temp\ksohtml2440\wps2.jpg)

​	检视器属性控制

​		参考：SimpleInspector.cs脚本

​	检视器高级显示控制

​		参考：PlayerEditor脚本

​	添加组件

​		![img](file:///C:\Users\Tangning\AppData\Local\Temp\ksohtml2440\wps3.jpg)

​		

![img](file:///C:\Users\Tangning\AppData\Local\Temp\ksohtml2440\wps4.jpg) 

检视器高级修改（外挂式编程）

​	项目中建立Editor目录

​	Editor目录下建立外挂式开发脚本

​	将编辑器脚本与原始脚本关联

​	创建菜单

​		参考TestMenu脚本

​	创建窗口

​		TestWindow脚本

​	编辑器工具实现分析

​		场景布置，Tool场景

​		点击点预制体制作

​		脚本挂载Plane，存储点击点，Update绘制点位增加时的两个点之间的线

​		组件扩展类NodeManagerEditor

​			OnInspectorGUI绘制列表和按钮

​			OnSceneGUI实现点击场景后，位置点生成

​				位置点由HandleUtility.GUIPointToWorldRay发出射线计算

​			问题，创建新的对象后，编辑状态会被取消，原因，新创建的对象，会占据Inspector面板，此问题有弹窗的Update，一直选中对象

​			删除按钮实现NodeManager下List内数据删除

​				Destroy会等当前帧执行完成后，删除GameObject

​				DestroyImmediate会直接删除对象，不安全，有可能后续有代码处理逻辑

​			编辑器下，对象没有脚本，生命周期函数不会运行，可以直接使用第二个API删除对象和列表中的数据

​		NodeWindow窗口实现

​			创建窗口时，需要将Plane对象传递过来，保证Update可以一直选中Plane

​			Update()实现一直选中Plane物体

​			

异或位运算

​	相同为零，不同为一

​	A与B异或得到C，那么

​		B与C异或，可再次获得A

​		A与C异或，可再次获得B

​	所以如果A是明文，B是密钥，C就是密文，使用异或实现的算法，可以将C密文利用B密钥，解出A即原始文字

 

