# lore + lei

public-inbox local email interface(lei)

lei是一个可以与lore.kernel.org对接的工具，lei可以帮你从任何列表下载电子邮件。

搜索remote资源并存储到本地
提供高效查询
可以投递到本地maildirs，或远端imap
可以查询 lore.kernel.org/all/

```
$ lei q -o ~/Mail/overlay -I https://lore.kernel.org/all/ \
 -t '(dfn:Documentation/devicetree/dynamic-resolution-notes.rst \
 OR dfn:Documentation/devicetree/overlay-notes.rst \
 OR dfn:drivers/of/overlay.c OR dfn:drivers/of/resolver.c \
 OR dfctx:of_overlay_notifier_*) AND rt:6.months.ago..'
```

-t: whole threads
rt: ..表示至今。建议第一次运行后改为"rt:1.week.ago.."
-o imaps://imap.gmail.com/lore/overlay

        lei edit-search ~/Mail/overlay

调整检索参数

        lei up ~/Mail/overlay

fetch 最新结果
删除的邮件不会被重新拉取

        lei up --all

为imap creds设置git-credential-helper

        lei ls-search
        lei forget-search ~/Mail/overlay

#### 高效查询

lei可以使用 public-inbox提供的高效搜索能力（基于Xapian），
可以定制自己非常精确的过滤条件

有关搜索的常用帮助信息可以在邮件列表链接+ `_/text/help/` 查看，
`https://lore.kernel.org/linux-riscv/_/text/help/`

更详细的搜索帮助信息可以翻阅xapian文档
`https://xapian.org/docs/queryparser.html`

```
$ lei q -o ~/Mail/riscv-inbox -I https://lore.kernel.org/all \
    -t 'dfn:fs/btrfs/* \
    OR s:btrfs-progs \
    OR dfn:drivers/block/nbd.c \
    OR tc:xingmingzheng@iscas.ac.cn) \
    AND rt:1.month.ago..'
```

dfn: match filename from diff
所有改动了 `fs/btrfs/*` 和 `drivers/block/nbd.c` 的patch

s:   match within Subject  e.g. s:"a quick brown fox"
邮件正文里提到了 `btrfs-progs` 关键词

tc:  match within the To and Cc headers
所有发送(To:)和抄送(Cc:)给我的邮件

rt:  match received time, like `d:' if sender's clock was correct
时间限定在一个月之前至今。（注意结尾的“..”）

不用订阅邮件列表的日常

        lei up ~/Mail/riscv-inbox

修改搜索条件

        lei edit-search ~/Mail/riscv-inbox

