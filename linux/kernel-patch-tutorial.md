# Welcome to the kernel-patch-tutorial!

本文档的目标是搜集/整理所有和做**kernel upstream流程**相关的，能提高效率的（工具，方法，规则，小tips等）。

本文档**不涉及**linux kernel内核开发的技术内容。

在内核开发流程中，你在下面哪个环节遇到了问题？

* [检索上游补丁](#1-检索上游补丁)
* [开发环境与代码风格](#todo)
* 提交内核代码
* review 内核补丁
* 内核邮件列表相关
* 想知道更多提高效率的工具

# 1. 检索上游补丁

检索上游补丁主要有以下几种方式。

* 检索 git commit 历史
* 搜索邮件列表归档
* patchwork

## 1.1 检索 git commit 历史

* 下载完整历史的 kernel 源码
* 常用的 git 命令指南

### 1.1.1 下载 kernel 源码

[官方 kernel 源码仓](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/)下载速度可能会慢，在国内可以尝试下面方法加速获取kernel源码。

* [(Comming soon...):git clone linux kernel](./git-clone-linux-kernel.md)

### 1.1.2 有关搜索的 git 命令

如果你已经有了完整的linux kernel代码仓库，那么几个常用的git命令就能达到检索目的。

* [(Comming soon...):检索git commit历史](./git.md)

## 1.2 搜索邮件列表归档

linux kernel开发一直是以电子邮件的形式进行的，因此邮件列表归档里可以找到很多有价值的信息。有关邮件列表的内容会在[FIXME:内核邮件列表相关](./mailing-list.md)章节介绍，这里只提供有关搜索邮件列表归档的指南。

* google
* lore
* lei

### 1.2.1 google

(Comming soon...)

### 1.2.2 lore

lore 一般是指 https://lore.kernel.org ,这里列出了 kernel.org 托管的全部邮件列表，可以到具体的邮件列表中搜索关键字，如 [linux-riscv mailing list](https://lore.kernel.org/linux-riscv/)，也可以进入 https://lore.kernel.org/all 在全部邮件列表中搜索。

注意除了关键字搜索，lore 还支持更复杂更精确的搜索方法，该方法基于 public-inbox 的内置搜索引擎 Xapian，比如[这个搜索](https://lore.kernel.org/all/?x=m&t=1&q=dfn%3Aarch%2Friscv%2FMakefile+AND+rt%3A1697550089..)可以在全部列表中找修改了 `arch/riscv/Makefile` 文件，且收件时间在过去两周内的补丁及邮件。该方法具体可参考下面 lei 小节中的相关指南。

### 1.2.3 lei

lei (public-inbox local email interface) 是一个可以与 lore.kernel.org 对接的工具，可以帮你从任何列表中检索并下载邮件。

lei的主要特点如下：

1. 搜索基于 public-inbox 的邮件列表并下载到本地
2. 提供了高效的检索方法
3. 检索结果可以下载到本地邮箱，也可以投递到IMAP服务器

lore.kernel.org 是基于 public-inbox 的，因此可以用 lei 来检索。lore + lei 甚至可以替代订阅邮件列表，使得只关注自己感兴趣的邮件，避免对自己无用的邮件充满邮箱。相关内容可参考下面文章。

* [lore + lei: 高效检索内核邮件列表](./lore+lei.md)

## 1.3 patchwork

(Comming soon...)

可以直接通过 https://patchwork.kernel.org 进行查找、下载补丁等操作。也可以使用一些工具。

* [git-pw](https://github.com/getpatchwork/git-pw)
* [pwclient](https://github.com/getpatchwork/pwclient)

# 2. 开发环境与代码风格

内核开发过程可能会在以下场景中遇到问题

* 下载最新的 kernel 源码
* 切换到正确的内核 git 分支
* 开发过程补丁管理
* 生成补丁与测试
* 发送补丁到邮件列表

## 2.1 下载最新的 kernel 源码

(Comming soon...)

下载有完整历史的 kernel 源码，之后切换到特定版本的分支进行开发。

* [(Comming soon...):git clone linux kernel](./git-clone-linux-kernel.md)

## 2.2 选取正确的内核开发分支

[kernel的相关分支](https://git.kernel.org/)数量很多，怎么知道我应该在哪个分支开发，并且切换到该分支。

* [(Comming soon...):选取正确的内核开发分支](./select-kernel-branch.md)

## 2.3 开发过程补丁管理

(Comming soon...)

在提交补丁过程中，很可能要反复修改，发布多个版本。如果是单个补丁，用 `git commit --amend` 就可以达到修改的目的。但有多个补丁时，如何方便的修改前面已经提交的补丁？可以用 stgit 来帮助管理补丁。stgit 可以移动补丁（改变顺序）、合并、分割（split）补丁，比 b4 的功能要强大。

* https://stacked-git.github.io/
* https://stacked-git.github.io/guides/tutorial/

## 2.4 生成补丁与测试

(Comming soon...)

## 2.5 发送补丁到邮件列表

(Comming soon...)

# 3. 提交内核代码

(Comming soon...)

# 4. review 内核补丁

(Comming soon...)

# 5. 内核邮件列表相关

* 邮件相关知识
* 订阅邮件列表
* 检索邮件列表
* 本地搭建列表镜像

## 5.1 邮件相关知识

如果对邮件系统相关的概念不熟悉，可以看下这篇介绍，补一下邮件相关的计算机网络基础。


* [todo 邮件相关概念介绍](./about-email.md)

## 5.2 订阅邮件列表

传统方式订阅邮件列表会因为流量太高，导致大量未读邮件充满邮箱，因此可以考虑其他更高效的替代方式。

* [todo 传统方法订阅邮件列表](./)
* [todo 订阅 NNTP](./)
* [todo lei 拉取感兴趣的邮件](./)
* [todo 更新本地邮件列表镜像](./)

## 5.3 检索邮件列表

在线搜索可以参考 [1.2 搜索邮件列表归档](#1-2-搜索邮件列表归档)，如果本地维护了列表镜像，方法基本类似，可以参考下面文章搜索本地镜像。

* []

## 5.4 本地搭建列表镜像

本地搭建列表镜像，相当于在本地有了完整的邮件归档，对开发和邮件检索都会带来便利。

# 6. 想知道更多提高效率的工具

(Comming soon...)

----
----
（以下为待整理内容...）

# 提交内核补丁
当我们准备开发并提交内核补丁时，大致有哪些环节，每个环节中一般会遇到哪些问题。

## 拉取内核源码的正确分支

### 基于哪个分支进行开发？

### 怎么拉取该分支？

#### git checkout

### 如何快速复制整个仓库

#### 用文件系统COW特性复制整个代码仓库，btrfs snapshot, zfs clone

## 修改或开发新的代码

### 基础开发环境怎么准备？

### 什么样的代码风格才会被社区接受？

### 怎么检查自己的代码是否符合规范？

### 如何正确整理每笔提交？

### 怎么写commit message

## 生成补丁

### 怎么正确生成单个补丁？

### 怎么正确生成补丁集（多个补丁）？
#### cover letter
#### 用b4 or git format-patch (参数)

## 正确提交到邮件列表

### 如何编写邮件？

给Linux内核发送补丁，首先要配置好邮件客户端（MUA）

MUA 的配置建议翻阅下面文档，里面有详细的介绍以及常见的MUA的设置方式：
https://www.kernel.org/doc/html/latest/translations/zh_CN/process/email-clients.html

这里摘录一些相对容易出错的配置和建议

- 最好不已附件的形式发送补丁，最好把补丁作为邮件体的内嵌文本；
- 一定要使用纯文本模式，不要使用html模式；
- 小心MUA的自动换行，可能会破坏补丁格式；
- 尽量不复制粘贴补丁，因为可能会使制表符转换为空格；
- 正式发送前，建议先发给自己，并用patch命令测试

### 邮件需要包含哪些内容？

### 正确的邮件格式什么样？

[内核邮件列表礼仪](https://subspace.kernel.org/etiquette.html)

### 怎么知道邮件应该发给谁

### 怎么优雅的发送邮件补丁？

### 邮件发送后要做什么？

## 补丁管理及迭代

### 社区有了review如何回复？

### 怎么根据review修改补丁？

### 新版补丁怎么提交？

## 如何应对补丁提交后常见的几种结果

### 补丁被接收
#### 是否涉及stable
#### 是否涉及安全漏洞
#### 补丁合入后又被发现Bug怎么办
### 补丁被拒绝怎么挽救
#### 社区已有类似或这重复的补丁
#### 有已知Bug

# 检索上游补丁
##当我们想查阅或学习上游已有的补丁时，有哪些途径，以及怎么才能更高效

## 如何高效使用邮件列表

### 怎么快速找到感兴趣的补丁及邮件？
#### google
#### lore （搜索mailinglist）
#### lei

- [lore+lei:取消订阅邮件列表，只看自己感兴趣的内容](./lore+lei.md)

#### patchwork

#### FAQ

- 梳理一下邮件相关的名词、邮件列表相关工具之间的关系，有助于理解本文档

mbox, maildir

 这两个是email常见的存储格式，
 mbox将文件夹中的所有邮件存储为一个文件，而maildir的特点是每个邮件以一个单
 独的文件存储。
 lei 工具过滤并下载到本地的邮件默认以maildir格式存储。
 thunderbird 默认用mbox格式存储邮件。[对maildir格式支持尚不完善。](https://support.mozilla.org/en-US/kb/maildir-thunderbird)

MUA, MTA，MDA

 email我们虽然使用起来很方便，但邮件系统也是由几个复杂部分连接成的，

发件人:MUA --发送--> MTA -> 若干个MTA... -> MTA -> MDA <--收取-- MUA:收件人

 MUA(Mail User Agent，邮件用户代理)
  MUA就是我们日常使用的邮件客户端，如thunderbird, outlook等
 MTA(Mail Transfer Agent，邮件传输代理)
  MUA不是直接将邮件发送到收件人邮箱，而是通过MTA代为传递，sendmail和msmtp
  就是扮演MTA的角色
 MDA(Mail Delivery Agent，邮件投递代理)
  一封邮件从MUA发出后，可能通过一个或多个MTA传递后最终到达MDA，这时就会把
  邮件存放在某个文件或特殊的数据库里，这个长期保存邮件的地方就是邮箱。
 邮件到达邮箱后就原地不动等待MUA取邮件，这就是一个完整的收发邮件过程。
 
POP3, IMAP, SMTP, NNTP

 可以理解为邮件传输过程中可用的协议。
 MUA到MTA，以及MTA到MTA之间使用的协议就是SMTP协议，而收邮件时，MUA到MDA之
 间使用的协议最常用的是POP3或IMAP。

 POP3 (Post Office Protocol 3)
  POP3协议允许 MUA 下载服务器上的邮件，但是在 MUA 的操作（如移动邮件、标记
  已读等），不会反馈到服务器上。比如对thunderbird用户最直观的影响就是在本
  地建立的filter过滤条件和分类的邮箱文件夹并不会同步到服务器上。因此在另外
  一个新的客户端读取邮件就看不到这些文件夹。

 IMAP (Internet Mail Access Protocol)
  IMAP MUA收取的邮件仍然保留在服务器上，同时在MUA上的操作都会反馈到服务器上。
  该协议最大的好处就是适合多个客户端查看邮件。

 SMTP (Simple Mail Transfer Protocol)
  用来发送邮件。
  是一组用于从源地址到目的地址传输邮件的规范，通过它来控制邮件的中转方式。

 NNTP (Network News Transfer Protocol)
  网络新闻传输协议。
  thunderbird支持该协议，可以订阅内核邮件列表的NNTP服务，特点是下载邮件标
  题而不下载内容，而是在阅读的时候下载内容。

lore, lei, public-inbox, grokmirror

 https://lore.kernel.org 这里列出了全部的邮件列表，
 https://erol.kernel.org 这里列出了每个邮件列表对应的git仓库

 邮件列表的git仓库clone下来之后还不能被直接使用。那需要怎么阅读归档中的邮
 件呢？可以参考[mirroriong instructions](https://lore.kernel.org/linux-riscv/_/text/mirror/)在git仓库之上建立public inbox。
 大致过程为 public-inbox-init + public-inbox-index，
 注意每个lore.kernel.org邮件列表页面最下面都会有mirroriong instructions。

 public inbox可在本地发起httpd、imapd、nntpd等服务，相当于在本地搭建了邮件
 列表服务器。MUA可以通过imapd、nntpd等服务访问归档中的邮件，也可以直接用浏
 览器来访问httpd服务，相当于搭建了自己的lore.kernel.org。并且可以配合lei来
 检索邮件。接收新的邮件则需要定期拉取更新，类似git pull。
 [public inbox简介](./public-inbox.md)
 [用public-inbox在本地搭建内核邮件列表镜像](./public-inbox-httpd-imapd-nntpd.md)
 
 grokmirror是镜像邮件列表的辅助工具，其中的grok-pull可以方便的拉取多个列表
 仓库，并定期更新。拉取的git仓库可以用上述的public-inbox服务来查看邮件，也
 可以搭配procmail等工具自动将更新的邮件发送到你的MUA，还可以同步到IMAP服务
 器。
 [Subscribing to lore lists with grokmirror](https://people.kernel.org/monsieuricon/subscribing-to-lore-lists-with-grokmirror)

1. 我订阅的内核邮件列表流量太高了，每天甚至上千封新邮件，怎么办？

内核邮件列表在演进过程中虽然已经划分了单独子系统的列表，但是子系统列表流量
依然是非常高，正常人应该都无法接受。想象一下，埋头一周时间专心搞开发，一周
后回来查阅已经挤满了数千封未读邮件的邮箱，这很难去仔细查找对自己有用的信息。

以下方案可以帮到你：

  - [lore+lei:取消订阅邮件列表，只看自己感兴趣的内容](./lore+lei.md)
  - [Using lei, b4, and mutt to do kernel development](https://josefbacik.github.io/kernel/2021/10/18/lei-and-b4.html)

2. lore.kernel.org是在线的http服务，离线时怎么访问邮件列表归档？

lore 是基于 public-inbox 的，邮件是以git commit的形式存储，可以很方便的将
完整的邮件列表镜像归档clone到本地，并且可以开启自己的http、nntp、imap等服
务。本地维护一份邮件列表归档，会使得开发、检索信息等更加方便。

  - [参考public inbox mirroring instructions建立内核邮件列表本地镜像](./public-inbox-mirring-instructions.md)
  - [使用grok-mirror建立内核邮件列表git镜像](./grok-mirror-lore.md)
  - [grok-mirror+procmail+pi-piper自动更新本地列表并同步到你的邮箱](https://people.kernel.org/monsieuricon/subscribing-to-lore-lists-with-grokmirror)
  - [lei+mutt:检索邮件下载到本地，使用mutt阅读和review](./lei+mutt.md)
  - [public inbox的httpd,nntpd,imapd服务搭建](public-inbox-server-setup.md)
  - [thunderbird访问本地public inbox nntpd,imapd服务](thunderbird+public-inbox.md)

3. 怎么用thunderbird访问自己的public-inbox-imapd服务？

参考[thunderbird访问本地public-inbox-imapd](./thunderbird+public-inbox-imapd.md)

4. mutt快速入门（基于命令行的MUA）

- [快速配置mutt](./mutt-config.md)
- [mutt常用快捷键参考](./mutt-shortcut-keys.md)

### 怎么及时收到有关自己的邮件？
#### subscribe 邮件列表+filter
#### public-inbox + lei

# 以下工具也许对你非常有帮助

### VIM
#### 插件相关

### git
#### 配置
#### 提高upstream效率的常用命令

用alias！在gitconfig文件中，可以配置
[alias]
        st = status
        ad = add
        rt = reset
        lg = log
        fh = fetch
        ci = commit
        co = checkout
        br = branch
        dc = dcommit
        rb = rebase
        cp = cherry-pick
        fm = format-patch
        df = diff
        cl = clean
        mb = merge-base
        mg = merge
        wc = whatchanged


### git send-email 
#### 如何设置

### public-inbox ( lei )

- https://people.kernel.org/monsieuricon/lore-lei-part-1-getting-started
- https://people.kernel.org/monsieuricon/lore-lei-part-2-now-with-imap


### b4

b4 am

将补丁集下载保存为.mbox，运用Reviewed-by

        b4 am <message-id>

or 直接打补丁

        b4 am -o - <message-id> | git am

`-m ~/Mail` 可以从本地存储的邮件中获取
配置文件存储在git config

        -g, --guess-base

缺少base-commit信息时，该参数可以用来猜测base

        -3, --prep-3way

准备3way合并
[(什么是3way？)](https://www.zhihu.com/question/30200228)

        -c, --check-newer-revisions

检查patch是否有更新

更多用法可参考 b4 am --help

b4 mbox

        b4 mbox -fo ~/Mail <msgid>

下载整个thread，存储到mailbox
过滤mailbox中重复的邮件
与mutt配合，抓取整个thread。适合有人在thread中Cc你

        macro index 4 "<pipe-message>b4 mbox -fo ~/Mail<return>"
        // press 4 to fetch the rest of the thread from lore

b4 diff

显示patch两个不同版本之间的range-diff
需要index包含在git tree中

        b4 diff <msgid> -v 3 5

#### 维护者： mbox, am, diff, pr, ty

#### 贡献者： 发补丁，生成补丁，开发过程中用于管理版本
https://b4.docs.kernel.org/en/latest/index.html

https://people.kernel.org/monsieuricon/sending-a-kernel-patch-with-b4-part-1
https://people.kernel.org/monsieuricon/end-to-end-patch-attestation-with-patatt-and-b4


### stgit
在提交补丁过程中，很可能要反复修改，发布多个版本，这时可以用stgit来管理补丁，比b4的功能要强大
单个补丁用git commit --amend就可以达到修改的目的

但有多个补丁时，如何方便的修改前面已经提交的补丁，移动补丁（改变顺序），合并，分割（split）补丁？stgit可以

https://stacked-git.github.io/
https://stacked-git.github.io/guides/tutorial/

### git-pw pwclient
通过patchwork查找，下载补丁，等操作

https://github.com/getpatchwork/git-pw
https://github.com/getpatchwork/pwclient

### mutt

### nntp
#### 订阅lore nntp 邮件列表

### msmtp

# 内核源码树文档快速索引
内核源码树中的文档相当详细，并且质量非常高，以下内容可以帮你快速找到自己需要的文档。

## 有关如何成为内核开发者以及如何融入社区
- Documentation/process/howto.rst
## 有关内核开发的整个流程以及可能遇到的问题
- Documentation/process/development-process.rst
## 开源书籍、出版读物、高质量参考资源
- Documentation/process/kernel-docs.rst

# 值得阅读的资源|博客|站点

- https://people.kernel.org/monsieuricon
