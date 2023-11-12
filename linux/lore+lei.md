# 使用 lei 拉取感兴趣的邮件

Linux 内核邮件列表流量很高，如果你不想订阅它，可以考虑使用 lei 工具。lei 具有非常强的过滤和筛选能力，可以用 lei 只拉取自己感兴趣的邮件，从而避免了对自己没用的邮件占领你的邮箱。首先简单介绍下 lei 和 public-inbox。

## lei, public-inbox介绍

lei 的全称是 public-inbox local email interface。它是 public-inbox 的接口工具。而 public-inbox 是为了更好的托管 Linux 邮件列表归档，由 Linux 基金会支持开发的一个项目。

在过去出现过很多好的邮件列表服务项目，比如 Gmane。但大部分项目都逐渐消失或者变得不可靠，这给邮件列表的归档带来困难。public-inbox 就是为了给 Linux 内核邮件列表建立一个完整统一的归档。它的设计有几个亮点：

* 以 git 的形式存储邮件，列表中的每个邮件都是一个 git commit ，整个 git 的提交历史也就是邮件列表的历史。
* 提供基于 xapian 的高效检索方式，对内核开发者更友好。比如检索条件可以是最近两周的、有关某一个文件的修改，或是邮件正文中提到了某个关键字，收件人或者抄送者有自己的邮件等等。
* 去中心化的设计，鼓励任何人维护一份自己的邮件列表镜像。得益于基于 git 的存储方式，复制并维护一份邮件列表镜像到本地非常容易。这样还对用户使用邮件列表提供了新的方式，即可以不用真正的去订阅列表，而是以类似 `git pull` 的方式按需定期拉取邮件。同样也不需要取消订阅，不再进行 `git pull` 就达到取消订阅的效果了。
* 提供了多种外部访问的接口。比如可以开其相应的服务来通过 http，imap, nntp 阅读邮件，用 public-inbox-watch 或者 public-inbox-mda 将邮件导入 public-inbox 仓库等。lei 也是操作 public-inbox 的一个重要工具，提供过滤下载邮件等功能。

浏览器访问 https://lore.kernel.org ，就是 public-inbox 镜像提供的 http 服务，这个镜像是 Linux 内核社区维护的邮件列表主线镜像，地址是 https://erol.kernel.org/ ，在这里可以看到每个邮件列表对应的 git 仓库。大的邮件列表会被拆分成多个 git 仓库，编号从 0 开始，每个仓库约 1G 大小，存储 50000 封邮件。

# lei 的使用

lei 可以帮你从内核邮件列表下载电子邮件，并提供搜索过滤功能。筛选后的邮件可以

* 投递到本地邮箱
* 也可以同步到 IMAP 服务器上

另外，lei 可以在 https://lore.kernel.org/all/ 上操作，也可以在本地 public-inbox 镜像上操作。使用 lei 的过程一般是先定义好自己的过滤条件，然后使用一条 lei 命令下载已归档的邮件，之后可以用 `lei up` 命令随时查收新的邮件。

## lei 拉取邮件到本地邮箱

### 拉取邮件

这里用一个实际例子来简单介绍下 lei 拉取邮件的方式：

```
$ lei q -o ~/Mail/overlay -I https://lore.kernel.org/all/ -f mboxrd \
 -t '(dfn:Documentation/devicetree/dynamic-resolution-notes.rst \
 OR dfn:Documentation/devicetree/overlay-notes.rst \
 OR dfn:drivers/of/overlay.c OR dfn:drivers/of/resolver.c \
 OR dfctx:of_overlay_notifier_*) AND rt:1.month.ago..'
```

* -o 将会建立相应的文件夹，并把搜索结果下载到文件夹中
* -I 和之后的地址表示将在全部邮件列表范围内搜索，不局限于邮件所在的子列表
* -f 指定 mboxrd 格式存储邮件，默认会以 maildir 格式存储
* -t: 将下载整个 threads。这在某些场景下很有用，比如你搜索到了某封邮件里提到了你关心的一个 bug，就会把会话完整的上下文下载下来

其他是一些检索邮件的过滤条件，如：

* rt: 选定时间范围，按邮件接收时间计算。其中的 `..` 表示至今。建议第一次运行后改为"rt:1.week.ago.."
* dfn: 匹配 diff 中的文件名，也就是邮件中 patch 所改动的文件名
* dfctx: 匹配 diff context

另外常用的过滤条件还有：

* s: 匹配邮件名 Subject
* b: 匹配邮件内容
* bs: 同时匹配邮件名和邮件内容
* nq: 匹配邮件内容，但不包括引用的文字
* t: c: f: a: 依次用来匹配 To: Cc: From: 或以上三者
* l: 匹配 List-Id header，通常可以用来匹配整个邮件列表
* dfhh: 匹配 diff hunk header context，通常是 patch 修改的函数名

