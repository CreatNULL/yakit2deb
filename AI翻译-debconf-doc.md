http://www.fifi.org/doc/debconf-doc/tutorial.html

# Debconf 程序员教程

乔伊·赫斯
Debian 项目
joeyh@debian.org

版权所有 © 1999, 2000 乔伊·赫斯

本文遵循 BSD 许可证条款（无广告条款）由作者版权所有。

## 目录
- 简介
- 开始使用
- 模板文件
- 配置脚本
- 修改现有的维护者脚本
- 收尾工作
- 测试
- 故障排除
- 高级主题
- A. 命令
- B. 问题层次结构

## 简介
本指南介绍了如何在您的软件包中使用 debconf，面向 Debian 开发者。

那么，什么是 debconf？简单来说（详细规范可在 Debian 政策文档中找到），debconf 是一个后端数据库，前端与之通信并向用户呈现界面。前端可以有多种类型，从纯文本到网页前端。前端还与 debian 软件包控制部分中的特殊配置脚本通信，并且通过特殊协议与 postinst 脚本及其他脚本通信。这些脚本告诉前端它们需要从数据库获取哪些值，如果这些值尚未设置，前端就会询问用户。

您的软件包需要向用户输出信息或询问问题时，就应该使用 debconf。本教程假设您已有一个需要转换使用 debconf 的软件包。

## 开始使用
首先，您的软件包必须依赖（或预依赖，如果其在 preinst[1] 中使用 debconf）于 debconf。因为 debconf 不是必需（Essential）软件包。

第一步是查看您的 postinst，以及 postinst 调用的任何程序（如 "packageconfig" 程序）、preinst，甚至 prerm 和 postrm。记录它们可能生成的所有输出和提示用户的所有输入。要让您的软件包使用 debconf，必须消除所有这些输出和输入。（stderr 的输出可以保留原样。）

注意：如果您的 preinst 使用 debconf，必须让软件包预依赖（Pre-Depend）于 debconf (>= 0.2.17)。

例如，一个假设的软件包 "foo" 有以下 postinst：

```bash
#!/bin/sh -e
echo -n "Do you like debian? [yn] "
read like
case "$like" in
n*|N*)
    echo "Poor misguided one. Why are you installing this package, then?"
    /etc/init.d/subliminal_messages start "I like debian."
;;
esac
```

它明显会询问一个问题，有时还会输出一条信息。在本教程中，我们将使其使用 debconf 来完成这两项任务。

## 模板文件
开始编写一个 `debian/templates` 文件。每当发现一个输出或问题时，就将其作为一个新模板添加到文件中。此文件格式简单，类似于 Debian 控制文件：

```
Template: 软件包名/某物
Type: [select, multiselect, string, boolean, note, text, password]
Default: [可选默认值]
Description: 简短描述？
 详细描述。详细描述。详细描述。详细描述？详细
 描述。详细描述。详细描述。
 .
 另一段详细描述。详细描述。详细描述。详细描述。详细描述。
 详细描述。

Template: ....
....
```

**表 1. 可用数据类型**

| 类型 | 描述 |
| :--- | :--- |
| string | 保存任意字符串数据。 |
| boolean | 保存 "true" 或 "false"。 |
| select | 保存有限个可能值之一。必须在名为 `Choices:` 的字段中指定可能值。用逗号和空格分隔，例如：`Choices: yes, no, maybe` |
| multiselect | 类似 select 类型，但用户可以从列表中选择任意数量的项目。这意味着 `Default:` 字段和问题的实际值可能是一个用逗号和空格分隔的值列表，就像 `Choices:` 字段一样。注意：为了兼容旧版 Debconf，如果使用此数据类型，请让您的软件包依赖 debconf (>= 0.2.26) |
| note | 此模板是显示给用户的注释。与 text 类型不同，它很重要，用户确实应该看到。如果 debconf 非交互式运行，它可能会被保存到日志文件或邮箱供用户稍后查看。 |
| text | 此模板是显示给用户的一段文本。旨在用于大多数装饰性原因，围绕同时提出的其他问题进行修饰。与 note 不同，它不被视为用户必须看到的内容。较简单的前端可能拒绝显示此类元素。 |
| password | 保存密码。谨慎使用。请注意用户输入的密码将被写入 debconf 的数据库。您应考虑尽快从数据库中清除该值。 |

注意 `Description` 字段有两部分：简短描述和详细描述。请注意，有些前端并不总是显示详细描述，通常只在用户请求额外帮助时才显示。因此简短描述应完全独立。

