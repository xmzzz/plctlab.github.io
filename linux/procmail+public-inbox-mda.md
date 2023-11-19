# 用 procmail + public-inbox-mda 镜像邮件列表

public-inbox-mda 可以方便的投递邮件到自己的 public-inbox 邮箱，是作为 MDA (Mail Delivery Agent，邮件投递代理) 的角色，一般需要配合 procmail 等工具来用。如果你熟悉邮件系统相关知识的话，配置起来很简单。如果还不熟悉，可以先阅读[邮件相关概念介绍](./about-email.md)。

我们选择使用 Grokmirror 工具来自动更新本地镜像，这里假设你已经成功配置了 [Grokmirror 管理镜像集群](./grokmirror.md)，如果还没有，可以先读下该文章。

1. 配置 procmail
2. 配置 grokmirror
3. 配置 grok-pi-piper
4. 配置 public-inbox-mda
5. 运行 grok-pull

## 配置 procmail

首先在你的发行版上安装 procmail 。[procmail](https://en.wikipedia.org/wiki/Procmail) 是一个邮件投递代理程序，可以设置详细的过滤条件对邮件进行分类，分别投递到不同的邮箱里。也可以配合 [SpamAssassin](https://en.wikipedia.org/wiki/Apache_SpamAssassin) 进行垃圾邮件过滤。本篇文章里我们只用来将 grokmirror 同步的新邮件投递到本地邮箱，所以配置简单，如果对其他配置方法感兴趣，可以参考 `man procmailrc` 。

```
$ cat ~/.procmailrc

# default mailbox
DEFAULT=/home/user/offlineMail/gmail/mda-linux-riscv

# log
LOGFILE=$HOME/.procmail.log
VERBOSE='yes'
LOGABSTRACT='all'

# filter
:0 Whbc: .msgid.lock
| /path/to/public-inbox-mda --no-precheck

```

作为演示的例子，上面配置中我多用了一个 offlineIMAP 配置的本地邮箱作为默认邮箱，配置方法可参考 [offlineIMAP + public-inbox-watch 创建邮件列表镜像](./offlineIMAP+public-inbox-watch.md) 。之后收到的每封邮件除投递到 public-inbox 邮箱之外，还会有一封副本投递到该默认邮箱。过滤部分的配置说明可以参考 `man procmailrc` 。

## 配置 grokmirror

参考 [用 Grokmirror 管理镜像集群](./grokmirror.md) 中的配置，增加一个 hook：

```
$ cd ~/Mail/grokmirror
$ cat lore.conf

[core]
toplevel = ~/Mail/grokmirror/lore
log = ${toplevel}/grokmirror.log

[remote]
site = https://lore.kernel.org
manifest = https://lore.kernel.org/manifest.js.gz

[pull]
post_update_hook = /path/to/grok-pi-piper -c ~/.config/pi-piper.conf
refresh = 300
include = /linux-riscv/*
          /soc/*
          /stable-rt/*
```

注意修改下你环境中 `grok-pi-piper` 的路径。

## 配置 grok-pi-piper

在 `post_update_hook` 中调用 `grok-pi-piper` 可以在镜像更新时将新邮件重定向到任意命令，具体命令可以在 `conf` 文件中定义。下面建立该配置文件：

```
$ cat ~/.config/pi-piper.conf

[DEFAULT]
pipe = /path/to/procmail

[stable-rt]
shallow = yes

[soc]
shallow = yes
```

这里注意上面配置里的 `shallow = yes` ，public-inbox 镜像中以 git commit 的方式存储了全部归档的历史邮件，会占用一定的磁盘空间。如果不想保存完整的历史，而只是通过 grok-pi-piper 转送到 procmail 和 public-inbox 邮箱，那么可以设置 `shallow = yes`，这将只保留最后一个 git commit，会节省空间。可以在 `[DEFAULT]` 中设置作用于全部邮件列表，也可以像上面例子中单独指定某个邮件列表。后期如果想找回删除的历史，可以在邮件列表仓库中执行 `git fetch _grokmirror master --unshallow` 。

## 配置 public-inbox-mda

首先建立空的 public-inbox 邮箱，这里以 linux-riscv 列表为例，其他列表类似：

```
$ cd ~/Mail
$ mkdir -p public-inbox/mda-linux-riscv
$ cd public-inbox
$ public-inbox-init -V2 --ng org.infradead.lists.linux-riscv \
    mda-linux-riscv ./mda-linux-riscv http://lore.kernel.org/linux-riscv \
    mda-linux-riscv@lists.infradead.org
```

修改 public-inbox config 对应部分：

```
cat ~/.public-inbox/config

[publicinbox "mda-linux-riscv"]
        address = mda-linux-riscv@lists.infradead.org
        url = http://lore.kernel.org/linux-riscv
        inboxdir = /home/user/Mail/public-inbox/mda-linux-riscv
        newsgroup = org.infradead.lists.linux-riscv
        listid = linux-riscv.lists.infradead.org
```

上面的配置中新加的 `listid` 参数就是用来指定 `public-inbox-mda` 的投递条件，效果是将来自 linux-riscv 列表的邮件投递到该邮箱。

## 运行 grok-pull

以上配置之后，运行 grok-pull 就可以每 300 秒拉取一次镜像更新，将新邮件投递到 publi-inbox 邮箱中：

```
$ grok-pull -o -c ~/.config/lore.conf
```

也可以用 systemd 来管理，参考 [用 Grokmirror 管理镜像集群](./grokmirror.md) 中的相关章节。

# 总结与致谢

本文主要参考了 [Subscribing to lore lists with grokmirror](https://people.kernel.org/monsieuricon/subscribing-to-lore-lists-with-grokmirror) ，感谢作者。作者在文章中详细介绍了 grokmirror + procmail 实现订阅邮件列表的效果，邮件会保存在本地邮箱。我在上面介绍的方法稍有区别，除了投递到本地邮箱之外，增加了使用 public-inbox-mda 投递到 public-inbox 邮箱。这样就可以通过 public-inbox-httpd 等服务实时看到邮件列表的更新，完成了镜像邮件列表的效果。
