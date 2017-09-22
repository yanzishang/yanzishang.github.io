---
title: Java开发环境配置
date: 2017-09-21 17:38:16
tags: develop-env
categories: 开发环境
---

> 工欲善其事，必先利其器

这里只讨论在\*nix系统的Java开发环境配置，那我们开始吧！

#### Zsh——终极Shell

shell：\*nix系统的壳，用户与系统交互的桥梁。
在\*nix系统中，默认安装的是bash，它也是shell的一种。我们可以通过如下命令来查看系统安装了哪些shell。

```shell
cat /etc/shells
```

zsh：shell的一种，因为其配置的复杂，令人望而却步，所以没有bash普及。

##### 那么为什么还要介绍zsh？

因为oh-my-zsh的出现，它是为了解决zsh复杂的配置而生！  
oh-my-zsh地址：https://github.com/robbyrussell/oh-my-zsh

##### 那又为什么使用zsh？

zsh优点：  

- 兼容bash
- 补全
- 强大的Kill
- Alias
- 跳转
- 历史记录

更多特性可以点[这里](https://www-s.acm.illinois.edu/workshops/zsh/why.html)查看。

##### 开始定制zsh

首先需要安装zsh，Linux运行如下命令安装。

```shell
sudo yum install zsh
或者
sudo apt-get install zsh
```

Mac默认安装zsh

运行如下命令，自动安装oh-my-zsh，前提是安装了wget和git，下面有具体介绍。

```shell
sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```

oh-my-zsh的默认安装目录:~/.oh-my-zsh
zsh替换了bash，配置文件~/.zshrc替换了~/.bashrc  
这里主要讲一下，zsh的主题和插件。

###### 主题

编辑.zshrc文件，更改ZSH_THEME为你喜欢的主题，并保证.oh-my-zsh/themes目录下有此主题   
更多主题：https://github.com/robbyrussell/oh-my-zsh/wiki/Themes

###### 插件

在~/.oh-my-zsh/plugins目录下找到需要的插件，推荐插件：git、gradle、autojump、zsh-syntax-highlighting、maven
编辑~/.zshrc文件，添加插件目录名称到plugins中即可

*主题和插件操作完后，需要source ~/.zshrc，使修改立即生效*

#### Git——版本控制工具

说到git，那也要提一下svn，两者都是版本控制工具，对比一下：

git | svn 
----|------
分布式 | 集中式  
元数据存储 | 文件存储  
可断网 | 必须联网

简单对比，不做过多介绍！

##### 开始定制git

首先安装git，Linux用过yum或者apt来安装git，mac可以通过[homebrew](https://brew.sh/)安装。

配置git的user.name和user.email，在提交代码的时候会有对应的提交者
通过如下命令配置，将xxx换成你的name或email

```shell
git config --global user.name xxx
git config --global user.email xxx
```

通过配置oh-my-zsh的git插件，可以实现git命令补全、git别名。

#### JDK——Java开发环境

既然干java的，那么jdk就不多说了，直接开搞！

Linux上去官网下载对应平台的gz包，解压到/usr/local/jdk。
##### 配置环境变量

使用vim编辑/etc/profile，在文件末尾添加：

```shell
export JAVA_HOME=/usr/local/jdk
export PATH=$PATH:$JAVA_HOME/bin
```
保存退出，然后执行如下命令使修改立即生效

```shell
source /etc/profile
```
Mac可以去官网直接下载dmg安装，也可以通过[homebrew](https://brew.sh/)安装。

现在你可以通过javac来编译.java文件，通过java来运行.class文件了！

#### 未完占坑
