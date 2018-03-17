---
title: 使用coding进行项目代码管理（全程可视化操作！）
date: 2016-08-27
tags: [Git, Coding, GitGui]
categories: 相关技能
---
## 第1步 下载并安装git
下载地址：https://git-scm.com/  
下载及安装过程略。

## 第2步 注册coding帐号
coding官网：https://coding.net   
注册完毕之后，需要到邮箱点击激活邮件中的链接验证邮箱。  
具体过程略。  

## 第3步 在coding中创建一个项目
在如下界面创建项目。

![创建项目](http://img.blog.csdn.net/20160827133222938)

创建完成之后，会得到一个git地址，点击右面的图标可以复制地址，如下：  

![git地址](http://img.blog.csdn.net/20160827133437807)

项目创建完成之后，可以添加项目成员，被添加的成员都可以访问到你的项目。

## 第4步 在本地创建仓库
安装完git后，右键菜单会多出两个条目，如图：

![GIT GUI](http://img.blog.csdn.net/20160827134032176)

选择Git GUI Here，调出git工具的可视化窗口。

![create new repository](http://img.blog.csdn.net/20160827134134270)

指定创建的文件夹名称。（这个文件夹必须不存在，之后会说如何将一个已经存在的文件夹链接到远程仓库）

![填写文件夹名字](http://img.blog.csdn.net/20160827134355677)

点击“Create”之后，git会在你指定的目录下创建一个文件夹，里面有一个.git的隐藏文件夹。  
进入到创建的文件夹中，将你的代码或其他文件放进来。

## 第5步 链接到远程仓库
我们在Git GUI中点击“Remote”菜单下的“Add”选项，进入如下界面。

![填写远程仓库信息](http://img.blog.csdn.net/20160827140020530)

Name是自己起的远程仓库的名字，Location就是之前在coding上复制的git地址，填写完毕，点击“Add”，出现如下界面代表链接完成。（第一次使用时应该会弹出一个对话框让你填写coding的用户名和密码）

![链接完成](http://img.blog.csdn.net/20160827140334563)


## 第6步 更改配置信息（用户名和邮箱）
我们在Git GUI中点击“Edit”菜单下的“Options”选项，进入git配置界面。

![进入配置界面](http://img.blog.csdn.net/20160827134929906)

在如下界面填写你的配置，主要填写User Name和Email Address，还有更改默认文件编码为utf-8，其他选项默认即可。

![填写配置](http://img.blog.csdn.net/20160907211730129)

填写完毕，点击“Save”。

## 第7步 Commit
界面如下图。

![commit](http://img.blog.csdn.net/20160827140256376)

相关说明已经在图中标出，此处不再细说。  

有时候在团队合作开发的时候，我们需要创建自己的分支，用来归类自己提交的代码。此时我们需要使用到界面下方生成的那一段字符串，它是用来标记本次的提交记录的。

## 可选 创建分支

我们在Git GUI中点击“Branch”菜单下的“Create”选项，进入创建分支界面，如下图。

![创建分支](http://img.blog.csdn.net/20160827141906381)

填写“Name”(分支名称)和Revision Expression，点击“Create”即可创建。


## 第8步 Push
在Git GUI主界面点击“Push”按钮，可以将本地仓库的代码提交到远程仓库。

![点击Push](http://img.blog.csdn.net/20160827142256965)

选择分支，然后“Push”。

![填写远程仓库信息](http://img.blog.csdn.net/20160827142625432)

当出现如下界面代表已经Push完成。  

![Push完成](http://img.blog.csdn.net/20160827142959101)

----
分割线
----
将已经存在的文件夹链接到远端仓库。
在该文件夹中点右键，选择Git Bash Here

使用两条命令：git init 和 git remote add
如图所示。

![git bash](http://img.blog.csdn.net/20160827143840425)
