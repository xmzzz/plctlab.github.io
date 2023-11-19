# 用 Grokmirror 管理镜像集群

grok|korg – Grokmirror is mirror of korg

[内核邮件列表仓库](https://erol.kernel.org/)与[内核仓库](https://git.kernel.org/)都是 git 仓库集群。如果我们本地维护了一份镜像集群，为了使本地集群与 master 保持同步，一种方法是用计划任务 `crond` 定时执行 `git pull`。但这样做会遇到问题，比如 master 集群如果新增了新的仓库，本地是不知道的。另外多个内核仓库之间是存在引用关系的，镜像同步时这部分重复的内容就会浪费存储空间和带宽。

[Grokmirror](https://github.com/mricon/grokmirror) 是一个 git 镜像集群管理工具。对于上面提到的问题，Grokmirror 使用 manifest 文件登记 git 仓库集群中需要镜像的仓库，mirror 用户根据配置定期拉取这个 manifest 文件，并与本地的 manifest 文件对比记录数和各条记录的时间戳，如果 master 有更新或者有新增仓库，那么就会自动更新本地的 manifest，使其与 master 保持同步，并且根据更新后的manifest，自动同步各个仓库。下面就来用 Grokmirror 管理本地邮件列表镜像。

## 安装 grokmirror

如果你的发行版有这个包，可以用包管理器安装。也可以用 pip 安装，这里用的 grokmirror 是 2.0.11 版本：

```
$ pip install --user grokmirror
```

## 配置 grokmirror

在工作目录中新建配置文件，步骤和内容如下：

```
$ cd ~/Mail/grokmirror
$ mkdir lore
$ vim lore.conf
$ cat lore.conf

[core]
toplevel = ~/Mail/grokmirror/lore
log = ${toplevel}/grokmirror.log

[remote]
site = https://lore.kernel.org
manifest = https://lore.kernel.org/manifest.js.gz

[pull]
refresh = 300
include = /linux-riscv/*
          /soc/*
          /stable-rt/*
```

简单说明下上述配置：

* `toplevel` 是存放镜像的根目录，必须存在，提前建好
* `refresh` 指定自动更新的频率，单位是秒
* 上面用了三个子列表为例，linux-riscv、soc、stable-rt

可用的邮件列表可以在 [lore.kernel.org](https://lore.kernel.org) 查询，也可以在 [erol.kernel.org](https://erol.kernel.org)看到对应的 git 仓库。

建议下载 grokmirror 源码仓，这里面有配置文件模板，注释很详细：

```
$ git clone git@github.com:mricon/grokmirror.git
$ cat grokmirror/grokmirror.conf
```
## 运行 grok-pull

可以用下面命令运行 grok-pull，如果按照上面配置加了 `refresh` 参数，这将产生一个持续的前台进程，每隔五分钟更新一次，并在前台打印 log。如果没有该参数，就会只执行一次更新就结束：

```
$ grok-pull -v -c lore.conf
```

加 `&` 符号可以放在后台运行：

```
$ grok-pull -v -c lore.conf &
```

另外还可以借助 systemd 等方便管理：

```
$ cat ~/.config/systemd/user/grok-pull@.service

[Unit]
Description=Grok-pull service for %I
ConditionPathExists=%h/Mail/%i.conf

[Service]
ExecStart=%h/.local/bin/grok-pull -o -c %h/Mail/%i.conf
Type=simple
Restart=on-failure

[Install]
WantedBy=default.target


$ systemctl --user enable grok-pull@lore
$ systemctl --user start grok-pull@lore
```

后期如果 `lore.conf` 文件有改动，比如增加了新的邮件列表，需要重启服务：

```
$ systemctl --user restart grok-pull@lore
```

## 查看本地邮件列表

上面我们做的是拉取并定时更新邮件列表仓库镜像，但镜像是 git 仓库的形式，还不能直接看到里面的邮件，如果要看邮件，下面几种方法可以尝试：

1. public-inbox-init + index + netd
2. offlineIMAP + public-inbox-watch 跟踪本地邮箱
3. procmail + public-inbox-mda 将邮件投递到自己的邮箱

### public-inbox-init + index + netd

git 镜像仓库可以按照 [用 public inbox 搭建内核邮件列表本地镜像](./public-inbox.md) 提到的方法进行初始化，建立索引，发起 http 或者 netd 服务，以 linux-riscv 列表为例，具体可以参考该文档：

```
$ cd ~/Mail/grokmirror/lore
$ public-inbox-init -V2 --ng org.infradead.lists.linux-riscv \
    linux-riscv ./linux-riscv http://lore.kernel.org/linux-riscv \
    linux-riscv@lists.infradead.org
$ public-inbox-index ./linux-riscv
$ public-inbox-httpd

# bound http://0.0.0.0:8080
PID=7644 is worker[0]
ignoring SIGWINCH since we are not daemonized
```

这时通过浏览器访问 `ip:8080/linux-riscv` 就可以看到邮件。也可以用 thunderbird + public-inbox-imapd or public-inbox-nntpd 等方式访问邮件。这种方法需要注意的是，每次要想看到更新的邮件，都需要进行一次 `public-inbox-index` 操作，可以放到 `lore.conf` 中的 `hook` 参数中自动进行：

```
$ cat lore.conf

[core]
toplevel = ~/Mail/grokmirror/lore
log = ${toplevel}/grokmirror.log

[remote]
site = https://lore.kernel.org
manifest = https://lore.kernel.org/manifest.js.gz

[pull]
refresh = 300
include = /linux-riscv/*
          /soc/*
          /stable-rt/*

post_work_complete_hook = /path/to/public-inbox-index ${toplevel}/linux-riscv ${toplevel}/soc ${toplevel}/stable-rt

$ systemctl --user restart grok-pull@lore
```

[grokmirror v2.1.0 版本](https://github.com/mricon/grokmirror/blob/master/CHANGELOG.rst) 新增了 grok-pi-indexer，应该可以更方便的完成 index 步骤，另外还可以用 public-inbox-extindex 来完成，但这两种方式我没试过，感兴趣可以参考下 man page。

### offlineIMAP + public-inbox-watch 跟踪本地邮箱

public-inbox-watch 可以跟踪一个本地邮箱的变化，自动更新到 public-inbox 邮箱中，在用 http 等方式可以实时看到更新。该本地邮箱配合 offlineIMAP 可以方便的同步到自己的 IMAP 服务器上，便于多个客户端查收邮件。具体见下面文章：

[offlineIMAP + public-inbox-watch 跟踪本地邮箱](./offlineIMAP+public-inbox-watch.md)

### procmail + public-inbox-mda 将邮件投递到自己的邮箱

public-inbox-mda 和 public-inbox-watch 都常用于镜像一个邮件列表，与 public-inbox-watch 不同的是，public-inbox-mda 是每来一封新邮件做一次邮件投递操作，通常配合 procmail 等工具使用。public-inbox-watch 是一个持久的进程，会持续扫描整个本地邮箱的变动，这非常适合在一个已经有大量邮件的邮箱做镜像的场景。procmail + public-inbox-mda 的使用方法见下面文章：

[procmail + public-inbox-mda 将邮件投递到自己的邮箱](./procmail+public-inbox-mda.md)

# 致谢

本文主要参考了下面文章：

- https://lwn.net/Articles/748184/
- https://people.kernel.org/monsieuricon/subscribing-to-lore-lists-with-grokmirror
- https://people.kernel.org/monsieuricon/mirroring-lore-kernel-org