更详细的帮助信息可以在邮件列表链接+ `_/text/help/` 查看，比如 `https://lore.kernel.org/linux-riscv/_/text/help/`

运行这条命令后会有类似下面的提示信息：

```
# /usr/bin/curl -Sf -s -d '' https://lore.kernel.org/all/?x=m&t=1&q=(dfn%3ADocumentation%2Fdevicetree%2Fdynamic-resolution-notes.rst+%5C+OR+dfn%3ADocumentation%2Fdevicetree%2Foverlay-notes.rst+%5C+OR+dfn%3Adrivers%2Fof%2Foverlay.c+OR+dfn%3Adrivers%2Fof%2Fresolver.c+%5C+OR+dfctx%3Aof_overlay_notifier_*)+AND+rt%3A1697109859..
# https://lore.kernel.org/all/ 5/?
# https://lore.kernel.org/all/ 20/?
# https://lore.kernel.org/all/ 60/?
# https://lore.kernel.org/all/ 109/?
# https://lore.kernel.org/all/ 129/129
# 129 written to /home/xmz/Mail/test-lei/overlay-mbox (132 matches, 129 duplicates)
```

从上面信息中的 `curl`命令可以看到，lei 将参数转换为了 `http` 的访问地址，这个地址复制到浏览器中是可以访问的。接下来的信息包含匹配到邮件数量。

邮件下载好之后可以用 mutt 或者 neomutt 来阅读：

```
$ mutt -f ~/Mail/overlay
```

再举一个例子，用 lei 的方式订阅邮件列表。这里假设你想订阅 linux-block 和 linux-scsi 两个列表，可以用下面命令。后期可以通过更新搜索条件，随时增加或删除某个列表。

```
$ lei q -I https://lore.kernel.org/all/ -o ~/Mail/lists --dedupe=mid \
      '(l:linux-block.vger.kernel.org OR l:linux-scsi.vger.kernel.org) AND rt:1.week.ago..'
```

### 更新邮件

查询新来的邮件可以用：

```
$ lei up ~/Mail/overlay
```

当我们有多个 lei 邮箱时，可以一次性查询全部邮箱：

```
$ lei up --all
```

注意 `lei up` 只搜索上次运行时间之后的新邮件，如果没有新的邮件就不会下载。

### 更新搜索条件

一次新的 `lei q` 命令运行后，搜索条件会被记录下来，这时可以用 `lei ls-search` 命令来查询已保存的搜索。

```
$ lei ls-search
/home/user/Mail/overlay
/home/user/Mail/lists
```

每个记录的搜索条件可以通过下面命令修改：

```
$ lei edit-search ~/Mail/overlay
```

如果第一次运行 `lei q` 时限定的时间范围很大，比如大于一年，那么建议在 `lei up` 之前修改时间范围为“过去一周”。另外这里建议，在修改生效前可以在 [lore.kernel.org](https://lore.kernel.org) 上测试下，因为有可能出错导致下载了数万条邮件信息。

### 删除搜索

如果要删除一个 lei 搜索，可以用：

```
$ lei forget-search ~/Mail/overlay
```

注意这里的删除是指不能再用 `lei up` 更新这个邮箱，并不会删除保存在文件夹里的邮件。

## lei 拉取邮件存放到 IMAP 服务器

将邮件放到远程的 IMAP 服务器上会方便我们在多台设备上查收邮件。需要准备好自己邮箱账户的 IMAP 服务器信息，主要包括服务器地址、IMAP 服务的端口，用户名，密码。注意这里的密码可能是专用的客户端密码，这个需要根据你自己邮箱的服务器设置来确定。中科院邮件系统的服务器信息在[这里](https://help.cstnet.cn/changjianwenti/youjianshoufa/xitongcanshu.html)。

首先要设置下 git 的密码管理，可以选择使用 libsecret 或者 store。libsecret 对密码进行了加密，store 是将密码明文存储在 `~/.git-credentials` 中。运行下面命令：

```
$ git config --global credential.helper store
```

使用下面命令拉取邮件到 IMAP：

```
$ lei q -I https://lore.kernel.org/all/ -d mid \
  -o imaps://mail.codeaurora.org/MAINTAINERS <<EOF
dfn:MAINTAINERS AND rt:1.month.ago..
EOF
```

第一次运行会要求输入用户名和密码。成功运行之后就会将检索到的邮件存放在 IMAP 服务器中。注意在邮件客户端可能需要刷新或者订阅新增的文件夹，才能看到文件夹以及其中的邮件。可以用 `watch -n 600 lei up --all` 或者 systemd user time 等方式设置定期自动拉取邮件。


# 致谢

本文主要参考：

* https://people.kernel.org/monsieuricon/lore-lei-part-1-getting-started
* https://people.kernel.org/monsieuricon/lore-lei-part-2-now-with-imap

