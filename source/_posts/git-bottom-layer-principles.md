---
title: git 的内部原理
date: 2016-07-31 20:01:08
tags: git
---
从根本上来讲 Git 是一套**内容寻址** (content-addressable) 文件系统，在此之上提供了一个 VCS(版本控制) 用户界面。
## 底层命令 (Plumbing) 和高层命令 (Porcelain)
`Plimbing命令`：底层命令。用于以 UNIX 风格使用或由脚本调用。
其他的更友好的命令则被称为 `porcelain` 命令（高层命令）。

我们一般使用的 Git 命令 `checkout` `branch` `remote` 为 procelain 命令。
<!--more-->
每一个 git 仓库都有一个 .git 目录，全新的 .git 目录的文件有：
```
HEAD            #指向当前分支
branches/       #老版本有，新版本不再使用
config          #包含了项目特有的配置选项
description     #仅供 GitWeb 程序使用
hooks/          #保存了客户端或服务端钩子脚本
index           #保存了暂存区域信息
info/           #保存了一份不希望在 .gitignore 文件中管理的忽略模式 (ignored patterns) 的全局可执行文件
objects/        #存储所有数据内容
refs/           #存储指向数据 (分支) 的提交对象的指针
```
## Git 对象
git 从核心来看只是简单的存储**键值对**（key-value）。它允许插入任意类型的内容，并会返回一个键值，通过该键值可以在任何时候再取出该内容。
通过底层的`hash-object` 可以演示该过程，传一些数据给该命令，它会将数据保存在 `.git` 目录并返回表示这些数据的键值。

Git 初始化了 `objects`目录，同时在该目录下创建了 `pack` 和 `info` 子目录，但是该目录下没有其他常规文件。
可以通过以下命令往 Git 数据库中写入内容：
```
$ echo 'test content' | git hash-object -w --stdin
d670460b4b4aece5915caf5c68d12f560a9fe3e4
```
参数`-w`指示`hash-object`命令存储 (数据) 对象，若不指定这个参数该命令仅仅返回键值；
`--stdin` 指定从标准输入设备 (stdin) 来读取内容，若不指定这个参数则需指定一个要存储的文件的路径。
该命令输出长度为 40 个字符的校验和。这是个 SHA-1 哈希值──其值为要存储的数据加上一种头信息的校验和。

查看到 Git 已经存储了数据:
```
$ find .git/objects -type f
.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4
```
可以看到，Git 存储数据内容的方式是： 为每份内容生成一个文件，取得该内容与头信息的 SHA-1 校验和，创建以该校验和**前两个字符**为名称的子目录，并以 (校验和) 剩下 38 个字符为文件命名 (保存至子目录下)。

通过 `cat-file` 命令可以将数据内容取回。。传入 -p 参数可以让该命令输出数据内容的类型：
```
$ git cat-file -p d670460b4b4aece5915caf5c68d12f560a9fe3e4
test content
```
也可以直接添加文件：
```
$ echo 'version 1' > test.txt
$ git hash-object -w test.txt
83baae61804e65cc73a7201a7252750c76066a30

$ echo 'version 2' > test.txt               #写入新内容再次保存
$ git hash-object -w test.txt
1f7a7a472abf3dd9643fd615f6da379c4acb3e3a    #可以发现SHA1码变了

$ find .git/objects -type f                 #查看现在的objects内容
.git/objects/1f/7a7a472abf3dd9643fd615f6da379c4acb3e3a
.git/objects/83/baae61804e65cc73a7201a7252750c76066a30
.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4

#将文件恢复到第一版本
$ git cat-file -p 83baae61804e65cc73a7201a7252750c76066a30 > test.txt
$ cat test.txt
version 1
```
### 树对象
>Git 存储文件的形式：
>所有内容以 `tree` 或 `blob` 对象存储，其中 `tree` 对象对应于 UNIX 中的目录，`blob` 对象则大致对应于 `inodes` 或文件内容。一个单独的 `tree` 对象包含一条或多条 `tree` 记录，每一条记录含有一个指向 `blob` 或子 `tree` 对象的 `SHA-1` 指针，并附有该对象的权限模式 (mode)、类型和文件名信息。

