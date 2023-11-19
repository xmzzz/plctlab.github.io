# 用 public inbox 搭建内核邮件列表本地镜像

如果对 public inbox 还不熟，可以先看下 [public inbox 的简单介绍](./lore+lei.md#lei-public-inbox介绍) 。

## clone && init

首先 clone 镜像仓库到本地，用 public-inbox-init 进行初始化。这部分步骤可以参考 [lore](https://lore.kernel.org) 每个列表页面最下方的 [mirroring instructions](https://lore.kernel.org/linux-riscv/_/text/mirror/)。这里用一个相对较小的 tools 列表为例。

```
$ mkdir ~/Mail && cd ~/Mail
$ git clone --mirror http://lore.kernel.org/tools/0 tools/git/0.git
$
$ # If you have public-inbox 1.1+ installed, you may
$ # initialize and index your mirror using the following commands:
$
$ public-inbox-init -V2 --ng org.kernel.linux.mylist-tools \
$   mylist-tools ./tools http://lore.kernel.org/tools \
$   tools@linux.kernel.org
$ public-inbox-index ./tools
```

`public-inbox-init` 的格式是：

```
public-inbox-init -V2 NAME INBOX_DIR MY_URL LIST_ADDRESS
```

这会在 `~/.public-inbox/config` 生成配置信息：

```
[publicinbox "mylist-tools"]
        address = mylist-tools@linux.kernel.org
        url = http://lore.kernel.org/tools
        inboxdir = /home/xmz/Mail/mail/tools
        newsgroup = org.kernel.linux.mylist-tools
```

这里简单说明下:

“mylist-tools” 是本地列表的名字，这也是后面用 http 访问时地址里要用到的，可以任意。
address 和 url 如果本地镜像不对外提供服务，暂时也可以任意填写。其中 address 会在 http 访问时显示在页面顶部。
inboxdir 是 clone 到本地的镜像地址。
newsgroup 是在订阅 nntp 服务时显示的目录结构，同时也是 IMAP 服务上的邮件目录结构。

## 启动 http、imap、nntp 服务

启动这三种服务只需要用到 public-inbox 的三个命令：

* public-inbox-httpd
* public-inbox-imapd
* public-inbox-nntpd

如果遇到了小于 1024 端口缺少权限的问题，可以先设置如下：

```
echo 0 > /proc/sys/net/ipv4/ip_unprivileged_port_start
```

如果是用 systemd ，可以如下设置：

```
# cat /etc/sysctl.d/local.conf
net.ipv4.ip_unprivileged_port_start = 0
```

也可以用 `-l` 指定端口号：

```
$ public-inbox-nntpd -l 0.0.0.0:7119
```

另外，为了方便同时部署多种协议的服务，public inbox 提供了 `public-inbox-netd` ：

```
$ public-inbox-netd -l http://0.0.0.0:7111 -l nntp://0.0.0.0:7112 -l imap://0.0.0.0:7113
# bound http://0.0.0.0:7111
# bound nntp://0.0.0.0:7112
# bound imap://0.0.0.0:7113
PID=953371 is worker[0]
```

这样就同时开启了三种服务。

### http 服务

只需要在主机上运行 `public-inbox-httpd` 命令：

```
$ public-inbox-httpd
# bound http://0.0.0.0:8080
PID=890256 is worker[0]
```

若出现类似上面提示，就表示已经开启了 public-inbox 的 httpd 服务。我们可以通过本地的浏览器输入地址 `localhost:8080/mylist-tools` 来访问本地列表。界面和 [lore.kernel.org/tools](https://lore.kernel.org/tools) 上基本一致，除去地址里自定义的 `mylist-tools` 和页面第一行的 `public inbox for mylist-tools@linux.kernel.org`。

也可以在局域网里通过地址 `ip-address:8080/mylist-tools` 来访问该服务。

### nntp 服务

只需要在主机上运行 `public-inbox-nntpd` 命令：

```
$ public-inbox-nntpd
# bound nntp://0.0.0.0:119
PID=903805 is worker[0]
```

默认开启的 119 号端口，可以通过邮件客户端（MUA）来订阅该 nntpd 服务。这里以 thunderbird 为例。下面的步骤在 thunderbird 中操作

```
thunderbird

Settings -> Account Settings -> Account Actions -> Add Newsgroup Account
-> Your Name: 和 Email Address: （暂时可任意填写）-> Next
 -> Newsgroup Server: (localhost 或 ip 地址) -> Next
  -> Account Name: mylist-tools-nntp -> Next
```

以上过程就建立好 nntp 账户，下面需要进行订阅：

```
thunderbird inbox

mylist-tools-nntp 账户 -> 右键选择 Subscribe...
 -> 勾选 org.kernel.linux.mylist-tools <*>
  -> OK
```

订阅后就会出现 o.k.l.mylist-tools 文件夹，在这里就可以看到邮件。第一次进入文件夹可能会提示你将下载 500 封邮件，根据你的设置而不同。注意这里下载的只是标题，不包括内容，内容在阅读邮件的时候才会下载。如果想下载到本地进行离线访问，需要进行下面设置：

```
thunderbird

Settings -> Account Settings -> mylist-tools-nntp
-> Synchronisation & Storage
 -> Message Synchronising
  -> Select newsgroups for offline use...
   -> 勾选该邮件列表
    -> OK
```

### imap 服务

只需要在主机上运行 `public-inbox-imapd`：

```
$ public-inbox-imapd
# bound imap://0.0.0.0:143
PID=907994 is worker[0]
```

IMAP 服务默认开启 143 号端口，是以 IMAP 匿名模式开启的服务，因此密码可以任意或者不填。接下来可以在 Thunderbird 中访问该服务：

```
Settings -> Account Settings -> Account Actions -> Add Mail Account

  Your full name: xxx           ; 必填
  Email Address: lists@tools    ; 该项内容需满足邮箱格式才能进行下一步
  -> 点击下方出现的 `Configure manually`
   -> 点击下侧的 `Advanced config`
    -> 弹出提示菜单 "...Do you want to proceed?"
     -> OK
```

注意:

1. 以下步骤建议按顺序进行，以免自动开启 IMAP 服务器端所有邮件的下载任务
2. 建议按如下方法每个 mailling list 单独开一个 Account，不然目录显示会很乱

```
Synchronisation & Storage ->
  < > Keep messages in all folders for this account on this computer (unchecked)
  该选项会自动检索IMAP服务器所有邮件并下载到本地，因此先关闭

Disk Space 按需选择
  经测试 "Synchronise the most recent xx Days" 选项似乎无效，仍会自动下载文件夹内全部邮件到本地
  "Don't download messages larger than xx kB"  选项有效

Server Settings
 -> Server Settings
  -> Advanced..
   -> IMAP server directory: lists.linux-riscv  (填 public-inbox-config 中的 newsgroup 值)
      < > Show only subscribed folders (unchecked)
       -> OK

Server Settings ->
  Server Name: 192.168.0.11 (localhost IP or IMAP server IP)
  Port: 143
  -> Restart thunderbird
```

IMAP password 不填或任意。public-inbox 会将大的邮件列表分成多个文件夹存储，编号最大的文件夹为最新的邮件仓库。

注意:
* 预览邮件文件夹会自动下载该文件夹内的全部邮件标题，并开始 [indexing 操作](https://support.mozilla.org/en-US/kb/rebuilding-global-database)。indexing 是为了建立索引来优化邮件的搜索，大的邮件列表该过程会耗时并占用 CPU。

## 更新镜像仓库

接下来我们就可以按需定期来拉取更新，从而查收到最新的邮件。我试过两种更新镜像的方式：

* git fetch 或者 git remote update
* public-inbox-fetch

### git fetch 或者 git remote update

public-inbox-v2-format 将大的邮件列表分成了多个 EPOCH，从 0 开始按数字排序。用该方法更新镜像理论上只更新当前的 EPOCH：

```
$ git --git-dir=tools/git/0.git fetch

或者

$ cd tools/git/0.git
$ git remote update
Fetching origin
...
```

这时如果有更新 git 就会拉取 `0.git` 的更新，更新后需要进行 index：

```
$ public-inbox-index
```

index 之后，通过 http 刷新下页面就可以看到新的邮件了，imap、nntp 需要检查下新的邮件，或者根据设置自动定期检查。

### public-inbox-fetch

先介绍下另一种拉取镜像的方式，用法是 `$ public-inbox-clone URL`：

```
$ cd ~/Mail
$ public-inbox-clone https://lore.kernel.org/tools

# /usr/bin/curl -Sf -R -o m-D4uX.tmp https://lore.kernel.org/tools/manifest.js.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   178  100   178    0     0    138      0  0:00:01  0:00:01 --:--:--   138
# /usr/bin/curl -Sf --compressed -R -o tools/inbox.config.example-QYbZ https://lore.kernel.org/tools/_/text/config/raw
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   329  100   329    0     0    251      0  0:00:01  0:00:01 --:--:--   251
# git clone --mirror https://lore.kernel.org/tools/git/0.git tools/git/0.git
Cloning into bare repository 'tools/git/0.git'...
remote: Enumerating objects: 4638, done.
remote: Total 4638 (delta 0), reused 0 (delta 0), pack-reused 4638
Receiving objects: 100% (4638/4638), 6.84 MiB | 127.00 KiB/s, done.
Resolving deltas: 100% (473/473), done.
# /usr/bin/curl -Sf --compressed -R -o tools/description-wb9d https://lore.kernel.org/tools/description
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    64    0    64    0     0     60      0 --:--:--  0:00:01 --:--:--    60
# mirrored https://lore.kernel.org/tools/ => tools
```

与 `git clone` 方法类似，拉取镜像后需要进行 `public-inbox-init` 和 `public-inbox-index`，参考上面小节。更新镜像时可以用：

```
$ cd tools/
$ public-inbox-fetch
$ public-inbox-index
```

或：

```
$ cd tools/
$ make update
```

注意 `public-inbox-fetch` 会检查是否有新的 EPOCH，因此可能会有报错信息，应该是正常的，测试也没发现异常现象。下面是我拉取 linux-riscv 列表的报错信息：

```
$ cd linux-riscv/
$ public-inbox-fetch

# inbox URL: http://lore.kernel.org/linux-riscv/
# /usr/bin/curl -Sf -R -o m-PaF6.tmp http://lore.kernel.org/linux-riscv/manifest.js.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   162  100   162    0     0     26      0  0:00:06  0:00:06 --:--:--    33
# git --git-dir=/home/xmz/Mail/mail/fetch-linux-riscv/git/0.git fetch
warning: redirecting to https://lore.kernel.org/linux-riscv/0/
remote: Enumerating objects: 138, done.
remote: Counting objects: 100% (15/15), done.
remote: Compressing objects: 100% (10/10), done.
remote: Total 138 (delta 1), reused 0 (delta 0), pack-reused 123
Receiving objects: 100% (138/138), 298.72 KiB | 1024 bytes/s, done.
Resolving deltas: 100% (7/7), done.
From http://lore.kernel.org/linux-riscv/0
   ca89ca9d6..d80f276a2  master     -> master
# git clone --mirror http://lore.kernel.org/linux-riscv/git/1.git /home/xmz/Mail/mail/fetch-linux-riscv/git/1.git
Cloning into bare repository '/home/xmz/Mail/mail/fetch-linux-riscv/git/1.git'...
remote: Not Found
fatal: repository 'http://lore.kernel.org/linux-riscv/git/1.git/' not found
error: could not lock config file /home/xmz/Mail/mail/fetch-linux-riscv/git/1.git/config: No such file or directory
git config -f /home/xmz/Mail/mail/fetch-linux-riscv/git/1.git/config include.path ../../all.git/config failed: $?=65280

$ public-inbox-index
```

## 了解更多有关 public inbox

摘录一下 public inbox 文档中有关工作流的内容，可以更加了解 public inbox：

```
# public-inbox data flow
#
# Note: choose either "delivery tools" OR "git mirroring tools"
# for a given inboxdir.  Using them simultaneously for the
# SAME inboxdir will cause conflicts.  Of course, different
# inboxdirs may choose different means of getting mail into them.
# You may fork any inbox by starting with "git mirroring tools",
# and switching to "delivery tools".

                                                 +--------------------+
                                                 |  delivery tools:   |
                                                 |  public-inbox-mda  |
                                                 | public-inbox-watch |
                                                 | public-inbox-learn |
                                                 +--------------------+
                                                   |
                                                   |
                                                   v
+----------------------+                         +--------------------+
| git mirroring tools: |                         |                    |
| public-inbox-clone,  |                         |                    |
| public-inbox-fetch,  |  git (clone|fetch) &&   |      inboxdir      |
|      grok-pull,      |  public-inbox-index     |                    |
|   various scripts    | ----------------------> |                    |
+----------------------+                         +--------------------+
                                                   |
                                                   |
                                                   v
                                                 +--------------------+
                                                 | read-only daemons: |
                                                 | public-inbox-netd  |
                                                 | public-inbox-httpd |
                                                 | public-inbox-imapd |
                                                 | public-inbox-nntpd |
                                                 +--------------------+
```


更多有关 public inbox 的内容，可以访问邮件列表： https://public-inbox.org/meta/ 

public-inbox 的文档在这里： https://public-inbox.org/ 其中可以先重点看下：

* public-inbox-overview.txt
* design_notes.txt
* public-inbox-v2-format.txt
* INSTALL
* README
* flow.txt

