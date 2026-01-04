https://serverfault.com/questions/332459/how-do-i-delete-values-from-the-debconf-database
---

**链接：debian - 如何从 debconf 数据库中删除值？- Server Fault**

**提问时间：** 13年前，10个月前
**查看次数：** 20k 次

根据这个问题，我的 gitolite 安装现在存在不正确的预填充值，导致 gitolite 安装失败。我需要从 debconf 数据库中清除几个键，但我没有看到实现这一点的方法。据我所知，这位朋友也没有找到方法。

是否可以从 debconf 数据库中清除几个值？

user67165 user67165

**回答**

```bash
echo PURGE | sudo debconf-communicate packagename
```

这将删除此软件包的所有配置，因此如果您想保留一些配置，请先使用 `debconf-get-selections` 获取它们，然后重新添加您想要保留的配置。

您可以在 Debian 打包手册中找到所有可能的操作。

除了清除特定软件包的所有问题，您还可以尝试：

```bash
echo RESET question | sudo debconf-communicate packagename
```

或者

```bash
echo UNREGISTER question | sudo debconf-communicate packagename
```

answered

9,588 1 1 金牌徽章 32 32 银牌徽章 43 43 铜牌徽章

您必须登录才能回答此问题。

**相关链接**
0 使用 debconf-set-selections 后进行清理 - 无交互安装

**相关问题**
274 如何运行 Debian stable 但从 testing 安装一些软件包？
1 debconf-set-selections 使用后是否会自动从 debconf 数据库中删除值？
0 具有多个 Web 应用程序的 Linode（Debian）盒子 - 是否可以通过 sendmail 从多个域发送电子邮件？

订阅 RSS 问题订阅
要订阅此 RSS 源，请将此 URL 复制到您的 RSS 阅读器中。

Cookie 设置
网站设计/徽标 © 2025 Stack Exchange Inc；用户贡献根据 CC BY-SA 许可。
