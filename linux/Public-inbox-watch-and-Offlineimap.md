# offlineIMAP + public-inbox-watch 创建邮件列表镜像

使用 offlineIMAP 工具可以在本地操作 IMAP 服务器上的邮件，并在定期执行 `offlineimap`  命令时将本地操作同步到服务器端，这在多个邮件客户端（MUA）访问 IMAP 服务等场景时很有用。

public-inbox-watch 可以监测某个邮箱，将符合条件的邮件投递到 public-inbox 邮箱中，public-inbox 1.6.0 版本以上支持监测本地 maildir、nntp、nntps、imap 和 imaps。在镜像邮件列表时，通常利用邮件头的 “List-Id” 进行过滤。如果你的邮箱中有任何邮件列表完整的历史邮件，可以很方便的用 public-inbox-watch 制作镜像。

本文章目的是介绍这两个工具的使用方法，为此虚拟几个场景作为示例，可能与你的日常需求不符，但当你明白 public-inbox 的使用方法后，就可以调整为适合自己的方式。这里假设我用自己的 gmail 邮箱订阅了 linux-riscv 邮件列表，我需要将 IMAP 服务器上的邮件定期拉取到本地，用 public-inbox-watch 将来自 linux-riscv 列表的邮件保存到自己的 public-inbox 邮箱。删除一周以上的邮件以及同步到 gmail IMAP服务器。

1. 配置 offlineIMAP
2. 配置 public-inbox-watch
3. 清理 1 周以上的邮件
4. 将本地操作同步到 IMAP

## 配置 offlineIMAP

这部分我找了知乎上的一篇文章，可参考其中的 offlineIMAP 配置部分，作者讲解的非常详细，很适合新手操作：

* [Offlineimap+Mutt+Msmtp实现多account收发邮件](https://zhuanlan.zhihu.com/p/268330262?utm_source=wechat_session)

## 配置 public-inbox-watch

假设我的 gmail 邮箱同步到了本地 `~/offlineMail/gmail/` 文件夹，其中来自 linux-riscv 邮件列表的邮件在 `~/offlineMail/gmail/linux-riscv` 子文件夹下。首先新建一个空的 public-inbox ：

```
$ cd ~/Mail
$ mkdir -p public-inbox/watch-linux-riscv
$ cd public-inbox
$ public-inbox-init -V2 --ng org.infradead.lists.linux-riscv \
    watch-linux-riscv ./watch-linux-riscv http://lore.kernel.org/linux-riscv \
    watch-linux-riscv@lists.infradead.org
```

修改 config 文件中的对应部分：

```
cat ~/.public-inbox/config

[publicinbox "watch-linux-riscv"]
        address = watch-linux-riscv@lists.infradead.org
        url = http://lore.kernel.org/linux-riscv
        inboxdir = /home/user/Mail/public-inbox/watch-linux-riscv
        newsgroup = org.infradead.lists.linux-riscv
        watch = maildir:/home/user/offlineMail/gmail/linux-riscv
        watchheader = List-Id:<linux-riscv.lists.infradead.org>
```

如上新增两个参数 `watch` 和 `watchheader`，其中 `watch` 的值就是本地 maildir 格式的邮箱。watchheader 的作用是过滤，每个邮件列表有自己独有的 List-Id，因此通常用 List-Id 来过滤邮件列表。

我在更新 config 后进行了 `public-inbox-index` ，该步骤可能没必要，但做了没坏处。

```
$ public-inbox-index ./watch-linux-riscv
```

这时开一个新的终端，运行：

```
$ public-inbox-watch
I: scanning
```

就会快速扫描你的本地邮箱，重启下 `public-inbox-httpd` 服务，在 `localhost:8080/watch-linux-riscv` 链接里就可以看到镜像列表了。

如果想镜像完整的邮件列表历史，而不只是自己邮箱里已有的部分，可以尝试下面配置：

```
cat ~/.public-inbox/config

[publicinbox "watch-linux-riscv"]
        address = watch-linux-riscv@lists.infradead.org
        url = http://lore.kernel.org/linux-riscv
        inboxdir = /home/user/Mail/public-inbox/watch-linux-riscv
        newsgroup = org.infradead.lists.linux-riscv
        watch = nntp://nntp.lore.kernel.org/org.infradead.lists.linux-riscv
```

重启下 `public-inbox-watch` ，会开始镜像整个 linux-riscv 邮件列表：

```
$ public-inbox-watch
I: scanning
W:
W: <nntp://nntp.lore.kernel.org/org.infradead.lists.linux-riscv> STARTTLS tried and failed (not requested)
#
# nntp://nntp.lore.kernel.org/org.infradead.lists.linux-riscv fetching ARTICLE 1..55323
```

过程需要点时间，watch IMAP 也是类似的用法。

## 清理 1 周以上的邮件

如果定期运行 offlineimap，本地已经通过 public-inbox-watch 镜像存储了邮件。那么你可以在常用的邮件客户端（MUA）中设置自动删除一周前的邮件列表邮件，以免邮箱爆满。

如果是普通的本地 maildir 格式邮箱，可以用下面命令删除超过一周的邮件：

```
$ cd /path/to/Maildir
$ find new cur -ctime +7 -type f -print0 | xargs -0 rm -f
```

但注意该方法不适合定期 offlineimap 的方式，因为每次同步操作会更新文件创建时间。

## 将本地操作同步到 IMAP

将本地邮箱的状态同步到 IMAP 服务器只需要：

```
$ offlineimap
```

# 参考链接

本文主要参考了下面链接：

- https://zhuanlan.zhihu.com/p/268330262?utm_source=wechat_session
- https://public-inbox.org/meta

