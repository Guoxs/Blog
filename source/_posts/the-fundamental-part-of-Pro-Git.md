---
title: 《Pro Git》学习笔记（基础篇）
date: 2016-06-26 19:33:19
tags: git
---

## git工作原理
Git 和其他版本控制系统的主要差别在于，Git只关心文件数据的**整体**是否发生变化，而大多数其他系统则只关心文件内容的具体差异。Git 并不保存这些前后变化的差异数据。实际上，Git 更像是把变化的文件作**快照**后，记录在一个微型的文件系统中。每次提交更新时，它会纵览一遍所有文件的**指纹信息**并对文件作一快照，然后保存一个指向这次快照的**索引**。为提高性能，若文件没有变化，Git 不会再次保存，而只对上次保存的快照作一**链接**。

![git工作原理][1]

<!-- more -->

>文件在保存到 Git 之前，所有数据都要进行内容的校验和（checksum）计算，并将此结果作为数据的唯一标识和索引。换句话说，不可能在你修改了文件或目录之后，Git 一无所知。这项特性作为 Git 的设计哲学，建在整体架构的最底层。所以如果文件在传输时变得不完整，或者磁盘损坏导致文件数据缺失，Git 都能立即察觉。

Git 使用 SHA-1 算法计算数据的校验和，通过对文件的内容或目录的结构计算出一个 SHA-1 哈希值，作为指纹字符串。该字串由 40 个十六进制字符（0-9 及 a-f）组成，看起来就像是：
>24b9da6552252987aa493b52f8696cd6d3b00373

**所有保存在 Git 数据库中的东西都是用此哈希值来作索引的，而不是靠文件名。**

## 文件的三种状态
对于任何一个文件，在 Git 内都只有三种状态：已提交（committed），已修改（modified）和已暂存（staged）。已提交表示该文件已经被安全地保存在本地数据库中了；已修改表示修改了某个文件，但还没有提交保存；**已暂存表示把已修改的文件放在下次提交时要保存的清单中**。
 Git 管理项目时，文件流转的三个工作区域：Git 的工作目录，暂存区域，以及本地仓库:
![git中文件三种状态][2]

>每个项目都有一个 Git 目录，如果 `git clone` 出来的话，就是其中 `.git` 的目录；如果 `git clone --bare` 的话，新建的目录本身就是 Git 目录。它是 Git 用来保存元数据和对象数据库的地方。该目录非常重要，**每次克隆镜像仓库的时候，实际拷贝的就是这个目录里面的数据。**

从项目中取出某个版本的所有文件和目录，。这些文件实际上都是从 Git 目录中的**压缩对象数据库**中提取出来的，接下来就可以在工作目录中对这些文件进行编辑。

所谓的暂存区域只不过是个简单的文件，一般都放在 Git 目录中。

基本Git工作流程：

- 在工作目录修改某些文件
- 对修改的文件进行快照，然后保存到暂存区域
- 提价更新，将保存在暂存区域的文件快照永久转储在Git目录中。

## Git 基本配置
Git 提供了一个叫做 `git-config` 的工具，专门用来配置或读取相应的工作环境变量。而正是由这些环境变量，决定了 Git 在各个环节的具体工作方式和行为。这些变量可以存放在以下三个不同的地方：

`/etc/gitconfig `文件：系统中对**所有用户**都普遍适用的配置。若使用 `git config` 时用 `--system` 选项，读写的就是这个文件。
`~/.gitconfig` 文件：**用户目录**下的配置文件只适用于该用户。若使用 `git config` 时用 `--global` 选项，读写的就是这个文件。
当前项目的 git 目录中的配置文件：这里的配置仅仅针对**当前项目**有效。每一个级别的配置都会覆盖上层的相同配置，所以 `.git/config` 里的配置会覆盖 `/etc/gitconfig` 中的同名变量。

在 Windows 系统上，Git 会找寻用户主目录下的 `.gitconfig` 文件。主目录即 `$HOME` 变量指定的目录，一般都是 `C:\Documents and Settings\$USER`。此外，Git 还会尝试找寻 `/etc/gitconfig` 文件，只不过看当初 Git 装在什么目录，就以此作为根目录来定位。

**用户信息**
```
git config --global user.name "name"
git config --global user.email 1111111@qq.com
```
**文本编辑器**
```
git config --global core.editor sublime
```
**差异分析比较器**
```
git config --global merge.tool vimdiff
```
Git 可以理解 kdiff3，tkdiff，meld，xxdiff，emerge，vimdiff，gvimdiff，ecmerge，和 opendiff 等合并工具的输出信息。

