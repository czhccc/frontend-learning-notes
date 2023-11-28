# Git

## .git文件

```javascript
hooks          目录包含客户端或服务端的钩子脚本 
info           包含一个全局性排除文件 
logs           保存日志信息 
objects        存储所有数据内容（所有项目的历史记录）
refs           存储指向数据的提交对象的指针（分支） 
config         文件包含项目特有的配置选项 
description    用来显示对仓库的描述信息 
HEAD           文件指示目前被检出的分支 
index          文件保存暂存区信息
```

## .gitignore

一般我们总会有些文件无需纳入 Git 的管理，也不希望它们总出现在未跟踪文件列表。通常都是些自动生成的文件，比如日志文件，或者编译过程中创建的临时文件等。我们可以创建一个名为 .gitignore 的文件，列出要忽略的文件模式。 

*.[oa] 

*~ 

第一行告诉 Git 忽略所有以 .o 或 .a 结尾的文件。一般这类对象文件和存档文件都是编译过程中出现的，我们用不着跟踪它们的版本。第二行告诉 Git 忽略所 有以波浪符（~）结尾的文件，许多文本编辑软件（比如 Emacs）都用这样的文 件名保存副本。此外，你可能还需要忽略 log、tmp 或者 pid 目录，以及自动生成 的文档等等。要养成一开始就设置好 .gitignore 文件的习惯，以免将来误提交这类无用的文件。

---

.gitignore 的格式规范：

所有空行或者以注释符号 ＃ 开头的行都会被 Git 忽略。

可以使用标准的 glob 模式匹配。

\* 代表匹配任意个字符  

? 代表匹配任意一个字符

** 代表匹配多级目录

匹配模式前跟反斜杠（/） 这个斜杠代表项目根目录。

匹配模式最后跟反斜杠（/）说明要忽略的是目录。

要忽略指定模式以外的文件或目录，可以在模式前加上惊叹号（!）取反。

---

示例：

 \# 此为注释 – 将被 Git 忽略 

\# 忽略所有 .a 结尾的文件 

\*.a 

\# 但 lib.a 除外 

!lib.a 

\# 仅仅忽略项目根目录下的 TODO 文件，不包括 subdir/TODO 

/TODO 

\# 忽略 build/ 目录下的所有文件 

build/ 

\# 会忽略 doc/notes.txt 但不包括 doc/server/arch.txt 

doc/*.txt

\# 忽略 doc/ 目录下所有扩展名为 txt 的文件 

doc/\*\*/*.txt（**通配符从 Git 版本 1.8.2 以上已经可以使用）

---

**GitHub 有一个十分详细的针对数十种项目及语言的 .gitignore 文件列 表，你可以在 https://github.com/github/gitignore 找到它。**

---

.gitignore 文件一般来说：

```
.DS_Store
node_modules/
/dist/
npm-debug.log*
yarn-debug.log*
yarn-error.log*

# Editor directories and files
.idea
.vscode
*.suo
*.ntvs*
.njsproj
*.sin 
```



## Git 的特点

1. **直接记录快照，而非差异比较**

   Git 和其他版本控制系统的主要差别在于，Git 只关心文件数据的整体是否发生变化；而大多数其他系统则只关心文件内容的具体差异，每次记录有哪些文件有更新以及更新的内容。

   下图是其他版本系统在每个版本中记录着各个文件的具体差异：

   <img src="Git.assets/1593236840944.png" alt="1593236840944" style="zoom:67%;" />

   Git 并不保存这些前后变化的差异数据。实际上，Git 更像是把变化的文件作快照后，记录在一个微型的文件系统中。每次提交更新时，会纵览一遍所有文件的指纹信息并对文件作一快照，然后保存一个指向这次快照的索引。为提高性能，若文件没有变化，Git 不会再次保存，而只对上次保存的快照作一链接。

   Git 的工作方式就像下图所示，保存每次更新时的文件快照：

   <img src="Git.assets/1593236858156.png" alt="1593236858156" style="zoom:67%;" />

   这是 Git 同其他系统的重要区别。它完全颠覆了传统版本控制的套路，并对各个环节的实现方式作了新的设计。Git 更像是个小型的文件系统，但它同时还提供了许多以此为基础的超强工具，而不只是一个简单的 VCS。

2. **近乎所有操作都是本地执行**

   Git 的绝大多数操作都只需要访问本地文件和资源，不用连网。因为 Git 在本地磁盘上就保存着所有当前项目的历史更新，所以处理起来速度更快。

3. **时刻保证数据完整性**

   在保存到 Git 之前，所有数据都要进行内容的校验和计算，并将此结果作为数据的唯一标识和索引。换句话说，不可能在你修改了文件或目录之后，Git 一无所知。这项特性作为 Git 的设计哲学，建在整体架构的最底层。所以如果文件在传输时变得不完整，或者磁盘损坏导致文件数据缺失，Git 都能立即察觉。Git 使用 SHA-1 算法计算数据的校验，通过对文件的内容或目录的结构计算出一个 SHA-1 哈希值，作为指纹字符串。该字串由 40 个十六进制字符（0-9 及 a-f）组成，看起来就像是：

   ​						24b9da6552252987aa493b52f8696cd6d3b00373 

   Git 的工作完全依赖于这类指纹字串，所以你会经常看到这样的哈希值。实际上，所有保存在 Git 数据库中的东西都是用此哈希值来作索引的，而不是靠文件名。 

4. **多数操作仅添加数据**

   常用的 Git 操作大多仅仅是把数据添加到数据库。因为任何一种不可逆的操作，比如删除数据，都会使回退或重现历史版本变得困难重重。在别的 VCS 中，若还未提交更新，就有可能丢失或者混淆一些修改的内容，但在 Git 里，一旦提交快照之后就完全不用担心丢失数据，特别是养成定期推送到其他仓库的习惯的话。这种高可靠性令我们的开发工作安心不少，尽管去做各种试验性的尝试，再怎样也不会弄丢数据。

