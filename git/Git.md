
- [分布式版本控制系统](#分布式版本控制系统)
- [Git特点](#git特点)
  - [直接记录快照而非差异比较](#直接记录快照而非差异比较)
  - [大部分工作本地完成](#大部分工作本地完成)
  - [完整性](#完整性)
  - [三种状态](#三种状态)
- [Git基础](#git基础)
  - [创建git仓库](#创建git仓库)
    - [创建一个git仓库](#创建一个git仓库)
    - [克隆远程的仓库](#克隆远程的仓库)
  - [git更新](#git更新)
    - [git文件周期](#git文件周期)
    - [git add](#git-add)
    - [忽略文件](#忽略文件)
    - [查看修改](#查看修改)
    - [提交更新](#提交更新)
    - [移除文件](#移除文件)
    - [移动文件](#移动文件)

# 分布式版本控制系统

像git这种版本控制系统使用的是**分布式版本控制系统**，客户端不仅提取最新的文件快照，而是把代码仓库完整镜像下来。任何一处发生障碍的话可以使用一个本地版本进行复原。

<img src="https://git-scm.com/book/en/v2/images/distributed.png" alt="分布式版本控制图解" style="zoom: 50%;" />

# Git特点

## 直接记录快照而非差异比较
Git 和其它版本控制系统（包括 Subversion 和近似工具）的主要差别在于 Git 对待数据的方式。 从概念上来说，其它大部分系统以文件变更列表的方式存储信息，这类系统（CVS、Subversion、Perforce、Bazaar 等等） 将它们存储的信息看作是一组基本文件和每个文件随时间逐步累积的差异 （它们通常称作 **基于差异（delta-based）** 的版本控制）。

![存储每个文件与初始版本的差异。](https://git-scm.com/book/en/v2/images/deltas.png)



Git 不按照以上方式对待或保存数据。反之，Git 更像是把数据看作是对小型文件系统的一系列快照。 在 Git 中，每当你提交更新或保存项目状态时，它基本上就会对当时的全部文件创建一个快照并保存这个快照的索引。 为了效率，如果文件没有修改，Git 不再重新存储该文件，而是只保留一个链接指向之前存储的文件。 Git 对待数据更像是一个 **快照流**。

![Git 存储项目随时间改变的快照。](https://git-scm.com/book/en/v2/images/snapshots.png)

## 大部分工作本地完成

git的绝大多数操作都是在本地完成的，在本地完成绝大多数操作之后再进行提交

## 完整性

git种的所有数据都在**存储前**计算校验和，这样保证不可能再git不知情的情况下对于文件内容及目录内容进行更改

## 三种状态

git有三种状态

* **已提交**：已经存储在本地数据库中
* **已暂存**：对于一个已修改文件的当前版本做出了标记，使之包含在下次提交的快照中
* **已修改**：修改了文件但是没有保存在数据库中

![工作区、暂存区以及 Git 目录。](https://git-scm.com/book/en/v2/images/areas.png)

# Git基础
## 创建git仓库
获取git主要有两种方式

1. 将没有进行版本控制的目录转换为git仓库
2. 从其他服务器克隆一个git仓库

### 创建一个git仓库
先使用指令到达目标文件夹中

然后使用以下命令进行创建
```
$ git init
```
如果是在一个原本有文件的文件夹中创建的话，需要手动进行文件的跟踪，主要使用的是以下的命令
```
$ git add *.c
$ git commit -m "add some files"
```
### 克隆远程的仓库
```
$ git clone https://github.com/libgit2/libgit2
```
还可以对于克隆下来的文件目录进行重命名
```
$ git clone https://github.com/libgit2/libgit2 newProject
```
Git 支持多种数据传输协议。 上面的例子使用的是 https:// 协议，不过你也可以使用 git:// 协议或者使用 SSH 传输协议，比如 user@server:path/to/repo.git 。 在服务器上搭建 Git 将会介绍所有这些协议在服务器端如何配置使用，以及各种方式之间的利弊。

## git更新

### git文件周期
git的文件周期大致如下
![文件周期](https://git-scm.com/book/en/v2/images/lifecycle.png)

可以使用命令`git status`查看当前目录下的文件处于哪一个状态

### git add
注意：**git add指令的作用不只是将文件进行跟踪，还有将文件从已修改状态转换为已缓存的状态**

所以对于这个指令的更精确的理解是**将该文件添加到下一次提交中**

还可以使用更加精简的方式查看文件状态

```
$ git status -s
   M README
MM Rakefile
A  lib/git.rb
M  lib/simplegit.rb
?? LICENSE.txt
```
这里左边表示暂存区的状态，右边表示工作区的状态

### 忽略文件
可以使用.gitignore来避免一些文件的提交

常用的方式如下
```
# 忽略所有的 .a 文件
*.a

# 但跟踪所有的 lib.a，即便你在前面忽略了 .a 文件
!lib.a

# 只忽略当前目录下的 TODO 文件，而不忽略 subdir/TODO
/TODO

# 忽略任何目录下名为 build 的文件夹
build/

# 忽略 doc/notes.txt，但不忽略 doc/server/arch.txt
doc/*.txt

# 忽略 doc/ 目录及其所有子目录下的 .pdf 文件
doc/**/*.pdf
```
### 查看修改
可以使用指令`git diff`来进行原本文件和修改后文件的对比

### 提交更新
可以使用`git commit`进行文件的更新

使用这个命令之后会使用vim进行更新信息的填写

另外可以在commit后加上`-m`来直接填写更新信息
```
$ git commit -m "Story 182: Fix benchmarks for speed"
[master 463dc4f] Story 182: Fix benchmarks for speed
 2 files changed, 2 insertions(+)
 create mode 100644 README
 ```

`commit`还可以跳过暂存区域
在commit后面加上`-a`可以跳过add操作，将所有跟踪过的文件一起提交

### 移除文件
使用git移除的必须是git已经跟踪的文件

可以使用`rm`删除文件（包括文件目录中和git中）

另外一种情况是，我们想把文件从 Git 仓库中删除（亦即从暂存区域移除），但仍然希望保留在当前工作目录中。 换句话说，你想让文件保留在磁盘，但是并不想让 Git 继续跟踪。 当你忘记添加 .gitignore 文件，不小心把一个很大的日志文件或一堆 .a 这样的编译生成文件添加到暂存区时，这一做法尤其有用。 为达到这一目的，使用 --cached 选项：

```
$ git rm --cached README
```

### 移动文件
`git mv`指令可以移动文件，还可以对于文件进行重命名的操作

```
$ git mv README.md README
```

其实，运行 git mv 就相当于运行了下面三条命令：

```
$ mv README.md README
$ git rm README.md
$ git add README
```