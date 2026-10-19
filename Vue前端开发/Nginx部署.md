## 1.准备工作

- 完整的Vue项目
- 一台服务器

准备工作很简单，只需要一个项目和一台服务器即可，项目甚至可以只是一个html文件。

## 2.初始化Vue项目

使用Vue-cli脚手架初始化一个最基本的Vue项目，命令如下：

```js
vue create myproject
```

我们初始化一个Vue2.0的项目，选好配置后一路回车：

![img](https://pic3.zhimg.com/80/v2-8ac8fd7525dc6348fb769b62cf86213e_720w.webp)

运行命令：

```js
npm run serve
```

项目运行成功：

![img](https://pic1.zhimg.com/80/v2-185d2a08d87933e6967875123b1665c0_720w.webp)

## 3.服务器安装Nginx

Nginx是一款轻量级的Web服务器、反向代理服务器，由于它的内存占用少，启动极快，高并发能力强，在互联网项目中广泛应用，我们的网站部署到Nginx服务器上。当然也可以选择其它web服务器，这里我们就使用主流的Nginx。

我这里的服务器是Centos系统，故已Centos为例。

**（1）先查看服务器是否有nginx**

命令如下：

```text
whereis nginx
```

如果出现如下界面，则代表未安装nginx，否则可跳过本节。

![img](https://pic4.zhimg.com/80/v2-dd04412943d7783d826445542c5299eb_720w.webp)

**（2）安装nginx**

命令如下：

```text
yum install -y nginx
```

如果未配置yum源的，可参考网上如何使用yum安装nginx，出现如下界面则代表安装完成。

![img](https://pic3.zhimg.com/80/v2-4eb2ade9aba107acb8a616dbe5fcb5fa_720w.webp)

**（3）查看nginx安装目录**

命令如下：

```text
whereis nginx
```

![img](https://pic1.zhimg.com/80/v2-715f23e2c9e860bf8d7deee28d50d560_720w.webp)

或者使用命令查看nginx版本号，出现版本号也代表安装成功。

输入命令：

```text
nginx -v
```

![img](https://pic4.zhimg.com/80/v2-613e8302aa14faf6abb35b7ceb19dbfb_720w.webp)

如果不能全局使用nginx指令，那么可能是环境变量没有配置，具体配置方法可参考网上，这里不做过多解释。

## 4.启动nginx

**（1）启动**

服务器安装号nginx后，我们尝试启动它、

命令如下：

```text
nginx
```

![img](https://pic4.zhimg.com/80/v2-db7e214922e03de04b80458fa986ee5b_720w.webp)

直接输入nginx即可启动服务，打开浏览器，访问服务器地址，出现如下界面则代表启动成功。

![img](https://pic2.zhimg.com/80/v2-55105008ebe0195451377513d70bef4d_720w.webp)

这里访问默认的是80端口，因为nginx服务就是默认的80端口，如果不能访问，大致有几种原因：

- nginx未启动
- 服务器安全组未加入80端口
- nginx配置错误

**（2）停止**

停止nginx服务，命令如下：

```text
nginx -s stop
```

此时访问浏览器则无法访问：

![img](https://pic4.zhimg.com/80/v2-97df692e290db7b8b1b28ea23351256f_720w.webp)

**（3）重启**

重启nginx服务，一般用于修改配置文件之后，命令如下：

```text
nginx -s reload
```

## 5.修改nginx配置

找到nginx配置文件存放位置，命令如下：

```text
whereis nginx
```

![img](https://pic4.zhimg.com/80/v2-b37def76b3314c9d7c617e34f454a26f_720w.webp)

此时/etc/nginx则是nginx配置文件存放位置，进入该文件夹：

![img](https://pic1.zhimg.com/80/v2-00648713bd3174b979585895b2ab7058_720w.webp)

nginx.conf则是默认配置文件，编辑该文件：

![img](https://pic2.zhimg.com/80/v2-108db02665ce78d8132f08fe99494865_720w.webp)

上面界面我们基本上不用改什么，重要的是server这个对象，可以看到默认监听的是80端口，所以我们直接访问服务器ip即可，接下来我们改为9000端口：

![img](https://pic2.zhimg.com/80/v2-91ccd3104093b0a841e166b09e89ee89_720w.webp)

重启nginx，命令如下：

```text
nginx -s reload
```

此时访问时则需要带上端口号（需要服务器安全组开启9000端口），访问地址：[http://ip:9000](https://link.zhihu.com/?target=http%3A//ip%3A9000)：

![img](https://pic2.zhimg.com/80/v2-f7c719cf436e385b4397983ca5734805_720w.webp)

我们还需要注意的一个地方就是root字段，该字段后面跟着的就是网站页面路径，也就是网站存放的路径。

假设我们网站的存放路径为：

```text
/home/www/dist
```

修改root，结果如下：

![img](https://pic3.zhimg.com/80/v2-a44038df5bb2a1e6660e3b9ade44c12e_720w.webp)

这个时候重启nginx，我们的网站无法访问了，因为我们没有这个文件夹。

![img](https://pic1.zhimg.com/80/v2-8091624ed4b69c019bcf53a8f4df1e00_720w.webp)

## 6.新建网站文件夹

刚刚我们设置了网站存放的文件夹路径为：

```text
/home/www/dist
```

此时我们需要在服务器上新建www文件夹：

```text
cd /home
mkdir www
```

![img](https://pic3.zhimg.com/80/v2-4c6b112c631022bed20181138c78b6da_720w.webp)

我们没有新建dist文件夹，因为我们待会儿vue项目打包就会生成dist文件夹。

## 7.打包部署vue项目

**（1）打包网站**

使用vue打包命令,生成dist文件夹：

```text
npm run build
```

**（2）上传至服务器**

我们可以使用ftp工具将dist文件夹上传至/home/www目录下，当然也可以使用命令：

```text
scp -r dist/ root@139.9.219.136:/home/www
```

![img](https://pic3.zhimg.com/80/v2-1fb4bec129abbf6162f23c0e1db78b02_720w.webp)

在我们的项目目录下用打开git命令行，将本地的dist整个文件夹复制到服务器，需要主要的时scp命令window不支持，所以使用了git命令行工具。

此时无需重启nginx，刷新浏览器，可以看到我们的网站已经部署成功了。

![img](https://pic4.zhimg.com/80/v2-baa84ef51d20fb0b2e3b5b7a201e08a7_720w.webp)

## 8.解决刷新路由404问题

当我们切换路由后，然后在刷新页面，会出现404的情况。

![img](https://pic4.zhimg.com/80/v2-a3447149902a9d91b2cc42879e16fb13_720w.webp)

这是因为我们vue项目采用了history的路由方式，至于原因主要是vue是单页面应用，细节部分大家可自行下来了解。

解决问题的办法：

（1）将路由模式改为hash

（2）修改nginx配置：

```text
location / {
  try_files $uri $uri/ /index.html; ---解决页面刷新404问题
}　

  (1)代理所有URI到后端服务器
    location / {
        proxy_pass http://192.168.49.142/;
    }
  (2)代理指定URL到后端服务器
    location /test1/ {
        proxy_pass http://192.168.49.142/test2/;
     将请求/test1下面的资源都转到后端/test2/下的资源
    location /test1 {
         proxy_pass http://192.168.49.142;
     将请求/test1/下面的资源都转到后端/test1/下的资源
```

![img](https://pic2.zhimg.com/80/v2-b7a4edb44ef9081d731bd82152e675f5_720w.webp)

修改之后重启nginx，这个时候刷新浏览器则不会出现404了。

## 总结

网站部署其实很简单，主要是需要了解nginx知识，懂了基本原理之后，部署任何网站都不会有问题的。