5. **文件的三种状态**

   对于任何一个文件，在 Git 内都只有三种状态（Git 外的状态就是一个普通文件）： 

   ​	已提交（committed）：表示该文件已经被安全地保存在本地数据库中了。

   ​	已修改（modified）：表示修改了某个文件，但还没有提交保存。

   ​	已暂存（staged）：表示把已修改的文件放在下次提交时要保存的清单中。

   由此我们看到 Git 管理项目时，文件流转的三个工作区域： 工作区、暂存区、版本库。

   <img src="Git.assets/1593237455629.png" alt="1593237455629" style="zoom: 67%;" />

## Git 的流程

每个项目都有一个 Git 目录（ .git ）。用来保存元数据和对象数据库。该目录非常重要，每次克隆镜像仓库的时候，实际拷贝的就是这个目录里面的数据。 
1. 在工作目录中修改某些文件。

  从项目中取出某个版本的所有文件和目录，用以开始后续工作的叫做工作目录。这些文件实际上都是从 Git 目录中的压缩对象数据库中提取出来的，接下来就可以在工作目录中对这些文件进行编辑。

2. 保存到暂存区域，对暂存区做快照

  暂存区域只不过是个简单的文件，一般都放在 Git 目录中。有时候人们会把这个文件叫做索引文件，不过标准说法还是叫暂存区域。

3. 提交更新，将保存在暂存区域的文件快照永久转储到本地数据库（Git 目录）中 
    我们可以从文件所处的位置来判断状态：如果是 Git 目录中保存着的特定版本文件，就属于已提交状态；如果作了修改并已放入暂存区域，就属于已暂存状态；如果自上次取出后，作了修改但还没有放到暂存区域，就是已修改状态。

## 3个对象

- 一次完整的流程，*最少* 包含：一个git对象，一个树对象，一个提交对象。

- 一次提交：一个或多个git对象，只有一个树对象，只有一个提交对象。

### git对象

Git 的核心部分是一个键值对数据库，可以向该数据库插入任意类型的内容，它会返回一个唯一的HASH值，通过该键值可以在任意时刻再次检索该内容。内容改变时，返回新的HASH值。存储在 objects 文件夹中。

git对象，就是一个键值对，（只能）用来存储文件的内容（存储数据），“键”是HASH值，“值”是存储的内容。键值对在git内部的数据类型是blob类型。

一个文件的一次修改对应生成一个git对象，即便是同一个文件N次的修改也会生成N个git对象。因此，即便只是git add，但是没有提交，git也会进行管理，只要找到对应的HASH即可。

可以说，git对象代表文件的一次次版本。

问题：

- 每一个git对象只能代表一个文件（只是文件，不是项目），不能代表项目的快照（一个版本）。
- git对象只能通过对应的HASH值拿到存储的内容，但我们并不会去记住HASH值。

- git对象只保存文件的内容，不保存文件名。
- 解决方法：树对象。

### 树对象

树对象，能解决文件名保存的问题，也允许我们将多个文件组织到一起。所有内容均以 树对象 和 git对象 的形式存储，树对象 对应目录项， git对象 对应文件内容。一个树对象包含了一条或多条记录，每条记录含有一个指向 git对象 或 子树对象 的 SHA-1指针，以及相应的模式、类型、文件名信息。一个树对象也可以包含另一个树对象。 

可以说，树对象就是真正的项目的一次快照，代表项目的一次次版本。

<img src="Git.assets/1593063681023.png" alt="1593063681023" style="zoom: 33%;" />

问题：

现在有三个树对象（执行了三次 write-tree），分别代表了我们想要跟踪的不同项目快照。然而问题依旧：

- 若想重用这些快照，必须记住所有三个 SHA-1哈希值。 
- 你也完全不知道是谁保存了这些快照，在什么时刻保存的，以及为什么保存这些快照。
- 解决方法：提交对象。

### 提交对象

提交对象，就是对树对象进行包裹，并附加一些注释，例如作者是谁、谁提交的等。

提交对象是链式的，可以找到前一个提交对象是谁。因此，想要回退版本，只需找到某个提交对象对应的HASH值即可。

<img src="Git.assets/1593063942928.png" alt="1593063942928" style="zoom: 50%;" />

可以说，一个提交对象就是一个项目的版本，提交对象只是对树对象进行了一次封装，附带一些注释信息，由此从项目的快照变为项目的版本。

## 3个区域

### 工作区

一个文件夹经过 `git init` 命令后，该文件夹就是工作区了。

工作区下的所有文件，都不外乎这两种状态：**已跟踪**  或  **未跟踪** 。已跟踪的文件有3种状态：**已提交**、**已修改**、**已暂存**。所有其他文件都属于未跟踪文件，既没有上次更新时的快照，也不在暂存区。

初次克隆某个仓库时，工作目录中的所有文件都属于已跟踪文件，且状态为已提交；在编辑过某些文件之后，Git 将这些文件标为已修改。我们逐步把这些修改过的文件放到暂存区域，直到最后一次性提交所有这些暂存起来的文 件。使用 Git 时的文件状态变化周期如下图所示：

<img src="Git.assets/1593070353720.png" alt="1593070353720" style="zoom: 50%;" />

### 暂存区

就是 .git / index 文件。

### 版本库

就是 .git / objects 文件夹。

## 分支

分支就是一个动态的指针（HEAD），指向最新的提交对象，随着提交对象移动。

分支切换会改变你工作目录中的文件，在切换分支时，一定要注意你工作目录里的文件会被改变。如果是切换到一个较旧的分支，你的工作目录会恢复到该分支最后一次提交时的样子。如果 Git 不能干净利落地完成这个任务，它将禁止切换分支，因此，每次在切换分支前，提交一下当前分支，确保当前分支是干净的，即处于已提交状态，git status 后 显示 `nothing to commit, working tree clean`。