如果您想不出详细描述，那么，先多想想。发帖到 debian-devel 寻求帮助。参加写作课！详细描述很重要。如果绞尽脑汁后仍一无所获，就留空。重复简短描述没有意义。

详细描述中的文本会自动换行，除非其前缀了额外的空格（超出所需的一个空格）。您可以通过在段落之间单独一行放置 " ." 来将它们分成不同的段落。

在我们的示例中，我们创建一个包含两个模板的模板文件：

```
Template: foo/like_debian
Type: boolean
Description: Do you like Debian?
 We'd like to know if you like the Debian GNU/Linux system.

Template: foo/why_debian_is_great
Type: note
Description: Poor misguided one. Why are you installing this package?
 Debian is great. As you continue using Debian, we hope you will
 discover the error in your ways.
```

### 本地化模板文件
稍后，您可能希望为模板文件添加翻译。这是通过添加更多包含翻译文本的字段来实现的。任何字段都可以被翻译。例如，您可能想将描述翻译成西班牙语。只需创建一个名为 `Description-es` [2] 的字段，其中包含翻译文本。当然，如果翻译字段不可用，它将回退到普通字段名。

请注意，您甚至可以翻译 select 或 multiselect 问题的 `Choices` 字段。如果这样做，您应列出与主 `Choices` 字段中相同顺序的相同选项。顺序保持一致至关重要。您无需翻译 select 或 multiselect 问题的 `Default` 字段 [3]，显示问题时返回的答案将始终是英文。

由于您可能无法将模板文件翻译成所有所需语言，因此需要一些外部帮助。请对译者友善，他们需要在您更新模板文件时得知，而无需手动持续检查，帮助他们最简单的方法是将翻译后的模板保存在单独的文件中，例如意大利语翻译保存为 `templates.it`。因此，当意大利语译者首次翻译您的模板文件时，他可以运行：

```bash
debconf-getlang it templates > templates.it
```

并更改所有可翻译字段。然后您要做的就是将此文件与您的模板文件一起放在 debian 子目录中。`debconf-mergetemplate` 程序可以将多个此类模板文件合并，以生成放入二进制软件包的组合文件。[4]

您可以通过运行以下命令检查是否所有字符串都已翻译：

```bash
debconf-getlang --stats templates templates.it
```

当您更新英文文本时，请勿修改翻译的模板文件。译者可以通过运行上述命令检查其翻译是否过时，并通过运行以下命令合并更改：

```bash
debconf-getlang it templates templates.it > new.it
```

模糊字段会以其名称添加 `-fuzzy` 进行标记，并且原始字段（无值）会以单行形式插入在其前面。译者根据英文文本更新此类字段，删除这些标记，并向您提供更新版本。同样，您要做的就是将此翻译后的模板文件以正确的名称与您的模板文件一起放在 debian 子目录中。

## 配置脚本
接下来，确定询问问题的顺序和向用户显示信息的顺序，找出在询问问题和显示信息之前要进行的测试，然后开始编写一个 `debian/config` 脚本来询问和显示它们。

注意：这些问题由一个单独的配置脚本询问，而不是由 postinst 询问，因此可以在安装软件包之前配置，或在安装后重新配置。**不要**让您的 postinst 使用 debconf 来提问。

根据您选择编写 `debian/config` 的语言，您有一些与前端通信的选择：

**Shell 脚本**
您可以导入 `/usr/share/debconf/confmodule`，这将使一些 shell 函数可供使用。每个 shell 函数对应协议中的一个命令（小写，名称前加上 "db_"）。您向它传递参数，并通过 `$RET` 变量获取结果。

注意：某些命令还会返回一个数字退出码，以指示失败和其他异常情况，因此需要捕获它以防止设置了 `set -e` 的 shell 脚本死亡。

有关详细信息，请参阅 confmodule.3 和 Debian 政策中的 debconf 协议规范。

**Perl**
您可以使用 Debconf::Client::ConfModule Perl 模块，该模块提供许多函数。每个函数对应协议中的一个命令——您向它传递参数，它返回结果。有关详细信息，请参阅 Debconf::Client::ConfModule (3pm) 手册页和 Debian 政策中的 debconf 协议规范。

**其他**
您将必须通过标准输入和标准输出直接与前端通信。这并不难；有关详细信息，请阅读协议规范（规范位于 Debian 政策中）。

您可用于与前端通信的所有命令的列表和说明位于附录 A（命令）中。配置脚本中最常用的命令是 "input"、"go" 和 "get"。简而言之，"input priority question" 要求前端确保已询问用户一个问题，指定询问用户的重要性。问题名称通常对应于模板文件中的模板名称。"go" 告诉前端向用户显示所有累积的输入命令。而 "get question" 要求前端将问题的答案返回给您。