树对象示意图：
![树对象][1]

**可以自己创建树对象：**

    通常 Git 根据你的暂存区域或 index 来创建并写入一个 tree 。因此要创建一个 tree 对象的话首先要通过将一些文件暂存从而创建一个 index 。可以使用 plumbing 命令 update-index 为一个单独文件创建一个 index　。通过该命令人为的将文件的首个版本加入到了一个新的暂存区域中。由于该文件原先并不在暂存区域中 (甚至就连暂存区域也还没被创建出来) ，必须传入 --add 参数;由于要添加的文件并不在当前目录下而是在数据库中，必须传入 --cacheinfo 参数。同时指定了文件模式，SHA-1 值和文件名：

```
$ git update-index --add --cacheinfo 100644 \
83baae61804e65cc73a7201a7252750c76066a30 test.txt
```
`100644` : 表明为普通文件
`100755` : 可执行文件
`120000` : 符号链接
这三种模式仅对 Git 中的 blob 有效。

现在可以用 `write-tree` 命令将暂存区域的内容写到一个 tree 对象了。无需 `-w` 参数 ── 如果目标 tree 不存在，调用 write-tree 会自动根据 index 状态创建一个 tree 对象。

```
 $ git write-tree
    d8329fc1cc938780ffdd9f94e0d364e0ea74f579
$ git cat-file -p d8329fc1cc938780ffdd9f94e0d364e0ea74f579
    100644 blob 83baae61804e65cc73a7201a7252750c76066a30 test.txt
    
$ git cat-file -t d8329fc1cc938780ffdd9f94e0d364e0ea74f579      #验证是否为tree对象
    tree
    
#创建一个新文件与新tree对象
$ echo 'new file' > new.txt
$ git update-index test.txt
$ git update-index --add new.txt

#创建该 tree 对象
$ git write-tree
    0155eb4229851634a0f03eb265b69f5a2d56f341
$ git cat-file -p 0155eb4229851634a0f03eb265b69f5a2d56f341
    100644 blob fa49b077972391ad58037050f2a75f74e3671e92 new.txt
    100644 blob 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a test.txt

#将一个已有的 tree 对象作为一个子 tree 读到暂存区域中
$ git read-tree --prefix=bak d8329fc1cc938780ffdd9f94e0d364e0ea74f579
$ git write-tree
    3c4e9cd789d88d8d89c1073707c3585e41b0e614
$ git cat-file -p 3c4e9cd789d88d8d89c1073707c3585e41b0e614
    040000 tree d8329fc1cc938780ffdd9f94e0d364e0ea74f579 bak
    100644 blob fa49b077972391ad58037050f2a75f74e3671e92 new.txt
    100644 blob 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a test.txt
```
 此时 tree 对象示意图：
 ![tree][2]

### commit 对象
commit 对象保存了“关于谁、何时以及为何保存了这些快照”的信息。
```
#创建 commit 对象
$ echo 'first commit' | git commit-tree d8329f
    fdf4fc3344e67ab068f836878b6c4951e3b15f3d
    
#查看 commit 对象
$ git cat-file -p fdf4fc3
    tree d8329fc1cc938780ffdd9f94e0d364e0ea74f579
    author Scott Chacon <schacon@gmail.com> 1243040974 -0700
    committer Scott Chacon <schacon@gmail.com> 1243040974 -0700

    first commit
/*
commit 对象有格式很简单：指明了该时间点项目快照的顶层树对象、作者/提交者信息（从 Git 设置的 user.name 和 user.email中获得)以及当前时间戳、一个空行，以及提交注释信息。
*/

#查看 git 历史
$ git log --stat 1a410e
     commit fdf4fc3344e67ab068f836878b6c4951e3b15f3d
    Author: Scott Chacon <schacon@gmail.com>
    Date: Fri May 22 18:09:34 2009 -0700

    first commit

    test.txt | 1 +
    1 files changed, 1 insertions(+), 0 deletions(-)
```
 blob，tree 以及 commit 对象都各自以文件的方式保存在 `.git/objects` 目录下。

 目前对象示意图：
 ![对象图][3]