切换分支会涉及3个地方：1.HEAD 2.暂存区 3.工作区

### 坑

- 在切换分支时，如果当前分支上有初始化(新创建)的文件，且没有被添加到版本库（无论是否被 git add 加入到暂存区），分支可以成功切换，但是初始化的文件会污染切换过去的目标分支。
- 如果当前分支上有已经被 git commit 的，即已经被添加到版本库中的文件，有新的修改，但新的修改没有被 git commit 添加到版本库，此时如果切换分支，Git 会报错，拒绝切换分支。

例如，test 分支下初始化创建 test.txt 文件，未提交(git commit)（无论是否已经 git add），然后切换回 master 分支，此时，可以成功切换回 master 分支，但 test.txt 文件会在 master 分支下出现，污染 master 分支。因为 `check out` 命令会改变工作区，从 test分支 的工作区变为 master分支 的工作区，但是 test.txt 文件并未提交(git commit)，此时 Git 为了安全性(我们做的工作不会白费)，会保留 text.txt 文件，因此造成污染 master 分支的工作区。但是，一旦 test.txt 被 git commit 添加到版本库中，此时，如果 test 分支是不干净的，就不能再成功切换回 master 分支，Git 会报错。

### 快进合并 和 典型合并

**快进合并（Fast-forward）**：

- 当前分支所指向的提交是当前提交的直接上游，所以 Git 只是简单的将指针向前移动。换句话说，试图合并两个分支时，如果顺着一个分支走下去能够到达另一个分支，那么 Git 在合并两者的时候，只会简单的将指针向前推进（指针右移），因为这种情况下的合并操作没有需要解决的分歧。

  <img src="Git.assets/1593148343815.png" alt="1593148343815" style="zoom: 67%;" />

**典型合并**：

-  在这种情况下，你的开发历史从一个更早的地方开始分叉开来（diverged）。因为，master 分支所在提交并不是 iss53 分支所在提交的直接祖先，此时可能会产生冲突，例如修改过同一行代码，Git 不得不做一些额外的工作。出现这种情况的时候，Git 会使用两个分支的末端所指的快照（C4 和 C5）以及这两个分支的工作祖先（C2），做一个简单的三方合并。 

  <img src="Git.assets/1593148427242.png" alt="1593148427242" style="zoom:67%;" />

  和之前快进合并不同的是，Git 将此次三方合并的结果做了一个新的快照并且自动创建一个新的提交指向它。 这个被称作一次合并提交，它的特别之处在于有不止一个父提交。Git 会自行决定选取哪一个提交作为最优的 共同祖先，并以此作为合并的基础。

  <img src="Git.assets/1593149056072.png" alt="1593149056072" style="zoom: 67%;" />

  如果对同一个文件的同一个部分进行了不同的修改，就会产生冲突，Git 就没法干净的合并它们，合并分支命令后 Git 会有相应的提示，需要进入文件手动完成合并操作，决定文件的最终内容，最终使用 git add 命令来将其标记为冲突已解决，后续可以进行 git commit 。

### 分支的本质

分支本质仅仅是指向提交对象的可变指针。如果说的再细一些，分支其实就是一个提交对象，HEAD 才是可变指针，每次移动的都是 HEAD，每次提交后都是 HEAD 带着它所指向的分支往前移动，分支自己本身并不会移动。

Git 的默认分支名字是 master。 在多次提交操作之后，其实已经有一个指向最后那个提交对象的 master 分支。 它会在每次的提交操作中自动向前移动。Git 的 “master” 分支并不是一个特殊分支。 它就跟其它分支完全没有区别。 之所 以几乎每一个仓库都有 master 分支，是因为 git init 命令默认创建它，并且大多数 人都懒得去改动它。 

### 分支的原理

.git/refs 目录，保存了所有的分支，及其对应的提交对象HASH值。 .git/HEAD 就是可变指针

当运行类似于 git branch xxx 这样的命令时，Git 会取得当前所在分支最新提交对应的 SHA-1 值，并将其加入你想要创建的任何新分支中。当你执行 git branch xxx 时，Git 通过 HEAD文件 获取最新提交的 SHA-1 值，HEAD 文件是一个符号引用，指向当前所在的分支。所谓符号引用，意味着它并不像普通引用那样包含一个 SHA-1 值。它是一个指向其他引用的指针。

### stash 存储

有时，当你在项目的一部分上已经工作一段时间后，所有东西都进入了混乱的状态，而这时你想要切换到另一个分支做一点别的事情。问题是，你不想仅仅因为过会儿回到这一点而为做了一半的工作创建一次提交，此时，可以将已经完成的内容“存储”起来，切换到其他分支完成任务后，再回到当前分支，将“存储”的内容取出继续完成剩余内容，此时，需要用到 git stash 相关命令。

### 远程跟踪分支 & 远程分支

本地分支 <--> 远程跟踪分支 <--> 远程分支   

远程跟踪分支 是 远程分支 的引用，你不能手动控制，一般也不需要我们自己手动关联两者，当你操作远程仓库时，会自动移动。它们以 <remote>/<branch> 形式命名，例如 origin/master 。

远程分支只有在你 push 后才会同步本地的数据，并不会自动同步本地。

push 时会自动生成远程跟踪分支，但并不会自动和本地分支相关联。

确保本地分支已经跟踪了远程跟踪分支，那么一个本地分支怎么关联一个远程跟踪分支？

1. 只有在克隆 clone 操作的时候，默认会有且仅有自动生成一个 已经跟踪了 对应的远程跟踪分支 origin/matser 的本地分支 master ，不需要手动关联。
2. 新建一个分支时，指定想要关联的远程跟踪分支：git checkout -b / git checkout --track
3. 将一个已经存在的本地分支，关联一个远程跟踪分支：git branch -u



