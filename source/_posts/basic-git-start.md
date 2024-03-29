---
title: git-start
date: 2020-10-12
banner_img: /img/basic/git-binner.jpg
index_img: /img/basic/git-index.jpg
tags: 
 - git
categories:
 - basic-component
 - git
---

# Git 使用

## 前言

Git 是目前世界上被最广泛使用的现代软件版本管理系统（Version Control System）。Git 本身亦是一个成熟并处于活跃开发状态的开源项目，今天惊人数量的软件项目依赖 Git 进行版本管理，这些项目包括开源以及各种商业软件。Git 在职业软件开发者中拥有良好的声誉，Git 目前支持绝大多数的操作系统以及 IDE（Integrated Development Environments）。

Git 最初是由 Linux 操作系统内核的创造者 Linus Torvalds 在 2005 年创造，Git 第一个可用版本是 Linus 花了两周时间用C写出来的。Git 第一个版本就实现了 Git 源码自托管，一个月之内，Linux系统的源码也已经由 Git 管理了！

Git 的第一个提交源码仅有约1000行，但是已经实现了Git的基本设计原理，比如初始化仓库、提交代码、查看代码diff、读取提交信息等，Git 定义了三个区：工作区（workspace）、暂存区（index）、版本库（commit history），也实现了三类重要的 Git 对象：blob、tree、commit。本文将从源码上分析 Git 的第一个提交并挖掘背后优秀的设计原理。

## 编译

#### 获取源码

在Github上可以找到Git的仓库镜像：
		https://github.com/git/git.git

```bash
# 获取 git 源码
$ git clone https://github.com/git/git.git

# 查看第一个提交
$ git log --date-order --reverse
commit e83c5163316f89bfbde7d9ab23ca2e25604af290
Author: Linus Torvalds <torvalds@ppc970.osdl.org>
Date:   Thu Apr 7 15:13:13 2005 -0700

    Initial revision of "git", the information manager from hell

# 变更为第一个提交，指定commit-id
$ git reset --hard e83c5163316f89bfbde7d9ab23ca2e25604af290
```

#### 文件结构

```
$ tree -h
.
├── [2.4K]  cache.h
├── [ 503]  cat-file.c                  # 查看objects文件
├── [4.0K]  commit-tree.c               # 提交tree
├── [1.2K]  init-db.c                   # 初始化仓库
├── [ 970]  Makefile
├── [5.5K]  read-cache.c                # 读取当前索引文件内容
├── [8.2K]  README
├── [ 986]  read-tree.c                 # 读取tree
├── [2.0K]  show-diff.c                 # 查看diff内容
├── [5.3K]  update-cache.c              # 添加文件或目录
└── [1.4K]  write-tree.c                # 写入到tree

# 统计代码行数，总共1089行
$ find . "(" -name "*.c" -or -name "*.h" -or -name "Makefile" ")" -print | xargs wc -l
 ...
 1089 total
```

#### 编译

编译第一个提交的Git会有编译问题，需要更改Makefile添加相关的依赖库：

```bash
$ git diff ./Makefile
... 
-LIBS= -lssl
+LIBS= -lssl -lz -lcrypto
...
```

编译：

```bash
# 编译
$ make
```

只支持在 linux 平台上编译运行。

#### 源码分析

Write programs that do one thing and do it well.
——Unix philosophy

查看编译生成的可执行文件，总共有7个：

#### ![](/blog/img/git-1.jpg)

命令使用过程：

#### ![](/blog/img/git-2.jpg)

#### init-db：初始化仓库

##### **命令说明**

```bash
$ init-db
```

```
创建目录：.dircache。
创建目录：.dircache/objects。
在 .dircache/objects 中创建了从 00 ~ ff 共256个目录。

.dircache/ 是Git的工作目录，最新版本的Git工作目录为 .git/ 。

运行示例

# 运行init-db初始化仓库
$ init-db
defaulting to private storage area 
# 查看初始化后的目录结构
$ tree . -a
.
└── .dircache                   # git工作目录
    └── objects                 # objects文件
        ├── 00
        ├── 01
        ├── 02
        ├── ......              # 省略
        ├── fe
        └── ff
258 directories, 0 files
```