**查看配置信息**
```
git config --list
```
**获取帮助**
```
git help <verb>
git <verb> --help
man git-<verb>
```

## Git 仓库
**初始化**
```
git init
```
**添加文件**
```
git add *.c
git add README
git add commit -m "initial project version"
```
### 仓库的操作
```
git clone git://github.com/schacon/grit.git
```
工作目录下面的所有文件都不外乎这两种状态：**已跟踪或未跟踪**。已跟踪的文件是指本来就被纳入版本控制管理的文件，在上次快照中有它们的记录，工作一段时间后，它们的状态可能是未更新，已修改或者已放入暂存区。而所有其他文件都属于未跟踪文件。它们既没有上次更新时的快照，也不在当前的暂存区域。初次克隆某个仓库时，工作目录中的所有文件都属于已跟踪文件，且状态为未修改。

文件的状态变化周期
![文件状态][3]

#### 检查当前文件状态
```
$ git status
    # On branch master
    nothing to commit (working directory clean)
```
有未跟踪文件时：
```
$ vim README
    $ git status
    # On branch master
    # Untracked files:
    # (use "git add <file>..." to include in what will be committed)
    #
    # README
    nothing added to commit but untracked files present (use "git add" to track)
```
>未跟踪的文件意味着Git在之前的快照（提交）中没有这些文件；Git 不会自动将之纳入跟踪范围，除非你明明白白地告诉它“我需要跟踪该文件”，因而不用担心把临时文件什么的也归入版本管理。

#### 跟踪新文件
```
git add <file>
```
```
$ git status
    # On branch master
    # Changes to be committed:
    # (use "git reset HEAD <file>..." to unstage)
    #
    # new file: README
    #
```
只要在 “Changes to be committed” 这行下面的，就说明是**已暂存**状态。如果此时提交，那么该文件此时此刻的版本将被留存在历史记录中。你可能会想起之前我们使用 git init 后就运行了 git add 命令，开始跟踪当前目录下的文件。在 git add 后面可以指明要跟踪的文件或目录路径。如果是目录的话，就说明要**递归**跟踪该目录下的所有文件。（其实 `git add` 就是把目标文件快照放入暂存区域，也就是 add file into staged area，同时未曾跟踪过的文件标记为需要跟踪。这样就好理解后续 add 操作的实际意义了。）

#### 暂存已修改文件
若修改已跟踪的文件 `benchmarks.rb`，然后在运行 status ：
```
 $ git status
On branch master

Initial commit

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

        new file:   test.txt

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   test.txt

```
文件 `benchmarks.rb` 出现在 “**Changes not staged for commit**” 这行下面，说明**已跟踪文件的内容发生了变化，但还没有放到暂存区**。可以运行`git add` 命令将已跟踪文件放入暂存区。`git add`是个**多功能**命令，根据目标文件的状态不同，此命令的效果也不同：可以用它开始跟踪新文件，或者把已跟踪的文件放到暂存区，还能用于合并时把有冲突的文件标记为已解决状态。

**注意：** 运行了 git add 之后又作了修订的文件，需要重新运行 git add 把最新版本重新暂存起来

#### 忽略某些文件
`.gitignore`用来存放忽略文件的过滤方法。
文件 .gitignore 的格式规范如下：

- 所有空行或者以注释符号 ＃ 开头的行都会被 Git 忽略。
- 可以使用标准的 glob 模式匹配。
- 匹配模式最后跟反斜杠（/）说明要忽略的是目录。
- 要忽略指定模式以外的文件或目录，可以在模式前加上惊叹号（!）取反。

**glob 模式**是指 shell 所使用的简化了的正则表达式。星号（*）匹配零个或多个任意字符；`[abc]` 匹配任何一个列在方括号中的字符（这个例子要么匹配一个 a，要么匹配一个 b，要么匹配一个 c）；问号（?）只匹配一个任意字符；如果在方括号中使用短划线分隔两个字符，表示所有在这两个字符范围内的都可以匹配（比如 [0-9] 表示匹配所有 0 到 9 的数字）。