例子：

假设你的网络里有一个在 git.ourcompany.com 的 Git 服务器。如果你从这里克隆，Git 的 clone 命令会为你自动将其命名 为 origin，拉取它所有的数据，创建一个指向它的 master 分支的指针，并且在本地将其命名为 origin/master。Git 也会给你一个与 origin/master 分支在指向同一个地方的本地 master 分支， 这样你就有工作的基础。

<img src="Git.assets/1593321812790.png" alt="1593321812790" style="zoom: 50%;" />

如果你在本地的 master 分支做了一些工作，然而在同一时间，其他人推送提交到 git.ourcompany.com 并更新了它的 master 分支，那么你们的提交历史将向不同的方向前进。 只要你不与 origin 服务器连接，你的 origin/master 指针就不会移动。

<img src="Git.assets/1593321903005.png" alt="1593321903005" style="zoom: 50%;" />

如果要同步你的工作，运行 git fetch origin 命令。这个命令查 找 “origin” 是哪一个服务器（在本例中，它是 git.ourcompany.com），从中抓取本地没有的数据，并且更新本地数据库，移动 origin/master 指 针指向新的、更新后的位置。

---

推送其他分支：

当你想要公开分享一个分支时，需要将其推送到有写入权限的远程仓库上。本地的分支并不会自动与远程仓库同步，你必须显式地推送想要分享的分支。这样，你就可以把不愿意分享的内容放到私人分支上，而将需要和别人协作的内容推送到公开分支，如果希望和别人一起在名为 xxx 的分支上工作，你可以像推送第一个分支那样推送它。

具体的命令见“实际中常用的命令 — 远程仓库”。

---

跟踪分支：

从一个远程跟踪分支（origin/master）检出一个本地分支会自动创建一个叫做 “跟踪分支”（有时候也叫做 “上游分支” ：master）。只有主分支 并且 克隆时才会自动建跟踪分支。跟踪分支是与远程分支有直接关系的本地分支。 如果在一个跟踪分支上输入 git pull，Git 能自动地识别去哪个服务器上抓取、合并到哪个 分支。如果你愿意的话可以设置其他的跟踪分支，或者不跟踪 master 分支。

具体的命令见“实际中常用的命令 — 远程仓库”。

## pull request

如果你想要参与某个项目，但是并没有推送权限，这时可以对这个项目进行“派生”（ Fork ）。派生的意思是指，GitHub 将在你的空间中创建一个完全属于你的项目副本，且你对其具有推送权限。通过这种方式，项目的管理者不再需要忙着把用户添加到贡献者列表并给予他们推送权限。人们可以派生这个项目，将修改推送到派生出的项目副本中，并通过创建合并请求（ Pull Request ）来让他们的改动进入源版本库。

基本流程： 

1. 从 master 分支中创建一个新分支（自己 fork 的项目） 
2. 提交一些修改来改进项目（自己 fork 的项目）
3. 将这个分支推送到 GitHub 上（自己 fork 的项目）
4. 创建一个合并请求
5. 讨论，根据实际情况继续修改 
6. 项目的拥有者合并或关闭你的合并请求

注意点：每次在发起新的 Pull Request 时，要去拉取最新的源仓库的代码，而不是自己 fork 的那个仓库：

git remote add <shortname 源仓库> <url 源仓库> 
git fetch 远程仓库名字 
git merge 对应的远程跟踪分支


## 实际中常用的命令

### 基本使用

```haskell
git init 
初始化仓库，git 开始对该文件夹下的内容进行管理。
作用：初始化后，在当前目录下会出现一个名为 .git 的目录，所有 git 需要的数据和资源都存放在这个目录中。不过目前，仅仅是按照既有的结构框架初始化好了里边所有的文件和目录，但还没有开始跟踪管理项目中的任何一个文件。
 
git status
查看工作区的文件的状态
如果要查看具体修改了什么地方，用 git diff 命令

git ls-files -s
查看暂存区

git add readme.txt
把某个有变更的文件添加到暂存区；跟踪文件，变为已暂存状态
git add .   
将工作区的所有变更添加到暂存区；如果为目录，递归跟踪该目录下的所有文件
git add 命令真正的操作：将工作区的修改作为 git对象 加入到版本库，再从版本库中加入到暂存区。注意：如果修改了3个文件，就会有3个git对象，即使是同一个文件，修改了3次也会生成3个git对象。详细见git对象。
如果一个文件在 git add 后，没有提交，但经过了修改，此时 git status 会发现该文件有2种状态：已暂存、已修改，Git 只暂存了你运行 git add 命令时的版本，新修改后的版本并没有被暂存，此时需要再次 git add

git commit -m "我是注释内容"
把暂存区中暂存的所有变更一次性提交到仓库，-m 后面输入的是本次提交的说明，可以输入任意内容。如果不加 -m ，会启动文本编辑器以便输入本次提交的说明。默认的提交消息包含最后一次运行 git status 的输出，放在注释行里， 另外开头还有一空行，供你输入提交说明。你完全可以去掉这些注释行， 不过留着也没关系，多少能帮你回想起这次更新的内容有哪些。
真正的操作：参照暂存区生成一个树对象，放到版本库中，如果有提交信息，再从版本库中取出树对象，附加注释后生成提交对象，放入版本库。 
注意：该命令不会提交未暂存的内容，提交不会清空暂存区。
注意：
当一个文件修改，并add后，但没有进行commit，再进行第二次修改，此时：
	1. 如果commit并且指定文件名（commit readme.txt -m ""），第二次修改的内容也会被提交
	2. 如果commit但不指定文件名（commit -m ""），只会提交第一次修改的内容
	
git commit -a -m "..."
-a 选项，Git 会自动把所有已经跟踪过的文件暂存起来一并提交，跳过 git add 步骤，注意：必须是已跟踪的文件才能 -a ，未跟踪的文件必须先手动 git add

git config --global alias.xxx commit
git config --global alias.xxx "reset --hard HEAD^"
给 Git 命令配置别名，上述配置后，当要输入 git commit 时，只需要输入 git xxx
当命令不止一个单词时，需要用双引号
```

