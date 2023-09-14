Welcome to the kernel-patch-tutorial!

目标为搜集/整理：
   所有和做kernel upstream相关的，能提高效率的（工具，流程，方法，规则，小tips）


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

### 邮件需要包含哪些内容？

### 正确的邮件格式什么样？

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
### google
#### lore （搜索mailinglist）
#### lei
#### patchwork

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

https://people.kernel.org/monsieuricon/lore-lei-part-1-getting-started
https://people.kernel.org/monsieuricon/lore-lei-part-2-now-with-imap


### b4
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