### 对象存储
当存储数据内容时，同时会有一个文件头被存储起来。 Git 是如何存储对象的呢？
```
//进入ruby交互模式
$irb
    >> content = "what is up, doc?"
    => "what is up, doc?"

/*
Git 以对象类型为起始内容构造一个文件头，本例中是一个 blob
然后添加一个空格，接着是数据内容的长度，最后是一个空字节 (null byte)：
*/
    >> header = "blob #{content.length}\0"
    => "blob 16\000"

 /*
 Git 将文件头与原始数据内容拼接起来，并计算拼接后的新内容的 SHA-1 校验和。可以在 Ruby 中使用 require 语句导入 SHA1 digest 库，然后调用 Digest::SHA1.hexdigest() 方法计算字符串的 SHA-1 值：
 */   
    >> store = header + content
    => "blob 16\000what is up, doc?"
    >> require 'digest/sha1'
    => true
    >> sha1 = Digest::SHA1.hexdigest(store)
    => "bd9dbf5aae1a3862dd1526723246b20206e5fc37"

/*
Git 用 zlib 对数据内容进行压缩，在 Ruby 中可以用 zlib 库来实现。首先需要导入该库，然后用 Zlib::Deflate.deflate() 对数据进行压缩：
*/
    >> require 'zlib'
    => true
    >> zlib_content = Zlib::Deflate.deflate(store)
    => "x\234K\312\311OR04c(\317H,Q\310,V(-\320QH\311O\266\a\000_\034\a\235"

/*
最后将用 zlib 压缩后的内容写入磁盘。需要指定保存对象的路径 (SHA-1 值的头两个字符作为子目录名称，剩余 38 个字符作为文件名保存至该子目录中)。在 Ruby 中，如果子目录不存在可以用 FileUtils.mkdir_p() 函数创建它。接着用 File.open 方法打开文件，并用 write() 方法将之前压缩的内容写入该文件：
*/
    >> path = '.git/objects/' + sha1[0,2] + '/' + sha1[2,38]
    => ".git/objects/bd/9dbf5aa  e1a3862dd1526723246b20206e5fc37"
    >> require 'fileutils'
    => true
    >> FileUtils.mkdir_p(File.dirname(path))
    => ".git/objects/bd"
    >> File.open(path, 'w') { |f| f.write zlib_content }
    => 32
```
这就创建了一个正确的 blob 对象。所有的 Git 对象都以这种方式存储，惟一的区别是类型不同 ── 除了字符串 blob   ，文件头起始内容还可以是 commit 或 tree 。不过虽然 blob 几乎可以是任意内容，commit 和 tree 的数据却是有固定格式的。

## Git References
使用 SHA1 作为文件的索引是比较难记的，可以用一个简单的名字来记录这些 SHA-1 值。在 Git 中称为“引用”。可以在 `.git/refs` 目录下面可以找到这些包含 SHA-1 值的文件。
```
$ find .git/refs
    .git/refs
    .git/refs/heads
    .git/refs/tags
```
如果想要创建一个新的引用来记住最后一次提交，可以这样做：
```
echo "1a410efbd13591db07496601ebc7a059dd55cfe9" > .git/refs/heads/master
```
现在，就可以在 Git 命令中使用刚才创建的引用而不是 SHA-1 值：
```
$ git log --pretty=oneline master
    1a410efbd13591db07496601ebc7a059dd55cfe9 third commit
    cac0cab538b970a37ea1e769cbbde608743bc96d second commit
    fdf4fc3344e67ab068f836878b6c4951e3b15f3d first commit
```
基本上 Git 中的一个分支其实就是一个指向某个工作版本一条 HEAD 记录的指针或引用。你可以用这条命令创建一个指向其他提交的分支。
```
$ git update-ref refs/heads/test cac0ca
```
`update-ref` 命令可以安全的更新一个引用。
现在 Git 数据库看起来是这样：
 ![Git 数据库][4]