### 分支

```haskell
git branch
列出当前所有存在的分支，当前分支前面会标一个 * 号

git branch  -v
查看每一个分支的最后一次提交

git branch xxx
创建一个 xxx 分支

git branch xxx 8b0d293
新建一个 xxx 分支并且使分支指向对应的提交对象，8b0d293 为提交对象的 HASH

git branch -d xxx
删除 xxx 分支，必须先切到其他分支，即，不能在 xxx 分支上删除 xxx 分支，要先切换到 master 或其他分支上，才能删除 xxx 分支
git branch -D xxx
强行删除没有合并的 xxx 分支，删除后不能找回

git branch –merged  
查看哪些分支已经合并到当前分支
在这个列表中分支名字前没有 * 号的分支通常可以使用 git branch -d 删除

git branch --no-merged
查看所有包含未合并工作的分支
使用 git branch -d 删除在这个列表中的分支会失败，如果真的想要删除分支并丢掉那些工作，可以使用 -D 选项强制删除

git checkout xxx  /  git switch xxx （推荐使用 switch ，新版命令）
切换到 xxx 分支
与 reset 不同，只会移动 HEAD ，不会带着分支一起移动

git checkout -b xxx  /  git switch -c xxx （推荐使用 switch ，新版命令）
创建一个 xxx 分支，并同时切换到 xxx 分支，相当于同时执行 git branch xxx、git checkout xxx

git merge xxx
快进合并模式 将指定分支到当前分支，并且丢掉分支信息
例如当前是 master 分支，则是把 xxx 分支合并到  master 分支

git merge --no-ff xxx
--no-ff 参数表示使用普通的合并模式合并，合并后的历史有分支，能看出来曾经做过合并

git log --graph
查看分支合并图，即分支的合并情况

git log --oneline –decorate
查看当前分支所指对象，提供这一功能的参数是 --decorate

git log --oneline --decorate --graph --all
查看项目分叉历史
```

### stash 存储

```haskell
git stash
可以把当前工作现场“存储”起来，等以后恢复现场后继续工作
该命令会将未完成的修改保存到一个栈(先进后出)，可以在任何时候重新应用这些改动(git stash apply) 
可以多次git stash

git stash list
查看所有“存储”的工作现场

git stash apply
恢复栈(先进后出)最顶部的 stash 的内容，注意：该命令只会应用栈顶的 stash ，并不会将其删除

git stash pop
恢复现场的内容的同时删除现场

git stash apply stash@{0}
恢复指定的stash

git stash drop stash@{0}
删除“存储”的现场的内容

git cherry-pick 4c805e2
复制一个特定的提交到当前分支
```

### 重命名

```haskell
git mv xxx.txt yyy.txt
将工作区的文件重命名，并将该操作添加到暂存区，只需要 git commit 提交即可
相当于以下三条命令：
1. mv xxx.txt yyy.txt
2. git rm xxx.txt
3. git add yyy.txt
```

### 比较3个区域

```haskell
git diff / git diff readme.txt    diff就是difference的意思
当前哪些更新还没有暂存
比较工作区的文件和暂存区快照（上次 git add 后的内容）的差异，也就是 修改后但还没有暂存 和 已经暂存的变化内容。

git diff --cached / git diff readme.txt --cached / git diff –staged
哪些更新已经暂存起来准备好了下次提交
比较暂存区的文件与仓库分支里（上次 git commit 后的内容）的区别，也就是 暂存后但还没有提交 和 仓库分支中已经提交的变化内容。

请注意，git diff 本身只显示尚未暂存的改动，而不是自上次提交以来所做的所有改动。所以有时候你一下子暂存了所有更新过的文件后，运行 git diff 后却什么也没有，就是这个原因。
```

### 删除

```haskell
git rm test.txt
同时从工作区和版本库中删除文件，本地文件也会被删除。相当于先在工作区中删除文件，再将删除操作自动提交到暂存区，可以继续 git commit ，可以通过 git reset 回退到历史版本，再依次 git reset HEAD readme.txt 、 git checkout -- readme.txt 拿回删除的文件的某个版本。（git reset HEAD用来撤销rm操作，否则暂存区中有rm操作，git checkout -- 就无法正常恢复文件内容）
简单来说，git rm 命令本质上也是向版本库中新增一个提交对象，删除后的文件仍然可以找回。

git rm test.txt --cached
只从版本中删除文件，本地的文件不会被删除，即git不再跟踪该文件。

rm test.txt
从本地删除文件，文件变为已修改状态(删除操作在 Git 看来就是修改操作)，不会自动将该操作添加到暂存区，可以继续 git add、git commit
只是当前项目快照中没有该文件，Git 中仍有记录，新增了一个提交对象和树对象，只是树对象中不包含git对象。
如果是误删，可以通过 git reset 版本回退到历史版本，再 git checkout -- readme.txt 拿回删除的文件的某个版本。
```

### 操作撤销 / 版本回退

