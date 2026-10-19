# Hbulider 运行微信开发者工具报错



使用HBuilder X 编写 uniapp ，点击运行到微信小程序，微信开发者工具未运行打开，在控制台报错。

报错信息：[微信小程序开发者工具] ? Enable IDE Service (y/N) [27D[27C
[error] 工具的服务端口已关闭。要使用命令行调用工具，请在下方输入 y 以确认开启，或手动打开工具 -> 设置 -> 安全设置，将服务端口开启。

![image-20231120101721806](C:\Users\Tangning\AppData\Roaming\Typora\typora-user-images\image-20231120101721806.png)

第一步：按照提示操作输入Y，可是一直输入不进去，不可行。

第二步：按照提示尝试微信开发者工具设置服务器端口。

![image-20231120101807361](C:\Users\Tangning\AppData\Roaming\Typora\typora-user-images\image-20231120101807361.png)

打开服务端口，返回HBuilder X重新运行编译即可成功

![image-20231120101819020](C:\Users\Tangning\AppData\Roaming\Typora\typora-user-images\image-20231120101819020.png)

![image-20231120101848007](C:\Users\Tangning\AppData\Roaming\Typora\typora-user-images\image-20231120101848007.png)