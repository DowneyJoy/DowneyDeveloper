## 目录说明

![](E:\Notes\study-notes\程序学习笔记\ImageContent\page.png)

## Uniapp和原生小程序开发区别

每个页面是一个 .vue 文件，数据绑定及事件处理同 Vue.js 规范：

1. 属性绑定 

​		src="{{ url }}" 升级成 :src="url"

2. 事件绑定 

​		bindtap="eventName" 升级成 @tap="eventName"，支持（）传参

3. 支持 Vue 常用指令 v-for、 v-if、v-show、v-model 等

温馨提示：调用接口能力，**建议**前缀 wx 替换为 uni ，养成好习惯，这样支持多端开发。

![](E:\Notes\study-notes\程序学习笔记\ImageContent\微信截图_20240205093649.png)

## 命令行创建uni-app项目

**uni-app 支持两种方式创建项目：**

• 通过HBuilderX 创建

• 通过命令行创建

**命令行创建 uni-app 项目：**

vue3+ts版: npx degit dcloudio/uni-preset-vue#vite-ts 项目名称

官网链接：https://uniapp.dcloud.net.cn/quickstart-cli.html#创建uni-app

![](E:\Notes\study-notes\程序学习笔记\ImageContent\创建uni-app.png)

**编译和运行 uni-app 项目：**

- [ ] npx degit xxx 创建项目
- [ ] manifest.json 添加 appid 
- [ ] pnpm install 
- [ ] pnpm dev:mp-weixin 
- [ ] 导入微信开发者工具

## 用 VS Code 开发 uni-app 项目

**为什么选择 VS Code ？**

⚫ HbuilderX 对 TS 类型支持暂不完善

⚫ VS Code 对 TS 类型支持友好，熟悉的编辑器

1. 创建 uni-app 项目，用 Vue 开发项目
2. 安装Vue项目依赖编译成小程序代码 
3. 开启服务端口 
4. 预览、调试

![](E:\Notes\study-notes\程序学习笔记\ImageContent\uniappVSCode.png)