```haskell
git checkout -- test.txt / git restore test.txt(推荐)
撤销 test.txt 文件在工作区的修改
其实是用暂存区或版本库中最新的版本重新覆盖工作区的文件，因此，这里有两种情况：
	1. test.txt已修改但还没有被添加暂存区，现在，撤销修改就回到和版本库一模一样的状态。
	2. test.txt已添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。
总之，就是让这个文件回到最近一次 git commit 或 git add 时的状态。
-- 很重要，没有 -- ，就变成了“切换分支”的命令。

git reset HEAD readme.txt / git restore --staged test.txt(推荐)
与 git add 相反，撤销从工作区提交到暂存区的操作，不会改变工作区的内容
--staged 参数表示仅仅恢复暂存区的

git commit --amend
撤销上一次提交操作，然后将当前暂存区中的内容重新提交一次，重新提交过程中可以指定新的提交注释
应用场景：
		1. 提交注释写错了，导致提交日志不干净。此时直接使用该命令可以修改注释
		2. git add 后，再次修改文件内容但新的修改内容没有再次 git add ，此时 git commit 只提交了第一次的内容，此时放弃这次提交。此时，先进行 git add 操作，将新的修改内容添加到暂存区，然后再 git commit --amend
注意：
		1. 最终你只会有一个提交，第二次提交将代替第一次提交的结果，但是本质上第一次提交结果并没有消失，还是有记录的，这里只是进行了覆盖。
		2. 这个命令会将暂存区中的文件提交，这也是为什么场景2中需要先进行 git add 操作将新的修改内容添加到暂存区。
		3. git commit --amend 的实现原理就是 git reset --soft HEAD~ 
		
git restore --source=HEAD~2 --staged --worktree test.txt 
从指定版本库中同时恢复工作区和暂存区

git reset --hard HEAD^
版本回退，但是如果提交到远程仓库，例如Github，是不能回退的
HEAD表示当前版本，上一个版本就是HEAD^，上上个版本就是HEAD^^，往上100个版本写成HEAD~100。

git reset --hard 1094a   
可以直接指定版本号，版本号没必要写全，前几位就可以了，Git会自动匹配，但也不能只写前一两位，因为Git可能会匹配到多个版本号，就无法确定是哪一个了。

git restore -s HEAD~1 test.txt	
将文件切换到上一个 commit 的版本

git restore -s dbv213 test.txt
将文件切换到指定的 commit 版本，dbv213 为HASH

git reset --soft HEAD~1 / git reset --soft 16b1c9a
改变 HEAD
回退到指定的 git commit 之前的版本，不影响工作区和暂存区（用某一个提交对象的内容重置 HEAD 内容）
本质上是撤销上一次 git commit 命令，将 HEAD 及其所指向的分支一起移动回 git commit 前的原来的位置，不会改变暂存区和工作目录。当再次运行 git commit 时，创建一个新的提交对象，并移动 HEAD 及其所指向的分支来指向该新创建的提交对象。即，该命令只移动 HEAD 及其所指向的分支，不影响工作区和暂存区的内容。
与 checkout 不同的是，checkout 只会移动 HEAD ，不会带着所指向的分支一起移动；而该命令是 HEAD 及其所指向的分支一起移动。
git reset --soft HEAD~ 就是 git commit --amend 的实现原理。
                             HEAD														 HEAD
                              ↓																↓	
 v表示提交后的版本            master    使用命令后变为					  master
                              ↓																↓
file.txt.v1  file.txt.v2  file.txt.v3   			file.txt.v1  file.txt.v2  file.txt.v3  

		HEAD			   暂存区         工作区     =>  		HEAD			   暂存区         工作区
file.txt.v3		file.txt.v3		file.txt.v3    	file.txt.v2		file.txt.v3		file.txt.v3


git reset HEAD~1 / git reset 16b1c9a / git reset [--mixed] HEAD~1
[--mixed] 表示是默认选项，可以不写，默认使用 HEAD^
改变 HEAD 和 暂存区
撤销上一次的提交，同时还会取消暂存所有的东西，相当于回滚到所有 git add 和 git commit 的命令执行之前，不影响工作区。 
                             HEAD														 HEAD
                              ↓																↓	
 v表示提交后的版本            master    使用命令后变为					  master
                              ↓																↓
file.txt.v1  file.txt.v2  file.txt.v3   			file.txt.v1  file.txt.v2  file.txt.v3  

		HEAD			   暂存区         工作区     =>  		HEAD			   暂存区         工作区
file.txt.v3		file.txt.v3		file.txt.v3    	file.txt.v2		file.txt.v2		file.txt.v3


git reset --hard HEAD~1 
改变 HEAD 、暂存区、工作区
撤销上一次的提交，清空暂存区，工作区的内容被版本库中的内容替换
                             HEAD														 HEAD
                              ↓																↓	
 v表示提交后的版本            master    使用命令后变为					  master
                              ↓																↓
file.txt.v1  file.txt.v2  file.txt.v3   			file.txt.v1  file.txt.v2  file.txt.v3  

		HEAD			   暂存区         工作区     =>  		HEAD			   暂存区         工作区
file.txt.v3		file.txt.v3		file.txt.v3    	file.txt.v2		file.txt.v2		file.txt.v2

注意：--hard 标记是 reset命令 唯一的危险用法，也是 Git 会真正地销毁数据的仅有的几个操作之一。其他的 reset命令 可以撤消，但是 --hard 不能，因为它强制覆盖了工作区中的文件。在这种特殊情况下，Git 数据库中的一个提交内还留有该文件的v3版本，可以通过 reflog 找回。但是如果该文件还未提交，Git 仍会覆盖它从而导致彻底的无法恢复。


git reset test.txt / git reset --mixed HEAD~1/指定HASH test.txt
指定 路径 ！！！   注意：只有 git reset --mixed 命令后面才能加路径，其他命令不支持
撤销暂存区中该文件的修改
本质上是用上一次提交对象的暂存区中的该文件的内容覆盖当前暂存区中该文件的内容（也可以理解为，用上一次提交对象的暂存区中的内容覆盖当前暂存区中的内容，但是只作用于指定的文件）
若给 reset 命令指定了一个路径，reset 会跳过第1步（即不会移动 HEAD ），并且将作用范围限定为指定的文件或文件集合，否则会应用于所有的文件。因为 HEAD 本质上是一个指针，指向一个提交对象，而一个 提交对象 可以对应多个 树对象，一个 树对象 可以对应多个 git对象 ，而一个git对象就是一个文件 ，因此 HEAD可以代表多个文件的状态 。因为 HEAD 只是一个指针，无法让它同时指向两个提交对象中各自的一部分，但是工作区和暂存区可以部分更新，所以重置会继续进行第2、3步。
                 HEAD														 					 HEAD
                  ↓																					↓	
                master    					  										master
                  ↓																					↓
            file.txt.v1    															file.txt.v1  
			使用 git add file.txt 命令后                 使用 git reset file.txt 命令后
		HEAD			   暂存区         工作区     =>  		HEAD			   暂存区         工作区
file.txt.v1		file.txt.v2		file.txt.v2    	file.txt.v1		file.txt.v1		file.txt.v2

------------ 内容补充 ------------
checkout：
不带路径：
git checkout [branch]
checkout 与 reset 区别：
  1. checkout 对于工作区是安全的，通过检查来确保不会将已更改的文件弄丢而。而 reset --hard 会强制覆盖，是不安全的
  2. checkout 移动 HEAD 不会带着所指向的分支，而 reset 会将 HEAD及其所指向的分支 一起移动
  				    HEAD                     HEAD                		  HEAD
  					 	  ↓										     ↓												↓ 
  master  		develop			     master  develop									master    develop
    ↓				 		↓						     ↓   ↙											 		 ↓					↓
 commit A ← commit B  	 	     commit A ← commit B      			commit A ← commit B
  **before command**      	    **after reset**							   **after checkout**
  
带路径：
git checkout commithash <file>   
checkout 指定一个文件路径，会像 reset 指定路径一样，不会移动 HEAD 。就像是 git reset --hard [branch] file。这样对工作目录并不安全，它也不会移动 HEAD，将会跳过第1步，更新暂存区和工作目录。


git checkout -- <file> 
相比于 git reset --hard commitHash --filename，第1、2步 都没做，只会影响工作目录，撤回工作目录的修改，拿 HEAD 中的内容覆盖工作目录。
```