关于配置脚本的其他注意事项：与其他维护者脚本一样，配置脚本必须是幂等的。配置脚本接收 2 个参数。第一个是 "configure" 或 "reconfigure"。后者仅当软件包被 `dpkg-reconfig` 重新配置时发生。第二个参数是上次配置的软件包版本。[5]。

继续我们的示例，我们需要一个配置脚本来显示第一个问题，如果用户表示不喜欢 debian，则应显示相关信息。

```bash
#!/bin/sh -e

# 导入 debconf 库。
. /usr/share/debconf/confmodule

# 您喜欢 Debian 吗？
db_input medium foo/like_debian || true
db_go

# 检查他们的答案。
db_get foo/like_debian
if [ "$RET" = "false" ]; then
    # 可怜的迷途者...
    db_input high foo/why_debian_is_great || true
    db_go
fi
```

注意：配置脚本在软件包解包之前运行。它应仅使用基本（essential）软件包中的命令。此外，它不应编辑系统上的文件，或以任何其他方式影响系统。

## 修改现有的维护者脚本
有了模板文件和配置脚本后，就可以开始在软件包的其他维护者脚本（如 postinst）中使用配置脚本收集的数据了。与配置脚本类似，postinst 和其他维护者脚本可以使用 confmodule 或 Debconf::Client::ConfModule 库，也可以直接通过标准输出与前端通信。

维护者脚本输出到标准输出的任何内容都会作为命令传递给前端，因此需要删除所有无关的输出，如启动和停止守护进程等。

postinst 通常用于与前端通信的唯一命令是 "get" [6]。通常，配置脚本提示用户输入，然后 postinst 通过 get 命令从数据库中提取该输入。

在我们的示例中，在使用 debconf 之前，软件包的 postinst 如下：

```bash
#!/bin/sh -e
echo -n "Do you like debian? [yn] "
read like
case "$like" in
n*|N*)
    echo "Poor misguided one. Why are you installing this package, then?"
    /etc/init.d/subliminal_messages start "I like debian."
;;
esac
```

我们的配置脚本已经处理了大部分。使用 debconf 后，postinst 变为：

```bash
#!/bin/sh -e

# 导入 debconf 库。
. /usr/share/debconf/confmodule

db_get foo/like_debian
if [ "$RET" = "false" ]; then
    /etc/init.d/subliminal_messages start "I like debian."
fi
```

您需要对 postrm 脚本进行另一项修改。当软件包被清除（purge）时，它应清除数据库中使用的所有问题和模板。实现这一点很简单；使用 "purge" 命令（但请确保在 debconf 已被移除时不会失败）：

```bash
if [ "$1" = "purge" -a -e /usr/share/debconf/confmodule ]; then
    # 导入 debconf 库。
    . /usr/share/debconf/confmodule
    # 删除我对数据库的更改。
    db_purge
fi
```

注意：如果您使用 `dh_installdebconf` 命令，Debhelper 会为您完成此操作。

注意：即使您的 postinst 不处理 debconf，您当前也需要确保它加载了一个 debconf 库。这是因为 debconf 库执行了深层魔法以使配置脚本工作。将来会改变这一点。

## 收尾工作
现在您有了一个配置脚本和一个模板文件。将两者安装到 `debian/tmp/DEBIAN/` 中。确保配置脚本可执行。[7]

您的软件包现在使用 debconf 了！

## 测试
在构建软件包之前，您可能需要测试编写的配置脚本。这是可以做到的，而无需安装软件包——只需运行配置脚本。但有一个问题：配置脚本依赖于模板在运行前已加载。安装使用 debconf 的软件包时，这会自动处理。幸运的是，在大多数情况下，当您手动运行配置脚本时，也会自动处理。Debconf 使用两条简单规则来尝试找出与配置脚本关联的模板文件，如果找到，就加载它。

首先，如果存在一个文件，其名称是在正在运行的配置脚本名称后附加 ".templates"，则 debconf 假定那就是模板文件。

如果失败，debconf 会检查正在运行的配置脚本的文件名是否以 "config" 结尾。如果是，并且存在一个相同名称的文件，只是 "config" 替换为 "templates"，则 debconf 假定那就是模板文件。

注意：虽然这有点丑，但它意味着您可以将配置脚本命名为 `debian/config` 或 `debian/package.config`，并以类似方式命名模板文件，然后直接运行它们，一切都会正常工作。

