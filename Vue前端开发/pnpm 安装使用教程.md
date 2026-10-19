pnpm 是一种快速、节省磁盘空间的包管理工具，它是 npm 和 Yarn 的替代品，使用硬链接和符号链接来节省磁盘空间。其根据自身独特的包管理方法解决了 npm、yarn 内部潜在的安全及性能问题。在多数情况下拥有更快速的安装速度、占用更小的存储空间。本文主要介绍pnpm 的安装及使用。

## 1、Windows 安装pnpm 

在Windows 系统上可以通过npm或直接从PowerShell安装：

1）通过npm

```
npm install -g pnpm
```

2）使用PowerShell

```
iwr https://get.pnpm.io/install.ps1 -useb | iex
```

## 2、Linux 安装pnpm

在Linux 系统上可以通过npm或直接从shell脚本安装：

1）通过npm

```
npm install -g pnpm
```

2）使用shell脚本

```
curl -fsSL https://get.pnpm.io/install.sh | sh -
```

或者

```
wget -qO- https://get.pnpm.io/install.sh | sh -
```

## 3、使用pnpm

pnpm 安装完成后，可以开始使用它来安装包和管理项目。常用命令如下，

1）安装全部依赖

```
pnpm install 
```

2）安装指定包

```
pnpm add package-name
```

3）删除指定包

```
pnpm remove package-name
```

4）运行脚本

```
pnpm run script-name
```

5）升级依赖

```
pnpm update
```

6）全局安装包

```
pnpm add -g package-name
```

7）更新pnpm的版本

```
pnpm add -g pnpm to update
```

## 4、将npm或yarn项目改为pnpm项目

1）删除现有的 node_modules

pnpm 使用不同的结构来存储依赖项，所以最好先删除现有的 node_modules 目录

2）删除 package-lock.json 或 yarn.lock

如果存在，删除 package-lock.json 或 yarn.lock 文件，因为 pnpm 使用 pnpm-lock.yaml 来锁定依赖项版本

3）使用 pnpm 安装依赖

```
pnpm install
```

4）更新脚本和配置

如 package.json 中的脚本或其他工具的配置引用了 npm 或 yarn，将它们更新为 pnpm。

5）验证项目

```
pnpm test
```

## 5、清除pnpm缓存

要清除 pnpm 的缓存，可以使用 `pnpm store` 命令

1）清除整个缓存

移除所有不再引用的包

```
pnpm store prune
```

2）输出 pnpm 缓存的位置

通过输出的缓存位置，可以手动删除该目录

```
pnpm store path
```

3）验证缓存

```
pnpm store verify
```

命令会验证缓存的数据，并确保所有包都完好无损。如果发现任何损坏的包，它们将会被移除。