```
# 此为注释 – 将被 Git 忽略
    # 忽略所有 .a 结尾的文件
    *.a
    # 但 lib.a 除外
    !lib.a
    # 仅仅忽略项目根目录下的 TODO 文件，不包括 subdir/TODO
    /TODO
    # 忽略 build/ 目录下的所有文件
    build/
    # 会忽略 doc/notes.txt 但不包括 doc/server/arch.txt
    doc/*.txt
```
**查看已暂存和未暂存的更新**
`git status` 列出了修改过的文件
`git diff` 使用文件补丁的格式显示具体添加和删除的行。
此命令比较的是工作目录中当前文件和暂存区域快照之间的差异，也就是修改之后还没有暂存起来的变化内容。
```
$ git diff
diff --git a/test.txt b/test.txt
index 18534df..77ba542 100644
--- a/test.txt
+++ b/test.txt
@@ -2,3 +2,8 @@ this is a test!
 ~~~
 fghdfkl
  g
+<B8>ջָ<B4><BD><A1><BF><B5> <B9>ۿ<B4>  <BF><AA><B9>ط<B3><B5>ú<DC>
+
+ghd
+ <CB><FB>
+
```
若要看已经暂存起来的文件和上次提交时的快照之间的差异，可以用 `git diff --cached` 命令。Git 1.6.1 及更高版本还允许使用 `git diff --staged`，效果是相同的。
```
$ git diff --cached
diff --git a/test.txt b/test.txt
index f862775..72825dd 100644
--- a/test.txt
+++ b/test.txt
@@ -7,4 +7,6 @@ fghdfkl
 ghd
  <CB><FB>

-<B7>ֹ<A4> <B5>ط<BD>
\ No newline at end of file
+<B7>ֹ<A4> <B5>ط<BD>
+<B4><F3><B8><C5>
+ Ⱦ<B7><A2><B8><E0>
\ No newline at end of file
```
>**注意：** 单单 `git diff` 不过是显示还没有暂存起来的改动，而不是这次工作和上次提交之间的差异。所以在`git commit`之前运行该命令，否则什么也没有。

#### 提交更新
`git commit` 使用之前一定要确认修改或新建的文件已经add过了，否则提交时不会记录这些还没暂存起来的变化。
提交时记录的是放在暂存区域的**快照**，任何还未暂存的仍然保持已修改状态，可以在下次提交时纳入版本管理。每一次运行提交操作，都是对项目作一次快照，以后可以回到这个状态，或者进行比较。

`git commit -a` 跳过暂存区域，自动把所有已经跟踪过的文件暂存起来一并提交，从而跳过 `git add` 步骤。
```
$ git commit -a -m "delete something"
[master 923cea9] delete something
 1 file changed, 4 deletions(-)
```
#### 移除文件
要从 Git 中移除某个文件，就必须要从已跟踪文件清单中移除（确切地说，是从暂存区域移除），然后提交。可以用 `git rm` 命令完成此项工作，并连带从工作目录中删除指定的文件，这样以后就不会出现在未跟踪文件清单中了。

如果只是简单地从工作目录中手工删除文件，运行 `git status` 时就会在 “Changes not staged for commit” 部分（也就是未暂存清单）看到：
```
$ git status
On branch master
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        deleted:    test.txt

no changes added to commit (use "git add" and/or "git commit -a")
```
然后再运行 git rm 记录此次移除文件的操作：
```
$ git rm test.txt
rm 'test.txt'
```
最后提交的时候，该文件就不再纳入版本管理了。**如果删除之前修改过并且已经放到暂存区域的话**，则必须要用强制删除选项 `-f`，以防误删除文件后丢失修改的内容。

如果想把文件从 Git 仓库中删除（亦即从暂存区域移除），但仍然希望保留在当前工作目录中。换句话说，仅是从跟踪清单中删除。比如一些大型日志文件或者一堆 .a 编译文件，不小心纳入仓库后，**要移除跟踪但不删除文件**，以便稍后在 .gitignore 文件中补上，用 --cached 选项即可：
```
git rm --cached readme.txt
```
也可以使用 glob 模式：
```
git rm log/\*.log
```
>**注意**: 星号 `* `之前的反斜杠 \，因为 Git 有它自己的文件模式扩展匹配方式，所以不用 shell 来帮忙展开（实际上不加反斜杠也可以运行，只不过按照 shell 扩展的话，**仅仅删除指定目录下的文件而不会递归匹配。**上面的例子本来就指定了目录，所以效果等同，但下面的例子就会用递归方式匹配，所以必须加反斜杠。）。此命令删除所有 log/ 目录下扩展名为 .log 的文件。类似的比如：
```
$ git rm \*~
```
会递归删除当前目录及其子目录中所有` ~ `结尾的文件。

#### 移动文件
不同于VCS系统，Git 并不跟踪文件移动操作。如果在 Git 中重命名了某个文件，仓库中存储的元数据并不会体现出这是一次改名操作。不过 Git 可以推断出发生了什么。
```
git mv <file_from> <file_to>
```
```
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        renamed:    test.txt -> list.txt
```
其实，运行 `git mv` 就相当于运行了下面三条命令：
```
$ mv README.txt README
$ git rm README.txt
$ git add README
```
如此分开操作，Git 也会意识到这是一次改名，所以不管何种方式都一样。

