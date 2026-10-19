### Unity IL2CPP技术原理和AOT

IL是什么？

Intermediate Language它是一种属于通用语言架构和.NET框架的低阶（lowest-level）的编程语言

CLI是什么？

Common Language Infrastructure特指在.Net平台下的IL标准，CLI类似一个面向对象的汇编语言，并且它是完全属于堆栈的，它运行在.net虚拟机上

IL2CPP项目：IL字节码-----静态的编译型语言（C、C++）的代码，IL----C++代码

AOT：Ahead of time离线编译，就是把高级语言----静态编译型语言---直接编译成二进制机器指令

![image-20240508101633501](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20240508101633501.png)



### huatuo热更新原理

它是基于IL2CPP，和其他的热更新技术：Lua内置Lua虚拟机+Lua代码；ILRuntime方案：内置C#虚拟机+解释执行ILRuntime

内置虚拟机：自己解释执行的一个运行环境，无法直接继承MonoBehaviour（IL2CPP级别的数据对象），需要封装一层

​		跨域访问，接口导出，需要开发者自己处理，不符合我们标准的unity开发

IL2CPP runtime环境（IL2CPP VM）编写了一个解释器，解释执行IL代码指令+使用的是AOT的数据内存对象



热更项目里，我们可以随意的继承使用我们的GameObject，MonoBehaviour

数据对象：在编译AOT的时候，热更项目的类型编译进去就可以了

解释执行IL new GameObject——————new AOT的GameObject对象



热更IL.dll文件，加载文件来解释执行



### huatuo的性能高及优势

华佗直接使用的时候我们的AOT项目中的内存对象，内存的占用，跨域都没有什么问题，性能好，内存占用少

不改变我们的上层开发习惯，内置虚拟机

Lua，ILRuntime热更新1.0-2.0   2.0apk也是解释执行

huatuo 1.0可以解释执行，2.0apk直接AOT执行，新版本更好的性能

使用普通的Unity模式开发就可以了



### huatuo热更开发环境搭建与测试