每当执行 `git branch` (分支名称) 这样的命令，Git 基本上就是执行 `update-ref` 命令，把现在所在分支中最后一次提交的 SHA-1 值，添加到要创建的分支的引用。

### HEAD 标记
HEAD 文件是一个指向你当前所在分支的引用标识符。这样的引用标识符其实并不包含 SHA-1 值，而是一个指向另外一个引用的指针。
```
$ cat .git/HEAD
    ref: refs/heads/master
```
如果执行 `git checkout test`，HEAD 文件也会改变为 `ref: refs/heads/test`
当再次执行 `git commit` 的时候，会创建了一个 commit 对象，把这个 commit 对象的父级设置为 HEAD 指向的引用的 SHA-1 值。

HEAD 文件安全修改命令： `symbolic-ref`
```
$ git symbolic-ref HEAD refs/heads/test
$ cat .git/HEAD
    ref: refs/heads/test
```
### Tags
Tag 对象非常像一个 commit 对象——包含一个标签，一组数据，一个消息和一个指针。最主要的区别就是 **Tag 对象指向一个 commit 而不是一个 tree**。它就像是一个分支引用，但是不会变化——永远指向同一个 commit，仅仅是提供一个更加友好的名字。

可以类似下面这样的命令建立一个 lightweight tag：
```
$ git update-ref refs/tags/v1.0 cac0cab538b970a37ea1e769cbbde608743bc96d
```
如果创建一个 annotated tag，Git 会创建一个 tag 对象，然后写入一个指向它而不是直接指向 commit 的 reference。可以这样创建一个 annotated tag（-a 参数表明这是一个 annotated tag）：
```
$ git tag -a v1.1 1a410efbd13591db07496601ebc7a059dd55cfe9 -m 'test tag'
```
### Remotes
第四种 reference 是 `remote reference`。如果你添加了一个 remote 然后推送代码过去，Git 会把你最后一次推送到这个 remote 的每个分支的值都记录在 refs/remotes 目录下。例如，你可以添加一个叫做 origin 的 remote 然后把你的 master 分支推送上去：
```
$ git remote add origin git@github.com:schacon/simplegit-progit.git
$ git push origin master
    Counting objects: 11, done.
    Compressing objects: 100% (5/5), done.
    Writing objects: 100% (7/7), 716 bytes, done.
    Total 7 (delta 2), reused 4 (delta 1)
    To git@github.com:schacon/simplegit-progit.git
    a11bef0..ca82a6d master -> master

$ cat .git/refs/remotes/origin/master
    ca82a6dff817ec66f44342007202690a93763949
```
Remote 应用和分支主要区别在于他们是不能被 checkout 的。Git 把他们当作是标记这些了这些分支在服务器上最后状态的一种书签。

## Packfiles
Git 往磁盘保存对象时默认使用的格式叫**松散对象** (loose object) 格式。Git 时不时地将这些对象打包至一个叫 `packfile` 的二进制文件以节省空间并提高效率。当仓库中有太多的松散对象，或是手工调用 `git gc` 命令，或推送至远程服务器时，Git 都会这样做。手工调用 `git gc` 命令让 Git 将库中对象打包。
```
$ git gc
    Counting objects: 17, done.
    Delta compression using 2 threads.
    Compressing objects: 100% (13/13), done.
    Writing objects: 100% (17/17), done.
    Total 17 (delta 1), reused 10 (delta 0)

$ find .git/objects -type f
    .git/objects/71/08f7ecb345ee9d0084193f147cdad4d2998293
    .git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4
    .git/objects/info/packs
    .git/objects/pack/pack-7a16e4488ae40c7d2bc56ea2bd43e25212a66c45.idx
    .git/objects/pack/pack-7a16e4488ae40c7d2bc56ea2bd43e25212a66c45.pack
```
查看一下 objects 目录，会发现大部分对象都不在了，与此同时在 pack 目录下出现了两个新文件。