### 查看提交历史
`git log` 默认不用任何参数的话，git log 会按提交时间列出所有的更新，最近的更新排在最上面。每次更新都有一个 SHA-1 校验和、作者的名字和电子邮件地址、提交时间，最后缩进一个段落显示提交说明。

```
git log -p                  #展开显示每次提交的内容差异
git log -2                  #仅显示最近的两次更新
git log --stat              #仅显示简要的增改行数统计
git log --pretty=oneline    #将每个提交放在一行显示，另外还有 short，full 和 fuller
```
另外还可以使用`format`定制要显示的记录格式，这样的输出便于后期编程提取分析
```
$ git log --pretty=format:"%h - %an, %ar : %s"
e314300 - guoxs, 25 minutes ago : recovery test.txt
923cea9 - guoxs, 60 minutes ago : delete something
989b58b - guoxs, 61 minutes ago : change test.txt
1362de0 - guoxs, 85 minutes ago : change test.txt
87d047f - guoxs, 87 minutes ago : add .gitignore
1f0fec8 - guoxs, 2 hours ago : first commit
```
常用的格式占位符写法及其代表的意义：

|  选项  |            说明             |
| :--: | :-----------------------: |
|  %H  |    提交对象（commit）的完整哈希字串    |
|  %h  |        提交对象的简短哈希字串        |
|  %T  |     树对象（tree）的完整哈希字串      |
|  %t  |        树对象的简短哈希字串         |
|  %P  |    父对象（parent）的完整哈希字串     |
|  %p  |        父对象的简短哈希字串         |
| %an  |       作者（author）的名字       |
| %ae  |         作者的电子邮件地址         |
| %ad  | 作者修订日期（可以用 -date= 选项定制格式） |
| %ar  |     作者修订日期，按多久以前的方式显示     |
| %cn  |     提交者(committer)的名字     |
| %ce  |        提交者的电子邮件地址         |
| %cd  |           提交日期            |
| %cr  |      提交日期，按多久以前的方式显示      |
|  %s  |           提交说明            |

用 `oneline` 或 `format` 时结合 `--graph` 选项，可以看到开头多出一些 ASCII 字符串表示的简单图形，形象地展示了每个提交所在的分支及其分化衍合情况。
```
$ git log --pretty=format:"%h %s" --graph
*   96c99cd pull
|\
| * baf89e7 merge
| *   8d6c560 merge
| |\
| | *   14cffc7 Merge pull request #8 from jiangjiuxing/master
| | |\
| | | *   7519696 Merge branch 'dev'
| | | |\
| | | * \   30e5c0e Merge branch 'dev'
| | | |\ \
| | | * \ \   849ef03 Merge pull request #7 from jiangjiuxing/dev
| | | |\ \ \
| | | * \ \ \   0c3056d Merge pull request #6 from jiangjiuxing/dev
| | | |\ \ \ \
| | | * | | | | b4b1e86 remove package data
| | | * | | | |   c921cac Merge branch 'dev'
| | | |\ \ \ \ \
| | | * | | | | | d622055 add package data
| | | * | | | | | 0f78e74 delete test.txt
| | | * | | | | |   602c999 Merge pull request #5 from jiangjiuxing/dev
| | | |\ \ \ \ \ \
| | | * \ \ \ \ \ \   ac98953 merge dev
| | | |\ \ \ \ \ \ \
| | | * | | | | | | | 0450640 delete test.txt
| | | * | | | | | | |   062a1cd Merge pull request #4 from jiangjiuxing/dev
| | | |\ \ \ \ \ \ \ \
| | | * \ \ \ \ \ \ \ \   8e8c7a8 Merge pull request #2 from jiangjiuxing/dev
| | | |\ \ \ \ \ \ \ \ \
| | | * | | | | | | | | | a0d1942 reset master
| | * | | | | | | | | | | 90dd06c fix bugs
| | | |_|_|_|_|_|_|_|_|/
| | |/| | | | | | | | |
| * | | | | | | | | | | f5e744c merge
```
`git log`一些其他的命令
```
-p                      #按补丁格式显示每个更新之间的差异。
--stat                  #显示每次更新的文件修改统计信息。
--shortstat             #只显示 --stat 中最后的行数修改添加移除统计。
--name-only             #仅在提交信息后显示已修改的文件清单。
--name-status           #显示新增、修改、删除的文件清单。
--abbrev-commit         #仅显示 SHA-1 的前几个字符，而非所有的 40 个字符。
--relative-date         #使用较短的相对时间显示（比如，“2 weeks ago”）。
--graph                 #显示 ASCII 图形表示的分支合并历史。
--pretty                #使用其他格式显示历史提交信息。可用的选项包括 oneline，short，full，fuller 和 format（后跟指定格式）。
```
**限制输出长度**
除了定制输出格式的选项之外，`git log` 还有许多非常实用的限制输出长度的选项，也就是只输出部分提交信息。
```
git log -n                   # n 可以是任何自然数，表示仅显示最近的若干条提交。
git log --since=2.weeks      #按照时间作限制的选项， --since 和 --until
git log --author             #显示指定作者的提交
git log --grep               #显示指定作者的提交 如果要得到同时满足这两个选项搜索条件的提交，就必须用 --all-match 选项
git log --path
```
### 撤销操作
```
git commit --amend                      #使用当前的暂存区域快照提交
git commit --amend -m "rename_commit"   #重新编辑提交说明
```
如果刚才提交时忘了暂存某些修改，可以先补上暂存操作，然后再运行 `--amend` 提交：
```
git commit -m 'initial commit'
git add forgotten_file
git commit --amend
```
上面的三条命令最终只是产生一个提交，第二个提交命令修正了第一个的提交内容。

