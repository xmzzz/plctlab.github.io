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

在提交补丁过程中，很可能要反复修改，发布多个版本。如果是单个补丁，用 `git commit --amend` 就可以达到修改的目的。但有多个补丁时，如何方便的修改前面已经提交的补丁？可以用 stgit 来帮助管理补丁。stgit 可以移动补丁（改变顺序）、合并、分割（split）补丁，比 b4 的功能要强大。

* https://stacked-git.github.io/
* https://stacked-git.github.io/guides/tutorial/

# 5. 内核邮件列表相关

* 邮件相关知识
* 订阅邮件列表
* 检索邮件列表
* 本地搭建列表镜像

## 5.1 邮件相关知识

如果对邮件系统相关的概念不熟悉，可以看下这篇介绍，补一下邮件相关的计算机网络基础。

* [邮件相关概念介绍](./about-email.md)

## 5.2 订阅邮件列表

传统方式订阅邮件列表会因为流量太高，导致大量未读邮件充满邮箱，因此可以考虑其他更高效的替代方式。

* [lei 拉取感兴趣的邮件](./lore+lei.md)

## 5.4 本地搭建列表镜像

本地搭建列表镜像，相当于在本地有了完整的邮件归档，对开发和邮件检索都会带来便利。

* [用 public-inbox 搭建内核邮件列表本地镜像](./public-inbox.md)
* [用 Grokmirror 管理镜像集群](./grokmirror.md)
* [用 offlineIMAP + public-inbox-watch 镜像邮件列表](./offlineIMAP+public-inbox-watch.md)
* [用 procmail + public-inbox-mda 镜像邮件列表](./procmail+public-inbox-mda.md)
