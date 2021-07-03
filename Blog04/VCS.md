# 有关VCS的一些知识

> 写在前面：
>
> * 本篇blog的主要原因是尝试做了一下HIT软件构造前些年的卷子，发现有关VCS这一块都考的比较细，其中不乏一些课件上没有的内容
> * 试卷上的部分选项在本人初次写该篇blog时仍然不确定，在得到确切答案后续会补上
> * ~~据说软件构造的blog要求是5篇，虽然考虑到前3篇的从内容来说应该足够了，但是还是补上来，内容相对单一、比较少~~

## 名词解释

*  VCS - Version Control System，版本控制系统
* SCM -  Software Configuration Management，软件配置管理
  * 用于追踪和控制软件的变化
* (S)CI -  (Software) Configuration Item，(软件)配置项
  * 软件中发生变化的**基本单元**
  * 个人理解上这里的“配置”和一般我们写例如web工程中的web.xml配置文件的“配置”并不是一个意思，注意区别，这里的配置就是软件中所有变化的单元
* baseline - 基线
  * 软件持续变化过程中的稳定时刻
  * 例如您所能看到各种标记有LTS (Long-term Support) 的发行版本
* CMDB - Configuration Management Database，配置管理数据库
  * 用于存储个配置随时间变化的信息 + 基线
  * 相当于是需要一个基本的信息，还要附带变化的信息，用以达到去除重复信息的目的

## 三种VCS

* Local VCS

  仓库存储在开发者的本地机器上，无法共享和协作

* Centralized VCS

  仓库存储在独立的服务器上，支持多开发者的协作

* Distributed VCS

  仓库存储在独立的服务器和每个开发者的本地机器上

## Git 与 SVN

### SVN

* 单词 subversion 的缩写

* **集中式版本控制工具**，用单一的服务器集中管理资源
* 协同工作的人们都通过客户端连到这台服务器，取出最新的文件或者提交更新

**优点**：

* 维护单一数据库的难度较低
* 访问者权限管理容易
* 每个人一定程度上都可以看到项目中其它人的工作

**缺点**：

* 无法处理单点宕机，即中央管理服务器故障的问题，这个是SVN最大的问题

### Git

* **分布式版本控制工具**
* 工作原理是所有客户端机器在本地创建完整的仓库镜像，这样任意一台机器发生故障，都可以利用其它任何一台机器的镜像进行恢复

**优点**：

* 服务器断网的情况下也可以进行开发，因为版本控制是在本地进行的
* 每个客户端保存的也都是整个完整的项目，包含历史记录，更加安全，能够很好解决SVN的单点宕机问题

#### Git仓库的三个组成部分

* .git目录：本地CMDB
* 工作目录：本地文件系统
* 暂存区：隔离工作目录和Git仓库，**是一块内存空间**

#### 关于Git的一些知识点

* 所有在Git上的操作在底层都用图结构进行标识和存储，称为 Git Object graph，这张图被存储在.git目录中，任何对git管理的项目的拷贝都意味着对这张图的拷贝

* Git**以文件为单位**进行存储
  * 意味着只要有代码发生变动，Git采取的方式是**存储这个变化的文件，而不是变化的代码行**
  * 传统的VCS倾向于存储变化的代码行
  
* Git中文件的三种状态：

  * Modified(已修改)：任何对现有文件的修改，使用Git Bash，再输入`git status`指令后会显示类似如下的内容：

    ```
    Untracked files:
     (use "git add <file>..." to include in what will be committed)
     
     	modified: hello.txt
     
    nothing added to commit but untracked files present (use "git add" 
    to track)
    ```

    其中 hello.txt 这一项会被标成红色，这就是Modified状态的文件

  * Staged(已暂存)

    在对上述文件使用`git add hello.txt`后，再输入`git status`会有类似如下的内容：

    ```
    warning: LF will be replaced by CRLF in hello.txt.
    The file will have its original line endings in your working 
    directory.
    ```

    其中 warning 那一行的原因在于Git采用的文本换行方式是LF，而Windows操作系统默认的文本换行方式是CRLF

  * Committed(已提交)

#### Git的常用操作

本地操作：

| 命令                            | 作用                                     |
| ------------------------------- | ---------------------------------------- |
| git init                        | 初始化本地库                             |
| git status                      | 查看本地库状态                           |
| git add 文件名                  | 将指定文件添加到暂存区                   |
| git commit -m “日志信息” 文件名 | 提交到本地库                             |
| git reflog                      | 查看所有版本信息                         |
| git reset --hard 版本号         | 版本穿梭                                 |
| git branch 分支名               | 创建分支                                 |
| git branch -v                   | 查看分支                                 |
| git checkout 分支名             | 切换分支                                 |
| git checkout -b 分支名          | 创建并切换分支                           |
| git merge 分支名                | 将**指定分支合并到当前分支**上           |
| git commit -m “日志信息”        | 在处理完冲突以后提交（**不能带文件名**） |

说明：

* `git add .`可以将所有未添加到暂存区的文件全部添加
* 分支合并产生冲突时需要手动修改代码，然后提交到本地库

远程操作：

| 命令                        | 作用                                       |
| --------------------------- | ------------------------------------------ |
| git remote -v               | 查看所有远程地址别名                       |
| git remote add 别名远程地址 | 给指定远程地址起别名                       |
| git push 别名 分支          | 将本地分支上的内容推送到远程仓库           |
| git clone 远程地址          | 克隆指定地址的仓库到本地                   |
| git pull 别名 远程库分支名  | 将对应分支拉取后**与当前本地分支直接合并** |