**取消已经暂存的文件**
```
git reset HEAD <filename>
```
**取消对文件的修改**
```
git checkout --<filename>   #把之前版本的文件复制过来重写了此文件，有一定危险性危险性
```
任何已经提交到 Git 的都可以被恢复。即便在已经删除的分支中的提交，或者用 --amend 重新改写的提交，都可以被恢复。可能失去的数据仅限于没有提交过的，对 Git 来说它们就像从未存在过一样。

## 远程仓库的使用
```
git remote                   #列出每个远程库的简短名字
git remote -v                #显示对应的克隆地址
git remote add pb <url>      #添加一个新的远程仓库，可以指定一个简单的名字，以便将来引用
git fetch pb                 #抓取所有 Pb 有的，但本地仓库没有的信息
git fetch [remote-name]      #从远程仓库抓取数据到本地
git push [remote-name] [branch-name]    #将本地仓库中的数据推送到远程仓库
git remote show [remote-name]           #查看某个远程仓库的详细信息
git remote rename               #修改某个远程仓库在本地的简称
git remote rm  [remote-name]    #移除对应的远端仓库
```

## 标签
```
git tag                     #列显已有的标签
git tag -l 'v1.4.2.*'       #用特定的搜索模式列出符合条件的标签
```
Git 使用的标签有两种类型：**轻量级的（lightweight）和含附注的（annotated）**。轻量级标签就像是个不会变化的分支，实际上它就是个指向特定提交对象的引用。而含附注标签，实际上是存储在仓库中的一个独立对象，它有自身的校验和信息，包含着标签的名字，电子邮件地址和日期，以及标签说明，标签本身也允许使用 GNU Privacy Guard (GPG) 来签署或验证。一般我们都建议使用含附注型的标签，以便保留相关信息。

**创建一个含附注类型的标签：**
```
git tag -a v1.4 -m 'my version 1.4'     #-a 指定标签名字 -m 指定了对应的标签说明
git show  v0.1                      #查看相应标签的版本信息，并连同显示打标签时的提交对象
```
**后期加注标签**
```
git tag -a v1.2 9fceb02[校验和前几位数字]
```
## 技巧与窍门
### 自动补全
下载 Git 的源代码，进入 contrib/completion 目录，会看到一个 git-completion.bash 文件。将此文件复制到你自己的用户主目录中（译注：按照下面的示例，还应改名加上点：cp git-completion.bash ~/.git-completion.bash），并把下面一行内容添加到你的 `.bashrc` 文件中：
```
source ~/.git-completion.bash
```
### Git 命令别名
用 `git config` 为命令设置别名:
```
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
```
现在，如果要输入 git commit 只需键入 git ci 即可。
使用这种技术还可以创造出新的命令，比方说取消暂存文件时的输入比较繁琐，可以自己设置一下：
```
git config --global alias.unstage 'reset HEAD --'
```
设置last:
```
git config --global alias.last 'log -1 HEAD'
```
设置图形log:
```
git config --global alias.log_graph 'log --pretty=format:"%h %s" --graph`
```
有时候我们希望运行某个外部命令，而非 Git 的子命令，只需要在命令前加上` ! `就行:
```
git config --global alias.visual '!gitk'
```

[1]: http://git.oschina.net/progit/figures/18333fig0105-tn.png
[2]: http://git.oschina.net/progit/figures/18333fig0106-tn.png
[3]: http://git.oschina.net/progit/figures/18333fig0201-tn.png