### 远程仓库

```haskell
远程协作的基本流程：
1. 项目经理创建一个空的远程仓库
2. 项目经理创建一个待推送的本地仓库
3. 项目经理为远程仓库配别名和用于信息
git remote add <shortname> <url>
添加一个新的远程 Git 仓库，同时指定一个可以轻松引用的简写

git remote
查看远程库的信息，远程仓库的默认名称是 origin
git remote -v
显示更详细的信息

git remote show <remote-name>
查看某一个远程仓库的更多信息 

git remote rename origin xxx 
重命名 

git remote rm <remote-name>
移除一个远程仓库

4. 项目经理推送本地项目到远程仓库
git push <remote-name> <branch-name>
将本地项目的分支推送到远程服务器 

5. 项目经理邀请成员加入项目
6. 成员克隆远程仓库
git clone <url>
git clone <url> -o xxx
克隆时为远程仓库起的默认别名为 origin ，远程仓库名字 origin 与分支名字 master 一样，在 Git 中并没有任何特别的含义一样。

git branch -vv
查看本地分支所跟踪的远程分支
结果示例：
	iss53    7e424c3 [origin/iss53: ahead 2] forgot the brackets   	
	master   1ae2a45 [origin/master] deploying index fix 
	* xxx   f8674d9 [teamone/yyy: ahead 3, behind 1] this should do it
	testing   5ea463a trying something new
含义：
	iss53 分支正在跟踪 origin/iss53 ，ahead 的值为2，表示本地有两个提交还没有推送到服务器上。 	     master 分支正在跟踪 origin/master 分支并且是最新的。
	xxx 分支正在跟踪 teamone 服务器上的 yyy 分支，并且领先3个版本、落后1个版本，意味着本地有三次提交还没有推送到服务器，并且同时服务器上有一次新的提交需要在本地合并。
	testing 分支没有跟踪任何远程分支。 
	
注意：ahead 和 behind 的值来自于你从每个服务器上最后一次抓取的数据，是当时抓取时在本地缓存的值，可能服务器最新的值已经改变。如果想要服务器上最新的值，需要在运行此命令前抓取所有的远程仓库：git fetch --all 、git branch –vv 

git checkout -b <branch> <remote-name>/<remote-branch> 
git checkout -b xxx origin/xxx
git checkout --track origin/xxx       --track 是快捷方式，两者名字相同
创建一个关联指定远程跟踪分支的本地分支，名称最好一致

git branch -u <remote-name>/<branch-name>
将当前所在分支关联指定的远程分支

git branch --set-upstream-to xxx origin/xxx
git branch --set-upstream-to=origin/xxx xxx
创建本地分支 xxx 和远程分支 xxx 的链接

7. 成员推送提交到远程仓库
git branch -r
查看远程的分支名

git push origin --delete <branchName>
删除指定的远程分支，删除远程分支后，本地仍然会有远程跟踪分支。 --delete 选项相当于推送本地空的分支到远程分支

git remote prune origin --dry-run
列出仍在远程跟踪但是远程已被删除的无用分支
git remote prune origin
清除上面命令列出来的本地远程跟踪分支

git pull
抓取最新的提交，如果有冲突，要先处理冲突。相当于一步完成 git fetch <remote-name> 操作，但是需要先将本地分支与指定的远程跟踪分支关联，这样，才能通过远程跟踪分支获取到正确的远程分支

---------------------------------------------------------------
git push <remote-name> <branch-name>:<remote>/<remote-branch-name>
push 只会生成远程跟踪分支，不会自动和本地分支相关联
推送分支，把该分支上的所有本地提交推送到远程仓库。推送时，要指定本地分支，这样，Git 就会把该分支推送到远程库对应的远程分支上。本地新建的分支如果不推送到远程，对其他人就是不可见的。
只有当你有所克隆服务器的写入权限，并且之前没有人推送过时，这条命令才能生效。当你和其他人在同一时间克隆，他们先更新了内容然后推送，你再推送到，你的推送就会被拒绝，因为此时你的修改和其他人的修改典型合并会产生冲突，你必须先将他们的工作拉取下来并将其合并进你的工作后才能继续推送。

git push localOriginName xxx:origin/yyy 完整的写法，将本地的 xxx 分支推送到远程的 origin/yyy 分支，本地的分支与远程的分支名字不需要相同，但一般为了便捷性两者是同名的，localOriginName 是 origin 在本地的别名。

git push origin xxx ，如果省略远程分支（<remote-branch-name>），则表示将本地分支推送到与之存在追踪关系的远程分支（通常两者同名），如果该远程分支不存在，则会被新建。注意：这里有些工作被简化了。 Git 自动将 serverfix 分支名字展开为 refs/heads/xxx:refs/heads/xxx 你也可以运行 git push origin xxx:xxx ，它会做同样的事，相当于，“推送本地的 xxx 分支，将其作为远程仓库的 xxx 分支”。
如果并不想让远程仓库上的分支叫做 xxx ，可以运行以下命令将本地的 xxx 分支推送到远程仓库上的 yyy 分支：git push origin xxx:yyy

git push origin :origin/xxx ，冒号后面的参数是要推送到服务器名和远端分支名，如果省略本地分支名（<branch-name>），则表示删除对应的远程分支，因为这等同于推送一个空的本地分支到远程分支进行覆盖，相当于 git push origin --delete <branchName> 。

git push origin ，如果当前分支与远程分支存在追踪关系，则本地分支和远程分支都可以省略，将当前分支推送到 origin 主机的对应分支。

git push ，如果当前分支只有一个远程分支，那么主机名都可以省略
---------------------------------------------------------------

本地分支与远程跟踪分支相关联后，就只需直接 git push 、git pull 即可

8. 项目经理更新成员提交的内容
git fetch <remote-name>
访问远程仓库，拉取所有你还没有的数据，并且将对应的远程跟踪分支指向新的数据，执行完成后，你将会拥有那个远程仓库中所有分支的引用，可以随时合并或查看，必须注意 git fetch 命令会将数据拉取到你的本地仓库，它并不会自动合并或修改你当前的工作。当准备好时你必须手动将其合并入你的工作。即，git fetch 后，你需要在本地手动创建或切换到某一分支，然后 git merge origin/xxx 合并对应的远程跟踪分支
```