## 故障排除
将某些内容转换到 debconf 时，有几件事可能会出错：

1. 您的 postinst 使用了 debconf 并启动了一个没有关闭所有继承的文件描述符的守护进程（所有此类守护进程都是错误的）。这会使 debconf 挂起，因为 debconf 前端等待守护进程关闭这些文件描述符后再继续。请注意，如果使用 confmodule，程序可能需要关闭文件描述符 0、1、2 和 3。

解决方法：
- 在运行守护进程之前，将文件描述符重定向到 /dev/null
- 或修复守护进程。
- 或在 postinst 末尾调用 "stop" 命令，让前端知道您已完成。

2. 发生了一些奇怪的事情，您不明白发生了什么。

尝试在环境中将 `DEBCONF_DEBUG` 设置为 'developer'。这会使 debconf 输出调试信息，包括您的脚本发送给它的命令及其响应。有关 `DEBCONF_DEBUG` 环境变量的更多信息，请参阅用户手册。

## 高级主题
现在我将继续讨论 debconf 的一些更复杂的方面。

### 块
这实际上相当容易做到。一些 debconf 前端能够同时在屏幕上显示多个问题。但是，问题之间不能相互依赖。[8] 只需将输入命令包装在 beginblock 和 endblock 命令之间。

### 允许用户返回
如果您正在询问一长串问题，用户能够跳回列表并更改答案会非常有用。Debconf 支持这一点，但您需要做相当多的工作才能使您的软件包支持它。

第一步是让您的配置脚本让 debconf 知道它能够处理用户按下后退按钮。您使用 capb 命令执行此操作，传递 "backup" 作为参数。

然后，在每个 go 命令之后，您必须测试用户是否按了后退按钮，如果是，则跳回上一个问题。如果用户按了后退按钮，go 命令将返回 30。

有多种方法可以编写程序的控制结构，使其能够在必要时跳回前面的问题。您可以编写充满 goto 的意大利面条代码。或者可以创建多个函数并使用递归。但也许最简洁、最简单的方法是构建一个状态机。让我们以前面教程中开发的示例配置脚本为例，使其支持返回。在该配置脚本中，只有一个地方用户可能按后退按钮：[9] 当向他们显示第二个问题时，返回第一个问题。以下是处理后退按钮的新版本脚本：

```bash
#!/bin/sh -e

# 导入 debconf 库。
. /usr/share/debconf/confmodule
db_version 2.0

# 此配置脚本能够返回
db_capb backup

STATE=1
while [ "$STATE" != 0 -a "$STATE" != 3 ]; do
    case "$STATE" in
    1)
        # 您喜欢 Debian 吗？
        db_input medium foo/like_debian || true
    ;;

    2)
        # 检查他们是否喜欢 Debian。
        db_get foo/like_debian
        if [ "$RET" = "false" ]; then
            # 可怜的迷途者...
            db_input high foo/why_debian_is_great || true
        fi
    ;;
    esac

    if db_go; then
        STATE=$(($STATE + 1))
    else
        STATE=$(($STATE - 1))
    fi
done
```

### 防止无限循环
使用 debconf 时要注意的一个陷阱是，如果配置脚本中有循环。假设您请求输入并进行验证，如果无效则循环：

```bash
ok=''
do while [ ! "$ok" ];
    db_input low foo/bar || true
    db_go || true

    db_get foo/bar
    if [ "$RET" ]; then
        ok=1
    fi
done
```

乍一看这没问题。但考虑一下，如果 foo/bar 的值为 "" 时进入此循环，并且用户的优先级设置得很高，或者他们使用的是非交互式前端，因此他们实际上没有被询问输入。foo/bar 的值不会被 db_input 改变，因此它无法通过测试并循环。并循环...

解决这个问题的方法是确保在进入循环之前，foo/bar 的值设置为能通过循环测试的值。因此，例如，如果 foo/bar 的默认值是 "1"，则可以在进入循环之前调用 "reset" 命令。

另一种解决方法是检查 "input" 命令的返回码。如果是 30，则表示用户没有被显示您问的问题，您应该跳出循环。

### 在相关软件包中选择
有时可以安装一组相关软件包，您想提示用户默认使用哪个软件包。此类集合的例子有窗口管理器或 ispell 字典文件。虽然可以简单地让集合中的每个软件包提示"此软件包应为默认吗？"，但如果安装了多个软件包，这会导致很多重复的问题。使用 debconf 可以显示集合中所有软件包的列表，并允许用户选择。方法如下。