仍保留着的几个对象是未被任何 commit 引用的 blob，它们没有添加至任何 commit，所以 Git 认为它们是 **"悬空"** 的，不会将它们打包进 packfile 。

剩下的文件是新创建的 `packfile` 以及一个索引。`packfile` 文件包含了刚才从文件系统中移除的所有对象。索引文件包含了 `packfile` 的**偏移**信息，这样就可以快速定位任意一个指定对象。运行 `gc` 命令前磁盘上的对象大小约为 **12K** ，而这个新生成的 `packfile` 仅为 **6K** 大小。通过打包对象减少了一半磁盘使用空间。

这是因为，Git 打包对象时，会查找命名及尺寸相近的文件，并只保存文件不同版本之间的差异内容。`git verify-pack` 命令用于显示已打包的内容。

## the Refspec
对于远程仓库连接的建立，在 `.git/config` 文件中有这样的信息：
```
[remote "origin"]
    url = git@github.com:schacon/simplegit-progit.git
    fetch = +refs/heads/*:refs/remotes/origin/*
```
其中，`Refspec` 的格式是一个可选的 **+** 号，接着是 `<src>:<dst>` 的格式，这里 `<src>` 是**远端**上的引用格式， `<dst>` 是将要记录在本地的引用格式。可选的 **+** 号告诉 Git 在即使不能快速演进的情况下，也去强制更新它。

缺省情况下 `refspec` 会被 `git remote add` 命令所自动生成， Git 会获取远端上 `refs/heads/` 下面的所有引用，并将它写入到本地的 `refs/remotes/origin/`。 所以，如果远端上有一个 master 分支，你在本地可以通过下面这种方式来访问它的历史记录：
```
$ git log origin/master
$ git log remotes/origin/master
 $ git log refs/remotes/origin/master
```
它们是等价的，因为 Git 把它们都扩展成 `refs/remotes/origin/master`

如果每次只想拉取远程的master分支，则可以修改：
```
fetch = +refs/heads/master:refs/remotes/origin/master
```
如果想一次性获取远程的多个分支：
```
$ git fetch origin master:refs/remotes/origin/mymaster \
    topic:refs/remotes/origin/topic
```
也可以修改配置文件(这里不能用通配符)：
```
[remote "origin"]
    url = git@github.com:schacon/simplegit-progit.git
    fetch = +refs/heads/master:refs/remotes/origin/master
    fetch = +refs/heads/experiment:refs/remotes/origin/experiment
```
### 推送 Refspec
推送到远程分支，可以这样:
```
$ git push origin master:refs/heads/qa/master
```
如果想让 Git 每次运行 git push origin 时都这样自动推送，可以在配置文件中添加 push 值：
```
[remote "origin"]
    url = git@github.com:schacon/simplegit-progit.git
    fetch = +refs/heads/*:refs/remotes/origin/*
    push = refs/heads/master:refs/heads/qa/master
```
### 删除引用
```
$ git push origin :topic
```
refspec 的格式是 <src>:<dst>, 通过把 <src> 部分留空的方式，这个意思是是把远程的 topic 分支变成空，也就是删除它。

## 传输协议
Git 可以以两种主要的方式跨越两个仓库传输数据：基于HTTP协议之上，和 file://, ssh://, 和 git:// 等智能传输协议。
### 哑协议
基于HTTP之上传输通常被称为哑协议，这是因为它在服务端不需要有针对 Git 特有的代码。这个获取过程仅仅是一系列GET请求，客户端可以假定服务端的Git仓库中的布局。

使用 `git clone` 做的第1件事情就是获取 `info/refs` 文件。这个文件是在服务端运行了 `update-server-info` 所生成的，所以服务端要想使用HTTP传输，必须要开启 `post-receive` 钩子。