最新版本Git使用 git init . 初始化仓库，而且初始化工作目录为 .git/，初始化后，.git/ 目录中的文件和功能也非常丰富，包括 .git/HEAD、.git/refs/ 、.git/info/ 等，以及很多的 hooks 示例：.git/hooks/**.sample。

#### update-cache：添加文件或目录

update-cache 主要是把工作区的修改文件提交到暂存区。工作区、暂存区等说明见下文【设计原理】 。

#### 命令使用

```bash
$ update-cache <file> ...
```

#### 运行流程

读取并解析索引文件 ：.dircache/index。
遍历多个文件，读取并生成变更文件信息（文件名称、文件内容sha1值、日期、大小等），写入到索引文件中。
遍历多个文件，读取并压缩变更文件，存储到objects文件中，该文件为blob对象。

如果是刚初始化的仓库，会自动创建索引文件。索引文件说明见下文【设计原理 - 索引文件】。blob对象的文件格式及说明见下文【设计原理 - blob对象】。sha1值说明见下文【设计原理 - 哈希算法】。

#### 示例

```bash
# 新增README.md文件
$ echo "hello git" > README.md

# 提交
$ update-cache README.md

# 查看索引文件
$ hexdump -C .dircache/index
00000000  43 52 49 44 01 00 00 00  01 00 00 00 af a4 fc 8e  |CRID............|
00000010  5e 34 9d dd 31 8b 4c 8e  15 ca 32 05 5a e9 a4 c8  |^4..1.L...2.Z...|
00000020  af bd 4c 5f bf fb 41 37  af bd 4c 5f bf fb 41 37  |..L_..A7..L_..A7|
00000030  00 03 01 00 91 16 d2 04  b4 81 00 00 ee 03 00 00  |................|
00000040  ee 03 00 00 0a 00 00 00  bb 12 25 52 ab 7b 40 20  |..........%R.{@ |
00000050  b5 f6 12 cc 3b bd d5 b4  3d 1f d3 a8 09 00 52 45  |....;...=.....RE|
00000060  41 44 4d 45 2e 6d 64 00                           |ADME.md.|
00000068

# 查看objects内容，sha1值从索引文件中获取
$ cat-file bb122552ab7b4020b5f612cc3bbdd5b43d1fd3a8
temp_git_file_61uTTP: blob
$ cat ./temp_git_file_RwpU8b
hello git 
```

#### cat-file：查看objects文件内容

cat-file 根据sha1值查看暂存区中的objects文件内容。cat-file 是一个辅助工具，在正常的开发工作流中一般不会使用到。

#### 命令使用

```bash
$ cat-file <sha1>
```

#### 运行流程

根据入参sha1值定位objects文件，比如 .dircache/objects/46/4b392e2c8c7d2d13d90e6916e6d41defe8bb6a
读取该objects文件内容，解压得到真实数据。
写入到临时文件 temp_git_file_XXXXXX（随机不重复文件）。

objects内容为压缩格式，基于zlib压缩算法，objects说明见【设计原理 - objects 文件】。

#### 运行示例

```bash
# cat-file 会把内容读取到temp_git_file_rLcGKX
$ cat-file 82f8604c3652fa5762899b5ff73eb37bef2da795
temp_git_file_tBTXFM: blob

# 查看 temp_git_file_tBTXFM 文件内容
$ cat ./temp_git_file_tBTXFM 
hello git!
```

#### show-diff：查看diff内容

查看工作区和暂存区中的文件差异。

**命令使用**

```
$ show-diff
```

**运行流程**

读取并解析索引文件：.dircache/index。
循环遍历变更文件信息，比较工作区中的文件信息和索引文件中记录的文件信息差异。
无差异，显示 : ok。
有差异，调用 diff 命令输出差异内容。

**运行示例**

```
# 创建文件并提交到暂存区
$ echo "hello git!" > README.md
$ update-cache README.md

# 当前无差异
$ show-diff
README.md: ok

# 更改README.md
$ echo "hello world!" > README.md

# 查看diff
$ show-diff
README.md:  82f8604c3652fa5762899b5ff73eb37bef2da795
--- -   2020-08-31 17:33:50.047881667 +0800
+++ README.md   2020-08-31 17:33:47.827740680 +0800
@@ -1 +1 @@
-hello git!
+hello world! 
```

#### write-tree：写入到tree

write-tree 作用将保存在索引文件中的多个objects对象归并到一个类型为tree的objects文件中，该文件即Git中重要的对象：tree。

**命令使用**

```
$ write-tree
```

**运行流程**

读取并解析索引文件：.dircache/index。
循环遍历变更文件信息，按照指定格式编排变更文件信息及内容。
压缩并存储到objects文件中，该object文件为tree对象。

tree对象的文件格式及相关说明见下文【设计原理 - tree对象】。

**运行示例**

```
# 提交
$ write-tree
c771b3ab2fe3b7e43099290d3e99a3e8c414ec72

# 查看objects内容
$ cat-file  c771b3ab2fe3b7e43099290d3e99a3e8c414ec72
temp_git_file_r90ft5: tree
$ cat ./temp_git_file_r90ft5
100664 README.md��`L6R�Wb��_�>�{�-��
```

#### read-tree：读取tree

read-tree 读取并解析指定sha1值的tree对象，输出变更文件的信息。

**命令使用**

```
$ read-tree <sha1>
```

**运行步骤**

解析sha1值。
读取对应sha1值的object对象。
输出变更文件的属性、路径、sha1值。

**运行示例**

```
# 提交
$ write-tree
c771b3ab2fe3b7e43099290d3e99a3e8c414ec72

# 读取tree对象
$ read-tree  c771b3ab2fe3b7e43099290d3e99a3e8c414ec72
100664 README.md (82f8604c3652fa5762899b5ff73eb37bef2da795)
```

#### commit-tree：提交tree

commit-tree 把本地变更提交到版本库里，具体是基于一个tree对象的sha1值创建一个commit对象。

**命令使用**

```
$ commit-tree <sha1> [-p <sha1>]* < changelog
```

**运行流程**

参数解析。
获取用户名称、用户邮件、提交日期。
写入tree信息。
写入parent信息。
写入author、commiter信息。
写入comments（注释）。
压缩并存储到objects文件中，该object文件为commit对象。

commit对象的文件格式及说明见下文【设计原理 - commit对象】。

**运行示例**

```
# 写入到tree
$ write-tree 
c771b3ab2fe3b7e43099290d3e99a3e8c414ec72

# 提交tree
$ echo "first commit" > changelog
$ commit-tree c771b3ab2fe3b7e43099290d3e99a3e8c414ec72 < changelog
Committing initial tree c771b3ab2fe3b7e43099290d3e99a3e8c414ec72
7ea820bd363e24f5daa5de8028d77d88260503d9

# 查看commit对象内容
$ cat-file 7ea820bd363e24f5daa5de8028d77d88260503d9
temp_git_file_CIfJsg: commit
$ cat temp_git_file_CIfJsg
tree c771b3ab2fe3b7e43099290d3e99a3e8c414ec72
author Xiaowen Xia <chenan.xxw@aos-hw09> Tue Sep  1 10:56:16 2020
committer Xiaowen Xia <chenan.xxw@aos-hw09> Tue Sep  1 10:56:16 2020

first commit
```

#### 设计原理

> Write programs to work together.
> ——Unix philosophy

与传统的集中式版本控制系统（CVCS）相反，Git 从一开始就设计成了去中心化的分布式系统，每个开发者本地工作区都是一个完整的版本库，拥有本地的代码仓库。另外，Git 的设计初衷是为了让更多的开发者一起开发软件。

该版本 Git 定义了三种对象：

blob 对象：保存着文件快照。
tree 对象：记录着目录结构和 blob 对象索引。
commit 对象：包含着指向前述 tree 对象的指针和所有提交信息。

三种对象相互之间的关系如下：

![image.png](/blog/img/nginx-3.jpg)

另外，Git 也定义了三个区，工作区（workspace），暂存区（index）和版本库（commit history）：

- 工作区（workspace）：我们直接修改代码的地方。
- 暂存区（index）：数据暂时存放的区域，用于在工作区和版本库之间进行数据交流。
- 版本库（commit history）：存放已经提交的数据。

每个可执行文件的具体分工是：init-db 用来创建一个初始化仓库，update-cache 会将 工作区 的变更写到 索引文件 （index）中，write-tree 会将之前的所有变更整理成 tree 对象，commit-tree 会将 指定的 tree 对象写到本地版本库中。另外，show-diff 用来查看 工作区 和 暂存区 中的文件差异，read-tree 用来读取 tree对象 的信息。

由此可以绘制一个简单的Git开发工作流：

![image.png](/blog/img/nginx-4.jpg)

#### objects 文件

objects文件是载体，用来存储Git中的3个重要对象：blob、tree、commit。

objects文件的存储目录默认为.dircache/objects，也可以通过环境变量： SHA1_FILE_DIRECTORY 指定。文件路径和名称根据sha1值决定，取sha1值的第一个字节的hex值为目录，其他字节的hex值为名称，比如sha1值为：
0277ec89d7ba8c46a16d86f219b21cfe09a611e1
的对象文件存储路径为：
.dircache/objects/02/77ec89d7ba8c46a16d86f219b21cfe09a611e1

为了节约存储，同时也能存储多个信息，objects文件内容都是经过 zlib 压缩过的。objects文件的格式由 + + <要存储的内容> 组成，其中 可以是"blob"（blob对象）、"tree"（tree对象）、"commit"（commit对象）。

使用 cat-file 可以查看object文件是什么类型的对象。

.dircache/objects 目录结构如下：

```
$ tree .git/objects
.git/objects
├── 02
│   └── 77ec89d7ba8c46a16d86f219b21cfe09a611e1
├── ......                                          # 省略
├── be
│   ├── adb5bac00c74c97da7f471905ab0da8b50229c
│   └── ee7b5e8ab6ae1c0c1f3cfa2c4643aacdb30b9b
├── ......                                          # 省略
├── c9
│   └── f6098f3ba06cf96e1248e9f39270883ba0e82e
├── ......                                          # 省略
├── cf
│   ├── 631abbf3c4cec0911cb60cc307f3dce4f7a000
│   └── 9e478ab3fc98680684cc7090e84644363a4054
├── ......                                          # 省略
└── ff
```

问：为什么 .dircache/objects/ 目录下面要以sha1值前一个字节的hex值作为子目录？

#### blob 对象

运行 update-cache 会生成 blob 对象。

blob 对象用于存储变更文件内容，其实就代表一个变更文件快照。blob 对象由 + + 拼装并压缩：

![image.png](/blog/img/git-5.jpg)

使用 cat-file 查看 blob 对象内容：

```
# 查看 blob 对象内容
$ cat-file 82f8604c3652fa5762899b5ff73eb37bef2da795temp_git_file_tBTXFM: blob

$ cat ./temp_git_file_tBTXFM 
hello git!
```

#### tree 对象

运行 write-tree 会生成 tree 对象。

tree 对象用于存储多个提交文件的信息。tree 对象由 + + 文件模式 + 文件名称 + 文件sha1值 拼装并压缩：

![image.png](/blog/img/git-6.jpg)

文件sha1值 使用binary格式存储，占用20字节。

使用 cat-file 查看 tree 对象内容：

```
# 查看 tree 对象内容
$ cat-file  c771b3ab2fe3b7e43099290d3e99a3e8c414ec72
temp_git_file_r90ft5: tree

$ cat ./temp_git_file_r90ft5
100664 README.md��`L6R�Wb��_�>�{�-��
```

文件sha1值 使用binary格式存储，所以打印的时候会有乱码。

#### commit 对象

运行 commit-tree 会生成 commit 对象。

commit 对象存储一次提交的信息，包括所在的tree信息，parent信息以及提交的作者等信息。commit 对象由 + + + * + + + 拼装并压缩：

![image.png](/blog/img/git-7.jpg)

tree sha1值 和 parent sha1值 使用hex字符串格式存储，占用40字节。

使用 cat-file 查看 commit 对象内容：

```
# 查看 commit 对象内容
$ cat-file 7ea820bd363e24f5daa5de8028d77d88260503d9
temp_git_file_CIfJsg: commit

$ cat temp_git_file_CIfJsg
tree c771b3ab2fe3b7e43099290d3e99a3e8c414ec72
author Xiaowen Xia <chenan.xxw@aos-hw09> Tue Sep  1 10:56:16 2020
committer Xiaowen Xia <chenan.xxw@aos-hw09> Tue Sep  1 10:56:16 2020

first commit
```

#### 索引文件

索引文件默认路径为：.dircache/index。索引文件用来存储变更文件的相关信息，当运行 update-cache 时会添加变更文件的信息到索引文件中。

同时也有一个叫 .dircache/index.lock 的文件，该文件存在时表示当前工作区被锁定，无法进行提交操作。

使用 hexdump 命令可以查看到索引文件内容：

```
$ hexdump -C .dircache/index 
00000000  43 52 49 44 01 00 00 00  01 00 00 00 ae 73 c4 f2  |CRID.........s..|
00000010  ce 32 c9 6f 13 20 0d 56  9c e8 cf 0d d3 75 10 c8  |.2.o. .V.....u..|
00000020  94 ad 4c 5f f4 5c 42 06  94 ad 4c 5f f4 5c 42 06  |..L_.B...L_.B.|
00000030  00 03 01 00 91 16 d2 04  b4 81 00 00 ee 03 00 00  |................|
00000040  ee 03 00 00 0b 00 00 00  a3 f4 a0 66 c5 46 39 78  |...........f.F9x|
00000050  1e 30 19 a3 20 42 e3 82  84 ee 31 54 09 00 52 45  |.0.. B....1T..RE|
00000060  41 44 4d 45 2e 6d 64 00                           |ADME.md.|
```

.dircache/index 索引文件使用二进制存储相关内容，该文件由 文件头 + 变更文件信息 组成：

![image.png](/blog/img/git-8.jpg)

文件头大小为32字节，一个变更文件信息大小至少是63字节。其中：文件头中的sha1值由整个索引文件内容（文件头 + 变更文件信息）计算得到的。变更文件信息的sha1值由变更文件内容（压缩后）计算得到的。

#### 哈希算法

该 Git 版本中使用的哈希算法为 sha1算法 ，代码中使用的是 OpenSSL 库中提供的sha1算法。

目前 Git 已经有了新的选择：sha256算法 ，且目前正在做 sha1 到 sha256 的迁移。

```
#include <openssl/sha.h>

static int verify_hdr(struct cache_header *hdr, unsigned long size)
{
  SHA_CTX c;
  unsigned char sha1[20];
        /* 省略 */
  /* 计算索引文件头sha1值 */
  SHA1_Init(&c);
  SHA1_Update(&c, hdr, offsetof(struct cache_header, sha1));
  SHA1_Update(&c, hdr+1, size - sizeof(*hdr));
  SHA1_Final(sha1, &c);
  /* 省略 */
  return 0;
} 
```

### 总结与思考

> Use software leverage to your advantage.

——Unix philosophy

#### 好的代码不是写出来的，是改出来的

Git 的第一个提交中，虽然实现了 Git 的分布式核心思想，以及三种对象，三个区等核心概念，但是 Git 的灵魂功能比如分支策略、远程仓库、日志系统、git hooks 等功能都是后面逐步迭代出来的。

#### 关于细节

问：为什么 .dircache/objects/ 目录下面要以 sha1 值前一个字节的 hex 值作为子目录？

答：ext3 文件系统下，一个目录下只能有 32000 个一级子文件，如果都把 objects 文件存储到一个 .git/objects/ 目录里，很大概率会达到上限。同时要是一个目录下面子文件太多，那文件查找效率会降低很多。

#### 关于代码质量

Git 的第一次提交源码，从代码质量、数据结构上看其实并没有多少参考价值，反而我还发现了很多可以优化的地方，比如：

- 异常处理不完善，经常出现段错误（SegmentFault）。
- 存在几处内存泄漏的地方，比如 write-tree.c > main函数 > buffer内存块 。
- 从索引文件中读取到的变更文件信息使用数组存储，涉及到了比较多的申请释放操作，性能上是有损失的，可以优化成链表存储。

不过这些都不重要，重要的是 Git 的设计原理和思想。