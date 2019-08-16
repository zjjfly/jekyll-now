---
layout: post
title: Haskell开发环境搭建
published: true
tags: Haskell
---
## 安装
首先安装Stack，一个Haskell的构建构建，类似Java的Gradle。Mac直接使用brew安装：

```sh
brew install stack
```

接着，需要安装Haskell的编译器。可以使用stack来安装

```sh
stack setup
```

然后，使用stack安装indent和stylish：

```sh
stack install hindent
stack install stylish
```

IDE我选择Intellij，所以要先安装**Intellij-Haskell**插件，然后在IDEA的设置的**Other Setting**->**Haskell**中设置hindent和stylish的可执行文件的路径，Mac中是在`~/.local/bin/`中，最好把这个目录也加到PATH中。![haskell-1-3.png]({{site.baseurl}}/images/20170805/haskell-1-3.png)
## 第一个Haskell项目

使用stack初始化一个项目：

```sh
stack new FirstApp
```

然后在IDEA中导入项目，点击在工具栏的**File**->**New**->**Project from Existing Sources**。选择新建的项目的路径，点击**open**，然后出来这个界面。![haskell-1-1.png]({{site.baseurl}}/images/20170805/haskell-1-1.png)
选择**Import project from external model**->**Haskell Stack**，然后点击**Next**。
![haskell-1-2.png]({{site.baseurl}}/images/20170805/haskell-1-2.png)
配置SDK,实际上只要选择`stack`这个命令所在的路径就可以，如果使用brew安装的stack，路径是`/usr/local/bin/stack`。最后点击**Finish**,导入就完成了。
## 运行
首先，编译项目。在IDEA中打开终端(默认会进入项目的根目录)，使用命令：

```sh
stack build
```

然后,进入GHC的REPL：

```sh
stack repl
```

stack会把自动加载项目。然后在REPL中输入：

```sh
main
```

会输出
> someFunc

至此，搭建工作顺利完成。