整个过程看起来像这样：
```
$ git clone http://github.com/schacon/simplegit-progit.git
    Initialized empty Git repository in /private/tmp/simplegit-progit/.git/
    got ca82a6dff817ec66f44342007202690a93763949
    walk ca82a6dff817ec66f44342007202690a93763949
    got 085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7
    Getting alternates list for http://github.com/schacon/simplegit-progit.git
    Getting pack list for http://github.com/schacon/simplegit-progit.git
    Getting index for pack 816a9b2334da9953e530f27bcac22082a9f5b835
    Getting pack 816a9b2334da9953e530f27bcac22082a9f5b835
    which contains cfda3bf379e4f8dba8717dee55aab78aef7f4daf
    walk 085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7
    walk a11bef06a3f659402fe7563abf99ad00de2209e6
```

- 获取 `info/refs` 文件，得到一个远程引用和SHA值得列表
- 寻找HEAD引用，确定什么应该被检出到工作目录
- 开始获取对象
- 使用 zlib 解压缩，去除头部，得到 commit 内容
- 得到进一步需要获取的对象
- 抓取树对象，分别从**本仓库/替代仓库/打包文件**中查找
- 在 commit 对象上继续下一步查找
- 下载全部完成后， 将 master 分支检出工作目录

### 智能协议
这些协议在远端都有**Git智能型进程**在服务 － 它可以读出本地数据并计算出客户端所需要的，并生成合适的数据给它，这有两类传输数据的进程：一对用于**上传数据**和一对用于**下载**。

#### 上传数据
当运行 `git push origin master`, 并且 origin 被定义为一个使用SSH协议的URL时， Git 会使用 `send-pack` 进程，它会启动一个基于SSH的连接到服务器。它尝试像这样透过SSH在服务端运行命令：
```
$ ssh -x git@github.com "git-receive-pack 'schacon/simplegit-progit.git'"
    005bca82a6dff817ec66f4437202690a93763949 refs/heads/master report-status delete-refs
    003e085bb3bcb608e1e84b2432f8ecbe6306e7e7 refs/heads/topic
    0000
```
>`git-receive-pack` 命令会立即对它所拥有的每一个引用响应一行。每一行以4字节的十六进制开始，用于指定整行的长度。

这里第1行以005b开始，这在十六进制中表示91，意味着第1行有91字节长第1行也包含了服务端的能力列表（这里是 report-status 和 delete-refs）。下一行以003e起始，表示有62字节长，所以需要读剩下的62字节。再下一行是0000开始，表示服务器已完成了引用列表过程。

了解了服务器的状态，`send-pack` 进程会判断哪些 commit 是它所拥有但服务端没有的。针对每个引用，这次推送都会告诉服务端的 `receive-pack` 这个信息。
```
0085ca82a6dff817ec66f44342007202690a93763949 15027957951b64cf874c3557a0f3547bd83b3ff6
    refs/heads/master report-status
    00670000000000000000000000000000000000000000 cdfdb42577e2506715f8cfeacdbabc092bf63e8d refs/heads/experiment
    0000
```
这里的全 '0' 的SHA-1值表示之前没有过这个对象 。如果你在删除一个引用，你会看到相反的： 就是右边是全'0'。

Git 针对每个引用发送这样一行信息，就是**旧的SHA值，新的SHA值，和将要更新的引用**的名称。第1行还会包含有客户端的能力。下一步，客户端会发送一个所有那些服务端所没有的对象的一**个打包文件**。最后，服务端以成功(或者失败)来响应：
```
000Aunpack ok
```
#### 下载数据
下载数据时，客户端启动 `fetch-pack` 进程，连接至远端的 `upload-pack` 进程，以协商后续数据传输过程。

`upload-pack` 进程的启动可以有多种方式，可以使用与 `receive-pack` 相同的透过SSH管道的方式，也可以通过 Git 后台来启动这个进程，它默认监听在9418号端口上。这里 `fetch-pack` 进程在连接后像这样向后台发送数据：
```
003fgit-upload-pack schacon/simplegit-progit.git\0host=myserver.com\0
```
它也是以**4字节**指定后续字节长度的方式开始，然后是要运行的命令，和一个空字节，然后是服务端的主机名，再跟随一个最后的空字节。 Git 后台进程会检查这个命令是否可以运行，以及那个仓库是否存在，以及是否具有公开权限。如果所有检查都通过了，它会启动这个 `upload-pack` 进程并将客户端的请求移交给它。