让集合中的所有软件包使用一个共享模板。类似这样：

```
Template: shared/window-manager
Type: select
Choices: ${choices}
Description: Select the default window manager.
 Select the window manager that will be started by default when X
 starts.
```

每个软件包都应包含此模板的副本。然后在配置脚本中包含类似以下代码：

```bash
db_metaget shared/window-manager owners
OWNERS=$RET
db_metaget shared/window-manager choices
CHOICES=$RET

if [ "$OWNERS" != "$CHOICES" ]; then
    db_subst shared/window-manager choices $OWNERS
    db_fset shared/window-manager seen false
fi

db_input medium shared/window-manager || true
db_go || true
```

需要一些解释。在配置脚本运行时，debconf 已经读取了正在安装的所有软件包的模板。由于这组软件包共享一个问题，debconf 在所有者（owners）字段中记录了这一点。出于奇怪的巧合，所有者字段的格式与选择（choices）字段的格式相同（逗号和空格分隔的值列表）。

"metaget" 命令可用于获取所有者列表和选择列表。如果它们不同，则表示安装了新软件包。因此使用 "subst" 命令将选择列表更改为与所有者列表相同，并询问问题。

当软件包被移除时，您可能想查看该软件包是否是当前选中的选项，如果是，则提示用户选择不同的软件包替换它。

这可以通过在相关软件包的 prerm 脚本中添加以下内容来实现（将 `<package>` 替换为软件包名称）：

```bash
if [ -e /usr/share/debconf/confmodule ]; then
    . /usr/share/debconf/confmodule
    # 我不再声称拥有此问题。
    db_unregister shared/window-manager

    # 查看共享问题是否仍然存在。
    if db_get shared/window-manager; then
        db_metaget shared/window-manager owners
        db_subst shared/window-manager choices $RET
        db_metaget shared/window-manager value
        if [ "<package>" = "$RET" ] ; then
            db_fset shared/window-manager seen false
            db_input critical shared/window-manager || true
            db_go || true
        fi

        # 现在执行 postinst 脚本为更新窗口管理器符号链接所做的任何操作。
    fi
fi
```

### 使用 debconf 的独立程序
实际上可以在独立程序中使用 debconf，而不仅仅是在 Debian 软件包脚本中使用。这样做是否是个好主意，则是另一个问题。

注意：请记住，debconf 并非设计为注册表。

为向用户提问而编写使用 debconf 的独立程序可能偶尔有意义。使用 debconf 的独立程序的工作方式类似于 debian 配置脚本：为您选择的语言加载 debconf 库，然后照常进行。

如果您正在做这种事情，您可能会发现在 `/usr/share/debconf/$0.templates` 中放置模板文件很有用，其中 $0 是正在运行的程序名称。程序运行时，Debconf 将加载此类模板。

## A. 命令
本附录已被移除，因为 debconf 规范现在位于 Debian 政策中，并且不再从此文档的同一树构建。请参阅政策以获取完整的命令列表。

## B. 问题层次结构
Debconf 数据库中的模板和问题按层次命名空间排列。它可以任意深，并由 '/' 字符分隔，类似于 Unix 目录层次结构。以下是我们目前用于划分命名空间的约定。

- **package/\*** - 这是单个软件包的属性，可以由该软件包任意细分。
- **shared/foo/\*** - 这保存多个软件包共享的条目。"foo" 被与它们都相关的名称替换。例如，新闻抓取器和阅读器都可以使用 shared/news/server 来存储它们使用的新闻服务器。

当前存在以下共享模板和问题：

- **shared/news/server** - 新闻服务器的主机名。

## 注释
[1] 由于政策不赞成未经 debian-devel 批准的预依赖，您也可以让软件包检测是否安装了 debconf，并使用不涉及 debconf 的合理回退方案。

[2] 实际上，会首先检查 Description-es_ES（或 es_MX 等）。

[3] 除非默认值依赖于区域设置。

[4] 如果您使用 `dh_installdebconf` 命令，Debhelper 会为您完成此操作。

[5] 这与传递给 postinst 脚本的参数非常相似。

[6] 虽然实际上它们可以使用任何命令，包括 "input"，但强烈建议不要这样做。

[7] 如果您使用 debhelper，这一切都会自动完成。

[8] 例如，如果问题 b 仅在问题 a 为真时才应被询问，您显然不能同时显示它们。

[9] 实际上有两个。另一个可能用到后退按钮的时机是在询问第一个问题时，它可能会从配置此软件包跳回到配置前一个软件包。然而，此功能如何工作的细节尚未敲定。
