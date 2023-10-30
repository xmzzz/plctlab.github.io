# public inbox

Maildir格式不适合存储LKML，因为
 需要一个目录中存储数百万条消息
 如果分解成多个文件夹，那么搜索邮件就变的困难

public-inbox 解决了这两方面的问题，它基于git来存储邮件，因此可以高效更新和
 clone邮件列表镜像。

## public-inbox 特点

邮件列表以git仓库形式存储
高效搜索，基于xapian
搜索的过滤条件对开发者更友好
  dfn: 匹配diff中的文件名
   dfn:MAINTAINERS
  dfhh: 匹配diff hunk header context，通常是函数名
   dfhh:apic_bus_freq_is_known
  dfpost:匹配 post-image git blob ID
   dfpost:064162d062
100% 自由软件
任何人都可以维护lore.kernel.org的完整镜像并保持更新
...
作为改善内核开发流程的工具

## 怎么阅读public-inbox邮件

public-inbox 已支持NNTP 和 downloadable mbox
可参考官方一下文档：
  Documentation/design_notes.txt
  Documentation/public-inbox-v2-format.pod
  INSTALL
  README
  https://public-inbox.org/public-inbox-overview.html


下载邮件搜索结果gzipped mboxrd，用mutt阅读

```
$ curl -XPOST -OJ -HContent-Length:0 "https://lore.kernel.org/linux-pci/?q=d:20231007..&x=m"
$ mutt -f /path/to/downloaded/mbox.gz
```

其中

```
SEARCH_QUERY=d:20230301..
```
关于搜索的更多帮助可参考 [public-inbox help](https://lore.kernel.org/linux-riscv/_/text/help/)