如果透过SSH使用获取功能， `fetch-pack` 是这样的：
```
$ ssh -x git@github.com "git-upload-pack 'schacon/simplegit-progit.git'"
```
在 fetch-pack 连接之后， upload-pack 都会以这种形式返回：
```
0088ca82a6dff817ec66f44342007202690a93763949 HEAD\0multi_ack thin-pack \
    side-band side-band-64k ofs-delta shallow no-progress include-tag
    003fca82a6dff817ec66f44342007202690a93763949 refs/heads/master
    003e085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7 refs/heads/topic
    0000
```
这与 `receive-pack` 响应很类似，但是这里指的能力是不同的。而且它还会**指出HEAD引用**，让客户端可以检查是否是一份克隆。

在这里， `fetch-pack` 进程检查它自己所拥有的对象和所有它需要的对象，通过发送 "want" 和所需对象的SHA值，发送 "have" 和所有它已拥有的对象的SHA值。在列表完成时，再发送 "done" 通知 upload-pack 进程开始发送所需对象的打包文件。这个过程看起来像这样：
```
0054want ca82a6dff817ec66f44342007202690a93763949 ofs-delta
    0032have 085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7
    0000
    0009done
```
## 维护及数据恢复
### 维护
Git 会不定时地自动运行称为 "auto gc" 的命令。大部分情况下该命令什么都不处理。不过要是存在太多松散对象 (loose object, 不在 packfile 中的对象) 或 packfile，Git 会进行调用 **git gc** 命令。 gc 指垃圾收集 (garbage collect)，此命令会做很多工作：收集所有松散对象并将它们存入 packfile，合并这些 packfile 进一个大的 packfile，然后将不被任何 commit 引用并且已存在一段时间 (数月) 的对象删除。

也可以手动运行 auto gc 命令：
```
git gc --auto
```
gc 还会将所有引用 (references) 并入一个单独文件。假设仓库中包含以下分支和标签：
```
$ find .git/refs -type f
    .git/refs/heads/experiment
    .git/refs/heads/master
    .git/refs/tags/v1.0
    .git/refs/tags/v1.1
```
这时如果运行 `git gc`, `refs` 下的所有文件都会消失。Git 会将这些文件挪到 `.git/packed-refs` 文件中去以提高效率，该文件是这个样子的：
```
$ cat .git/packed-refs
    # pack-refs with: peeled
    cac0cab538b970a37ea1e769cbbde608743bc96d refs/heads/experiment
    ab1afef80fac8e34258ff41fc1b867c702daa24b refs/heads/master
    cac0cab538b970a37ea1e769cbbde608743bc96d refs/tags/v1.0
    9585191f37f7b0fb9444f35a9bf50de191beadc2 refs/tags/v1.1
    ^1a410efbd13591db07496601ebc7a059dd55cfe9
```
当更新一个引用时，Git 不会修改这个文件，而是在 refs/heads 下写入一个新文件。当查找一个引用的 SHA 时，Git 首先在 refs 目录下查找，如果未找到则到 `packed-refs` 文件中去查找。

上面文件最后以 `^` 开头的那一行。这表示该行上一行的那个标签是一个 `annotated` 标签，而该行正是那个标签所指向的 `commit` 。

### 数据恢复
恢复丢失后的 commit ，通常最快捷的办法是使用 `git reflog` 工具。当我们在一个仓库下 工作时，Git 会在我们每次修改了 `HEAD` 时悄悄地将改动记录下来。当提交或修改分支时，reflog 就会更新。`git update-ref` 命令也可以更新 reflog。运行 `git reflog` 命令可以查看当前的状态：
```
$ git reflog
    1a410ef HEAD@{0}: 1a410efbd13591db07496601ebc7a059dd55cfe9: updating HEAD
    ab1afef HEAD@{1}: ab1afef80fac8e34258ff41fc1b867c702daa24b: updating HEAD
```
运行 `git log -g` 会输出 reflog 的正常日志，从而显示更多有用信息。从而找到删除的 commit ，然后新建一个分支指向它。

