https://stackoverflow.com/questions/10885177/how-to-read-input-while-installing-debian-package-on-debian-systems
---

**如何在 Debian 系统上安装 Debian 软件包时读取输入**


我创建了一个小的 Debian 软件包，它需要接收用户的输入并将其打印出来。

为了接收用户输入，在 postinst 脚本中使用 "read" 命令在 Debian 系统上不起作用，我不清楚具体原因，但它在 Ubuntu 系统上可以工作。

后来我发现对于 Debian 系统，必须使用 "debconf" 并配合一个模板文件。

模板文件：
```
Template: test/input
Type: text
Description: 输入一些文本，这些文本将会被显示
```

postinst 脚本：
```
db_get test/input
echo "you have entered ::$RET" >&2
```

但是当我安装我的测试包时，我遇到了这个错误：
```
Can't exec "postinst": No such file or directory at /usr/share/perl/5.10/IPC/Open3.pm line 168.
open2: exec of postinst configure failed at /usr/share/perl5/Debconf/ConfModule.pm line 59
```

有谁知道我做错了什么吗？


编辑于 2015年4月3日 19:36
mins 的用户头像
mins
7,7821313 枚金徽章7171 枚银徽章8787 枚铜徽章
提问于 2012年6月4日 17:04
forum.test17 的用户头像
forum.test17
2,22977 枚金徽章3434 枚银徽章6363 枚铜徽章
我已经解决了自己的问题，我遗漏了 config 脚本并且避免了在 config 脚本中使用 echo 语句
– 
forum.test17
 评论于 2012年6月12日 9:18
添加评论

**1 个回答**
按默认排序方式：

得票最高的答案（默认）
1

你的 postinst 脚本应该如下所示：
```bash
#!/bin/bash

set -e

. /usr/share/debconf/confmodule

case "$1" in
  configure)
    db_get test/input
    echo "you have entered ::$RET" >&2
  ;;
esac
db_stop
```
分享
改进此回答
关注
