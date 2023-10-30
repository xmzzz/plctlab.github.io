## public-inbox-config 参考

```
[publicinbox "linux-riscv-xmz"]
        address = linux-riscv@lists.infradead.org
        url = http://lore.kernel.org/linux-riscv
        inboxdir = /home/xmz/archive/mirror-lists/linux-riscv
        newsgroup = lists.linux-riscv
[publicinbox "tools-xmz"]
        address = tools@linux.kernel.org
        url = https://lore.kernel.org/tools
        inboxdir = /home/xmz/archive/mirror-lists/tools
        newsgroup = lists.tools
```

- PS: 以上config中的 newsgroup 参数值在IMAP服务中表示server端的存储目录

```
$ public-inbox-imapd
# bound imap://0.0.0.0:143
PID=13846 is worker[0]
```
## thunderbird 中操作

Settings -> Account Settings -> Account Actions -> Add Mail Account

  Your full name: xxx           ; 必填
  Email Address: lists@tools    ; 该项内容需满足邮箱格式才能进行下一步
  点击下方出现的 `Configure manually` -> 点击下侧的 `Advanced config`
  弹出提示菜单"...Do you want to proceed?" -> OK

PS:
1. 以下步骤建议按顺序执行，以免自动开启IMAP服务器端所有邮件的下载任务
2. 建议按如下方法每个mailling list单独开一个Account，否则目录显示会很乱

Synchronisation & Storage ->
  < > Keep messages in all folders for this account on this computer (unchecked)
  该选项会自动检索IMAP服务器所有邮件并下载到本地

Disk Space 按需选择
  经测试 "Synchronise the most recent xx Days" 选项无效
  仍会自动下载文件夹内全部邮件到本地
  "Don't download messages larger than xx kB"  选项有效

Server Settings ->
  Server Settings ->
    Advanced.. ->
      IMAP server directory: lists.linux-riscv  (填public-inbox-config中的newsgroup值)
      < > Show only subscribed folders          (unchecked)
    OK

Server Settings ->
  Server Name: 192.168.0.11                     (localhost IP or IMAP server IP)
  Port: 143
  -> Restart

IMAP password 任意填

public-inbox 会将大的邮件列表分成多个文件夹存储，编号最大的文件夹为最新的邮件仓库。

PS:
  预览邮件文件夹会自动下载该文件夹内的全部Header，并开始indexing
  大的邮件列表中该过程会耗时并占用CPU。
  https://support.mozilla.org/en-US/kb/rebuilding-global-database