如果 commit 丢失并没有记录在 reflog 中，还可以使用 `git fsck` 工具，该工具会检查仓库的数据完整性。如果指定 --ful 选项，该命令显示所有未被其他对象引用 (指向) 的所有对象。

### 移除对象
git clone 会将仓库包含的每一个文件的历史版本下载下来，若仓库中有大型文件，这将使得仓库非常大。
可以利用 `git gc` 命令查看文件占用的空间，利用 `git count-objects` 查看使用多少空间：
```
$ git gc
    Counting objects: 21, done.
    Delta compression using 2 threads.
    Compressing objects: 100% (16/16), done.
    Writing objects: 100% (21/21), done.
    Total 21 (delta 3), reused 15 (delta 1)```

$ git count-objects -v
    count: 4
    size: 16
    in-pack: 21
    packs: 1
    size-pack: 2016
    prune-packable: 0
    garbage: 0
```
使用 `git verfity-pack` 识别大对象，对输出的第三列信息即文件大小进行排序，还可以将输出定向到 tail 命令。
```
$ git verify-pack -v .git/objects/pack/pack-3f8c0...bb.idx | sort -k 3 -n | tail -3
    e3f094f522629ae358806b17daf78246c27c007b blob 1486 734 4667
    05408d195263d853f09dca71d55116663690c27c blob 12908 3478 1189
    7a9eb2fba2b1811321254ac360970fc169ba2330 blob 2056716 2056872 5401
```
比如要删除最底下那个大文件，可以运行 `rev-list`命令。若在此传入 `--objects`选项，它会列出所有 commit SHA 值，blob SHA 值及相应的文件路径。可以这样查看 blob 的文件名：
```
$ git rev-list --objects --all | grep 7a9eb2fb
    7a9eb2fba2b1811321254ac360970fc169ba2330 git.tbz2
```
接下来要将该文件从历史记录的所有 tree 中移除:
```
$ git log --pretty=oneline --branches -- git.tbz2
    da3f30d019005479c99eb4c3406225613985a1db oops - removed large tarball
    6df764092f3e7c8f5f94cbe08ee5cf42e92a0289 added git tarball
```
必须重写从 6df76 开始的所有 commit 才能将文件从 Git 历史中完全移除。需要用到 `filter-branch` 命令：
```
$ git filter-branch --index-filter \
    'git rm --cached --ignore-unmatch git.tbz2' -- 6df7640^..
    Rewrite 6df764092f3e7c8f5f94cbe08ee5cf42e92a0289 (1/2)rm 'git.tbz2'
    Rewrite da3f30d019005479c99eb4c3406225613985a1db (2/2)
    Ref 'refs/heads/master' was rewritten
```
`--index-filter` 传入一个命令修改暂存区域或索引。使用 `git rm --cached` 来从索引而不是磁盘删除文件，这样能提高速度，也可以使用 `--tree-filter` 达到相同的目的。
在这之后， `.git/refs/original` 添加的一些 refs 中仍有对它的引用，因此需要将这些引用删除并对仓库进行 repack 操作。
```
$ rm -Rf .git/refs/original
$ rm -Rf .git/logs/
$ git gc
```
在此看看空间占用：
```
$ git count-objects -v
    count: 8
    size: 2040
    in-pack: 19
    packs: 1
    size-pack: 7
    prune-packable: 0
    garbage: 0
```
repack 后仓库的大小减小到了 7K ，远小于之前的 2MB 。从 size 值可以看出大文件对象还在松散对象中，其实并没有消失，不过再进行推送或复制，这个对象不会再传送出去。如果真的要完全把这个对象删除，可以运行 `git prune --expire` 命令。

[1]: http://git.oschina.net/progit/figures/18333fig0901-tn.png
[2]: http://git.oschina.net/progit/figures/18333fig0902-tn.png
[3]: http://git.oschina.net/progit/figures/18333fig0903-tn.png
[4]: http://git.oschina.net/progit/figures/18333fig0904-tn.png