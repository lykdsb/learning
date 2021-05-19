# Git

[TOC]

## 分布式版本控制系统

像git这种版本控制系统使用的是**分布式版本控制系统**，客户端不仅提取最新的文件快照，而是把代码仓库完整镜像下来。任何一处发生障碍的话可以使用一个本地版本进行复原。

<img src="https://git-scm.com/book/en/v2/images/distributed.png" alt="分布式版本控制图解" style="zoom: 50%;" />

## Git特点

### 直接记录快照而非差异比较

Git 和其它版本控制系统（包括 Subversion 和近似工具）的主要差别在于 Git 对待数据的方式。 从概念上来说，其它大部分系统以文件变更列表的方式存储信息，这类系统（CVS、Subversion、Perforce、Bazaar 等等） 将它们存储的信息看作是一组基本文件和每个文件随时间逐步累积的差异 （它们通常称作 **基于差异（delta-based）** 的版本控制）。

![存储每个文件与初始版本的差异。](https://git-scm.com/book/en/v2/images/deltas.png)



Git 不按照以上方式对待或保存数据。反之，Git 更像是把数据看作是对小型文件系统的一系列快照。 在 Git 中，每当你提交更新或保存项目状态时，它基本上就会对当时的全部文件创建一个快照并保存这个快照的索引。 为了效率，如果文件没有修改，Git 不再重新存储该文件，而是只保留一个链接指向之前存储的文件。 Git 对待数据更像是一个 **快照流**。

![Git 存储项目随时间改变的快照。](https://git-scm.com/book/en/v2/images/snapshots.png)

### 大部分工作本地完成

git的绝大多数操作都是在本地完成的，在本地完成绝大多数操作之后再进行提交

### 完整性

git种的所有数据都在**存储前**计算校验和，这样保证不可能再git不知情的情况下对于文件内容及目录内容进行更改

### 三种状态

git有三种状态

* **已提交**：已经存储在本地数据库中
* **已暂存**：对于一个已修改文件的当前版本做出了标记，使之包含在下次提交的快照中
* **已修改**：修改了文件但是没有保存在数据库中

![工作区、暂存区以及 Git 目录。](https://git-scm.com/book/en/v2/images/areas.png)