### tag

```haskell
Git 可以给历史中的某一个提交打上标签，以示重要。可以使用这个功能来标记发布结点（例如 v1.0）。
两种主要类型的标签：轻量标签 与 附注标签  
轻量标签
			就像一个不会改变的分支，它只是一个特定提交的引用。
附注标签
			是存储在 Git 数据库中的一个完整对象。可以被校验；其中包含打标签者的名字、电子邮件地址、日期时间；还有一个标签信息；通常建议创建附注标签，这样你可以拥有以上所有信息；但是如果只是想用一个临时的标签，或者因为某些原因不想要保存那些信息，轻量标签也是可用的
			
远程标签
			默认情况下，git push 不会传送标签到远程仓库服务器上。在创建完标签后必须显式地推送标签到共享服务器上。你可以运行 git push origin [tagname] 如果想要一次性推送很多标签，可以使用带有 --tags 选项的 git push 命令。这将会把所有不在远程仓库 服务器上的标签全部传送到那里。

git tag
列出标签
git tag -l 'v1.8.5*'
列出1.8.5后的所有版本：
 		v1.8.5 v1.8.5-rc0 v1.8.5-rc1 v1.8.5-rc2 v1.8.5-rc3 v1.8.5.1 v1.8.5.2 v1.8.5.3

git tag <tagname> / git tag <tagname> <commitHash>
打轻量标签，如果不指定提交对象的 HASH ，则默认使用最新的提交对象

 
git tag -a v1.4 
git tag -a v1.4 <commitHash>  
git tag -a v1.4 <commitHash> -m 'my version 1.4'
打附注标签

git show <tagname>
查看指定的标签，可以显示任意类型的对象（git 对象 树对象 提交对象 tag 对象）

git push origin --tags
远程标签

git tag -d <tagname>
删除掉你本地仓库上的标签，
例如，可以使用下面的命令删除掉一个轻量级标签：git tag -d v1.4 
上述命令并不会从任何远程仓库中移除这个标签，必须使用 git push <remote> :refs/tags/<tagname> 来更新你的远程仓库：
git push origin :refs/tags/v1.4

git checkout <tagname>  +  git checkout -b <branchname>
查看某个标签所在的文件版本
回退到某一 tag 所在的版本，会处于“分离头指针”状态（HEAD 没有指向任何分支），此时，如果做了某些更改然后提交它们，标签不会发生变化，但新的提交将不属于任何分支，且无法访问，除非访问确切的提交对象的 HASH 。因此，如果在回退后进行更改，比如修复旧版本的错误，需要创建一个新分支，让 HEAD 指向该新创建的分支 
```

### 历史记录

```haskell
git log
查看提交历史记录，不会包含撤销操作等命令的历史

git log --pretty=oneline
--pretty=oneline 参数、结果集中到一行
    
git log --oneline
HASH 值变短
HEAD 表示当前版本，上一个版本就是HEAD^，上上一个版本就是HEAD^^，往上100个版本写成HEAD~100

git log -g
以标准日志的格式输出引用日志

git reflog
查看真正的完整的命令历史，包括撤销操作等，可以用来在版本回退后再想回到最新版本时，找到最新版本的版本号。只要 HEAD 有变化，那么 reflog 就会记录。
```



# 参考来源

1. [廖雪峰 Git](https://www.liaoxuefeng.com/wiki/896043488029600)
2. 尚硅谷达姆老师 Git 公开视频