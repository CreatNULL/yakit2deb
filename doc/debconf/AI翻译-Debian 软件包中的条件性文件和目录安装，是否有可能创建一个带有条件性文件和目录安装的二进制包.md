https://askubuntu.com/questions/642864/conditional-file-and-directory-installation-in-deb-packages
---

**Debian 软件包中的条件性文件和目录安装**

是否有可能创建一个带有条件性文件和目录安装的二进制包（例如，在 `/etc/init.d/` 中安装一个脚本，但需在得到用户确认后）？

**解决方案**

为了在软件包安装过程中交互式地提问，可以使用 **debconf**。要动态地创建和管理配置文件（`/etc/init.d/` 中的文件被视为配置文件），可以使用 **ucf**。

关于如何使用 debconf 的教程可以在这里找到：
http://www.fifi.org/doc/debconf-doc/tutorial.html

**最小示例**

**1. debconf 模板**
将此内容放入文件 `debian/templates` 中。它包含了在安装过程中向用户显示的文本。请务必将 `demo-pkg` 替换为你实际的软件包名称。

```
Template: demo-pkg/install-init.d
Type: boolean
Default: false
Description: Would you like to install a service for this package?
 Services are really cool! They allow stuff to be started in the
 background without you having to start them manually!!!
```

**2. 软件包配置脚本**
这是向用户（交互式地）询问你需要知道的信息的地方。这个脚本很特殊，因为当安装多个软件包时，所有软件包的所有这些文件都会在实际安装过程开始之前运行。这意味着，所有的问题都会在安装开始时为所有软件包一次性提出。

将以下内容放入一个名为 `debian/config` 的文件中，并将其标记为可执行（记得将 `demo-pkg` 替换为正确的软件包名称）：

```bash
#!/bin/sh

# 确保此脚本在发生任何意外错误时失败
set -e

# 加载 debconf 库
. /usr/share/debconf/confmodule

# 是否应该安装一个 init 服务？
db_input high demo-pkg/install-init.d || true
db_go
```

**3. 维护者脚本**

**postinst 脚本**（`debian/postinst`）：
```bash
#!/bin/sh
set -e
. /usr/share/debconf/confmodule

if [ "$1" = "configure" ];
then
    db_get demo-pkg/install-init.d
    if [ "${RET}" != "false" ];
    then
        # 使用 ucf 安装 init 脚本
        ucf /usr/share/demo-pkg/init-service /etc/init.d/demo-service
        # 注册并启动服务
        update-rc.d demo-service defaults
        invoke-rc.d demo-service start
    fi
fi
```

**prerm 脚本**（`debian/prerm`）：
```bash
#!/bin/sh
set -e
. /usr/share/debconf/confmodule

db_get demo-pkg/install-init.d
if [ "${RET}" != "false" ];
then
    # 在升级或移除前停止服务
    invoke-rc.d demo-service stop
    if [ "$1" = "remove" ] || [ "$1" = "deconfigure" ];
    then
        # 在移除前取消注册服务
        update-rc.d -f demo-service remove
    fi
fi
```

**postrm 脚本**（`debian/postrm`）：
```bash
#!/bin/sh
set -e

if [ "$1" = "purge" ];
then
    # 清理服务文件
    if type ucf >/dev/null 2>&1;
    then
        ucf --purge "/etc/init.d/demo-service"
    fi
    rm -f "/etc/init.d/demo-service"
fi
```

**4. 最终步骤**
1. 在 `debian/control` 文件的 `Pre-Depends:` 行中添加对 `debconf` 的依赖
2. 在 `debian/control` 文件的 `Depends:` 行中添加对 `ucf` 的常规依赖
3. 确保文件 `/usr/share/demo-pkg/init-service` 被正确安装
