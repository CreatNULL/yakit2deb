https://www.tecmint.com/dpkg-reconfigure-installed-package-in-ubuntu-debian/
---

**链接：如何在 Ubuntu 和 Debian 中重新配置已安装的软件包**

`dpkg-reconfigure` 是一个强大的命令行工具，用于重新配置已安装的软件包。它是 `dpkg`（Debian/Ubuntu Linux 上的核心包管理系统）提供的几个工具之一。它与 `debconf`（Debian 软件包的配置系统）协同工作。Debconf 记录了系统上所有已安装软件包的配置。

此工具实际上可用于重新配置整个 Ubuntu 或 Debian 系统安装。只需提供要重新配置的软件包名称，它就会提出一系列配置问题，方式与软件包最初安装时相同。它允许您检索已安装软件包的设置，并更改 `debconf` 中记录的该软件包的当前设置。

您可以重新配置的一类常见软件包是那些其配置由软件包安装脚本中的问题决定的软件包，这些问题通常在软件包安装过程中通过图形界面显示，例如 `phpmyadmin`。

**查看已安装软件包的配置**
要查看已安装软件包（如 "phpmyadmin"）的当前配置，请使用 `debconf-show` 实用程序：
```bash
$ sudo debconf-show phpmyadmin
```

<img width="632" height="572" alt="image" src="https://github.com/user-attachments/assets/b1421d9f-f4d3-41e2-bb7e-d2e7bea33aa0" />


**在 Debian 和 Ubuntu 中重新配置已安装的软件包**
如果您已经安装了一个软件包（例如 `phpmyadmin`），可以通过将软件包名称传递给 `dpkg-reconfigure` 来重新配置它：
```bash
$ sudo dpkg-reconfigure phpmyadmin
```
运行上述命令后，您应该能够开始重新配置 phpmyadmin。您将被问到一系列问题，选择您想要的设置并完成该过程。

<img width="852" height="572" alt="image" src="https://github.com/user-attachments/assets/b1fcead8-accc-4276-b294-d14c83e76d93" />

<img width="852" height="496" alt="image" src="https://github.com/user-attachments/assets/dbeccdd4-9519-4bee-9d1c-e4f07d987a19" />



当 phpmyadmin 重新配置过程完成后，您将看到有关新软件包设置的一些有用信息。

<img width="732" height="154" alt="image" src="https://github.com/user-attachments/assets/0f7e2c5e-4467-4a72-aed0-0553d4f5f94b" />

有一些有用的选项允许您更改其默认行为，下面解释一些实际有用的选项。

`-f` 标志用于选择要使用的前端（如 dialog、readline、Gnome、Kde、Editor 或 noninteractive）。
```bash
$ sudo dpkg-reconfigure -f readline phpmyadmin
```

您可以通过运行以下命令，使用 `debconf` 永久更改默认前端：
```bash
$ sudo dpkg-reconfigure debconf
```
使用上下键选择一个选项，按 TAB 键选择 Ok 然后按 Enter。

<img width="732" height="515" alt="image" src="https://github.com/user-attachments/assets/9730b426-508e-4ac3-b708-c12f4d261872" />

还可以根据优先级级别选择要忽略的问题,如截图所示，然后按回车。

<img width="732" height="515" alt="image" src="https://github.com/user-attachments/assets/cdcb93f7-ef02-47bf-88fa-49049c02c1f3" />

要直接从命令行指定将显示的问题的最低优先级，请使用 `-p` 选项。
```bash
$ sudo dpkg-reconfigure -p critical phpmyadmin
```

某些软件包可能处于不一致或损坏状态，在这种情况下，您可以使用 `-f` 标志强制 `dpkg-reconfigure` 重新配置软件包。请谨慎使用此标志！
```bash
$ sudo dpkg-reconfigure -f package_name
```

更多信息，请参阅 `dpkg-reconfigure` 的手册页。
```bash
$ man dpkg-reconfigure
```

以上就是全部内容！如果您对如何使用 `dpkg-reconfigure` 有任何疑问，或有任何其他想法要分享，请通过下面的评论部分联系我们。

**标签:** Ubuntu 提示

---
**用户评论：**
*   **Null n'Dvoiduvphunck:** 使用 `dpkg-reconfigure tzdata` 来覆盖 systemd 在安装时自动检测时区的功能是一个强大的终端技巧。
*   **Jim Hessin:** 询问如何获取这个工具，因为他尝试 `sudo apt-get install` 时出现“无法定位软件包”错误。

**文章作者:** Aaron Kili
**文章来源:** TecMint 网站
