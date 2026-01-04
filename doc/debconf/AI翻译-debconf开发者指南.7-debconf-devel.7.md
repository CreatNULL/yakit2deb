https://manpages.debian.org/jessie/debconf-doc/debconf-devel.7.en.html
---

**名称**  
debconf - 开发者指南  

**描述**  
这是一份为使用 debconf 的软件包开发提供的指南。本手册假设您熟悉 debconf 作为用户，并且熟悉 Debian 软件包构建的基础知识。本手册首先解释了添加到使用 debconf 的 Debian 软件包中的两个新文件。然后，它说明了 debconf 协议的工作原理，并为您指出一些可以让您的程序使用该协议的库。它讨论了 debconf 通常使用的其他维护者脚本：postinst 和 postrm 脚本。接着，本手册将探讨更高级的主题，例如共享 debconf 模板、调试，以及使用 debconf 编程的一些常见技术和陷阱。最后，本手册讨论了 debconf 当前的缺点。  

**配置脚本**  
Debconf 在 Debian 软件包（包括 postinst、preinst、postrm 和 prerm）中可以包含的维护者脚本集中，增加了一个额外的维护者脚本——配置脚本。配置脚本负责询问配置软件包所需的任何问题。  
**注意**：dpkg 将运行软件包的 postinst 脚本称为“配置”软件包，这有点令人困惑，因为使用 debconf 的软件包通常在其 postinst 脚本运行之前就已经通过其配置脚本进行了完整的预配置。嗯，就这样吧。  
与 postinst 类似，运行配置脚本时会传递两个参数。第一个参数指定正在执行的操作，第二个参数是当前已安装软件包的版本。因此，与 postinst 一样，您可以在 $2 上使用 `dpkg --compare-versions` 来实现在从特定版本升级时才触发某些行为等功能。  
配置脚本可以通过以下三种方式之一运行：

1. 如果软件包是通过 `dpkg-preconfigure` 进行预配置的，其配置脚本将以参数 "configure" 和 installed-version 运行。  
2. 当软件包的 postinst 脚本运行时，debconf 也会尝试运行配置脚本，并传递与预配置时相同的参数。这是必要的，因为软件包可能没有被预配置，配置脚本仍然需要运行的机会。详情请参见 HACKS 部分。  
3. 如果软件包通过 `dpkg-reconfigure` 重新配置，其配置脚本将以参数 "reconfigure" 和 installed-version 运行。

请注意，典型的软件包安装或升级使用 apt 会执行步骤 1 和 2，因此配置脚本通常会被运行两次。第二次运行时应该不做任何操作（重复提问是烦人的），并且它绝对应该是幂等的。幸运的是，debconf 默认避免重复提问，因此这通常很容易实现。  
**注意**：配置脚本在软件包解包之前运行。它应该只使用 Essential 软件包中的命令。当配置脚本运行时，您的软件包唯一能保证满足的依赖是（可能指定版本的）debconf 本身的依赖。  
配置脚本根本不需要修改文件系统。它只检查系统状态并提问，debconf 存储答案以供稍后的 postinst 脚本执行操作。  
相反，postinst 脚本几乎不应使用 debconf 提问，而应基于配置脚本所提问题的答案执行操作。  

**模板文件**  
使用 debconf 的软件包可能需要一些问题。这些问题以模板形式存储在模板文件中。与配置脚本一样，模板文件放置在 deb 包的 control.tar.gz 部分。其格式类似于 Debian 控制文件：由空行分隔的一组节，每节采用类似 RFC822 的格式：

```
Template: foo/bar
Type: string
Default: foo
Description: 这是一个示例字符串问题。
这是其扩展描述。
.
```

请注意：
- 与在 Debian 软件包描述中类似，单独一行上的点 . 表示新段落开始。
- 大部分文本会自动换行，但双缩进的文本会保持原样，因此您可以将其用于项目列表，就像这个列表。请注意，因为它不自动换行，如果太宽，看起来会很差。最好用于短项目（所以这是一个糟糕的示例）。

```
Template: foo/baz
Type: boolean
Description: 很清楚，对吧？
```

这是另一个布尔类型的问题。  
有关模板文件的实际示例，请参见 `/var/lib/dpkg/info/debconf.templates` 以及该目录中的其他 .templates 文件。  
我们来逐一查看每个字段：

**Template**  
模板名称，在 'Template' 字段中，通常以软件包名称作为前缀。之后，命名空间是开放的；您可以使用如上所示的简单平面布局，也可以为相关问题设置“子目录”。

**Type**  
模板的类型决定了向用户显示哪种类型的小部件。当前支持的类型有：

*   **string**：生成一个自由格式的输入字段，用户可以输入任何字符串。
*   **password**：提示用户输入密码。请谨慎使用此类型；请注意，用户输入的密码将被写入 debconf 的数据库。您可能应在可能的情况下尽快从数据库中清除该值。
*   **boolean**：真/假选择。
*   **select**：在多个值中选择一个。选项必须在名为 'Choices' 的字段中指定。用逗号和空格分隔可能的值，像这样：
    ```
    Choices: yes, no, maybe
    ```
*   **multiselect**：类似于 select 数据类型，但用户可以从选项列表中选择任意数量的项目（或一个也不选）。
*   **note**：这本身不是一个问题，这种数据类型指示可以显示给用户的注释。它应仅用于用户真正需要看到的重要说明，因为 debconf 会极力确保用户看到它；暂停安装等待用户按键。最好仅用于警告非常严重的问题，而 error 数据类型通常更合适。
*   **error**：此数据类型用于错误消息，例如输入验证错误。即使优先级太高或用户已看过，debconf 也会显示此类型的问题。
*   **title**：此数据类型用于标题，通过 SETTITLE 命令设置。
*   **text**：此数据类型可用于文本片段，例如标签，在某些前端显示中出于美观原因使用。其他前端可能完全不使用它。目前使用此数据类型没有意义，因为没有前端很好地支持它。未来甚至可能将其移除。

**Default**  
'Default' 字段告诉 debconf 默认值应该是什么。对于 multiselect，它可以是选项列表，用逗号和空格分隔，类似于 'Choices' 字段。对于 select，它应该是选项之一。对于 boolean，它是 "true" 或 "false"，对于 string 可以是任何值，对于 password 则被忽略。不要误认为默认字段包含问题的“值”，或者它可以用于更改问题的值。它没有，也不能，它只是为问题首次显示时提供一个默认值。要提供动态更改的默认值，您必须使用 SET 命令来更改问题的值。

**Description**  
'Description' 字段，类似于 Debian 软件包的描述，有两部分：简短描述和扩展描述。请注意，某些 debconf 前端不显示长描述，或者可能仅在用户请求帮助时显示。因此，简短描述应该能够独立存在。  
如果您想不出长描述，那么，先再想想。发帖到 debian-devel。请求帮助。上个写作课！扩展描述很重要。如果在所有这些之后您仍然想不出任何东西，就留空。重复简短描述没有意义。  
扩展描述中的文本将自动换行，除非它前面有额外的空白（超出所需的一个空格）。您可以通过在它们之间放置一个单独一行的 "." 来将其分成单独的段落。

**问题**  
问题是从模板实例化的。通过要求 debconf 显示一个问题，您的配置脚本可以与用户交互。当 debconf 加载模板文件时（每当运行配置脚本或 postinst 脚本时都会发生），它会自动从每个模板实例化一个问题。实际上，可以从同一模板实例化几个独立的问题（使用 REGISTER 命令），但这很少需要。模板是来自模板文件的静态数据，而问题用于存储动态数据，例如问题的当前值，用户是否已看到问题等等。记住模板和问题之间的区别，但不要太担心。

**共享模板**  
实际上，可以拥有一组软件包共享的模板和问题。所有软件包必须在它们的模板文件中提供模板的相同副本。如果一组软件包需要询问相同的问题，并且您只想向用户询问一次，这会很有用。共享模板通常放在 debconf 模板命名空间的 shared/ 伪目录中。

**DEBCONF 协议**  
配置脚本使用 debconf 协议与 debconf 通信。这是一个简单的面向行的协议，类似于常见的互联网协议，如 SMTP。配置脚本通过将命令写入标准输出向 debconf 发送命令。然后，它可以从标准输入读取 debconf 的回复。Debconf 的回复可以分为两部分：数字结果代码（回复的第一个单词）和可选的扩展结果代码（回复的其余部分）。数字代码使用 0 表示成功，使用其他数字表示各种类型的失败。有关详细信息，请参阅 Debian 政策中 debconf 规范文档中的表格。  
扩展返回码通常是自由形式且未指定的，因此您通常应忽略它，并且当然不应尝试在程序中解析它以找出 debconf 正在做什么。例外情况是像 GET 这样的命令，它会导致在扩展返回码中返回值。  
通常，您会希望使用特定于语言的库来处理与 debconf 建立连接和通信的细节。目前，这里有协议中的命令。这不是权威定义，请参阅 Debian 政策的 debconf 规范文档以获取权威定义。

*   **VERSION number**：您通常不需要使用此命令。它与 debconf 交换正在使用的协议版本号。当前协议版本是 2.0，2.x 系列版本将向后兼容。您可以指定您说的协议版本号，debconf 将在扩展结果代码中返回它所说的协议版本。如果您指定的版本太低，debconf 将以数字代码 30 回复。
*   **CAPB capabilities**：您通常不需要使用此命令。它与 debconf 交换支持的功能列表（以空格分隔）。您和 debconf 都支持的功能将被使用，debconf 将回复它支持的所有功能。如果在您的功能中发现 'escape'，debconf 将期望您发送给它的命令对反斜杠和换行符进行转义（分别为 \\ 和 \n），并将相应地转义其回复中的反斜杠和换行符。这可以用于例如将多行字符串替换到模板中，或使用 METAGET 可靠地获取多行扩展描述。在此模式下，您必须自己转义输入文本（如果需要，可以使用 debconf-escape(1) 帮助），但 confmodule 库将为您对回复进行反转义。
*   **SETTITLE question**：这设置 debconf 向用户显示的标题，使用指定问题的模板的简短描述。模板应为 title 类型。您很少需要此命令，因为 debconf 可以基于您的软件包名称自动生成标题。从模板设置标题意味着它们与其余 debconf 问题存储在相同位置，并允许进行翻译。
*   **TITLE string**：这将 debconf 向用户显示的标题设置为指定的字符串。通常更倾向于使用 SETTITLE 命令，因为它允许标题翻译。
*   **INPUT priority question**：要求 debconf 准备向用户显示一个问题。该问题在发出 GO 命令之前不会实际显示；这允许依次给出多个 INPUT 命令，以构建一组问题，这些问题可能在一个屏幕上全部询问。priority 字段告诉 debconf 此问题向用户显示的重要性。优先级值为：
    *   **low**：非常琐碎的项目，其默认值在绝大多数情况下都有效；只有控制狂才能看到这些。
    *   **medium**：具有合理默认值的普通项目。
    *   **high**：没有合理默认值的项目。
    *   **critical**：没有用户干预可能会破坏系统的项目。  
Debconf 基于问题的优先级、用户是否已看过它以及正在使用的前端来决定是否实际显示该问题。如果问题不显示，debconf 以代码 30 回复。
*   **GO**：告诉 debconf 向用户显示累积的一组问题（来自 INPUT 命令）。如果支持 backup 功能并且用户表示要后退一步，debconf 以代码 30 回复。
*   **CLEAR**：清除累积的一组问题（来自 INPUT 命令）而不显示它们。
*   **BEGINBLOCK**、**ENDBLOCK**：一些 debconf 前端可以一次向用户显示多个问题。也许将来前端甚至能够将这些问题在屏幕上分组。BEGINBLOCK 和 ENDBLOCK 可以放在一组 INPUT 命令周围以指示问题块（块甚至可以嵌套）。由于目前没有 debconf 前端如此复杂，这些命令目前被忽略。
*   **STOP**：此命令告诉 debconf 您已完成与它的对话。通常 debconf 可以检测到您的程序终止，此命令不是必需的。
*   **GET question**：在使用 INPUT 和 GO 显示问题后，您可以使用此命令获取用户输入的值。该值在扩展结果代码中返回。
*   **SET question value**：这设置问题的值，并可用于用您的程序动态计算的内容覆盖默认值。
*   **RESET question**：这将问题重置为其默认值（如其模板的 'Default' 字段中所指定）。
*   **SUBST question key value**：问题可以在其 'Description' 和 'Choices' 字段中嵌入替换（在 'Choices' 字段中使用替换有点 hack；最终将开发更好的机制）。这些替换看起来像 "${key}"。当问题显示时，替换被其值替换。此命令可用于设置替换的值。如果您需要向用户显示无法在模板文件中硬编码的某些消息，这很有用。  
**不要**尝试使用 SUBST 更改问题的默认值；它不起作用，因为有 SET 命令明确用于此目的。
*   **FGET question flag**：问题可以有关联的标志。标志的值可以是 "true" 或 "false"。此命令返回标志的值。
*   **FSET question flag value**：这设置问题的标志的值。该值必须是 "true" 或 "false"。一个常见的标志是 "seen" 标志。通常仅当用户已看过问题时才设置它。Debconf 通常仅当 seen 标志设置为 "false"（或正在重新配置软件包）时才向用户显示问题。有时您希望用户再次看到问题——在这些情况下，您可以将 seen 标志设置为 false 以强制 debconf 重新显示它。
*   **METAGET question field**：这返回问题的关联模板的任何字段的值（例如 Description）。
*   **REGISTER template question**：这创建一个绑定到模板的新问题。默认情况下，每个模板都有一个具有相同名称的关联问题。但是，实际上任何数量的问题都可以与一个模板关联，这允许您创建更多此类问题。
*   **UNREGISTER question**：这从数据库中删除一个问题。
*   **PURGE**：在您的 postrm 中调用此命令，当您的软件包被清除时。它从 debconf 的数据库中删除您的软件包的所有问题。
*   **X_LOADTEMPLATEFILE /path/to/templates [owner]**：此扩展将指定的模板文件加载到 debconf 的数据库中。所有者默认为正在用 debconf 配置的软件包。

以下是 debconf 协议的一个简单示例：

```
INPUT medium debconf/frontend
30 question skipped
FSET debconf/frontend seen false
0 false
INPUT high debconf/frontend
0 question will be asked
GO
[ 这里 debconf 向用户显示一个问题。 ]
0 ok
GET no/such/question
10 no/such/question doesn't exist
GET debconf/frontend
0 Dialog
```

**库**  
设置与 debconf 通信的环境，并手动使用 debconf 协议有点太麻烦了，因此存在一些轻量级库来减轻这种繁琐的工作。  
对于 shell 编程，有 `/usr/share/debconf/confmodule` 库，您可以在 shell 脚本的顶部导入它，并以相当自然的方式与 debconf 对话，使用小写版本的 debconf 协议命令，并以前缀 "db_" 开头（即 "db_input" 和 "db_go"）。有关详细信息，请参阅 confmodule(3)。  
Perl 程序员可以使用 Debconf::Client::ConfModule(3pm) Perl 模块，Python 程序员可以使用 debconf Python 模块。本手册的其余部分将在示例 shell 脚本中使用 `/usr/share/debconf/confmodule` 库。  
以下是使用该库的示例配置脚本，它只问一个问题：

```
#!/bin/sh
set -e
. /usr/share/debconf/confmodule
db_set mypackage/reboot-now false
db_input high mypackage/reboot-now || true
db_go || true
```

请注意 "|| true" 的用法，以防止在 debconf 决定无法显示问题或用户试图后退时脚本死亡。在这些情况下，debconf 返回非零退出代码，并且由于此 shell 脚本设置了 set -e，未捕获的退出代码将使其中止。  
以下是相应的 postinst 脚本，它使用用户对问题的答案来查看系统是否应重新启动（一个相当荒谬的例子..）：

```
#!/bin/sh
set -e
. /usr/share/debconf/confmodule
db_get mypackage/reboot-now
if [ "$RET" = true ]; then
shutdown -r now
fi
```

注意使用 $RET 变量获取 GET 命令的扩展返回码，其中包含用户对该问题的答案。

**POSTINST 脚本**  
上一节有一个 postinst 脚本的示例，该脚本使用 debconf 获取问题的值并据此执行操作。编写使用 debconf 的 postinst 脚本时，请记住以下几点：

*   避免在 postinst 中提问。相反，配置脚本应使用 debconf 提问，以便预配置能够工作。
*   始终在 postinst 的顶部导入 `/usr/share/debconf/confmodule`，即使您不会在其中运行任何 db_* 命令。这是确保配置脚本有机会运行所必需的（详情请参见 HACKS）。
*   避免在 postinst 中向标准输出输出任何内容，因为这可能会混淆 debconf，而且 postinst 无论如何不应该冗长。如果必须，向标准错误输出是可以的。
*   如果您的 postinst 启动一个守护进程，请确保在末尾告诉 debconf STOP，否则 debconf 可能会对您的 postinst 何时完成感到有点困惑。
*   让您的 postinst 脚本接受第一个参数为 "reconfigure"。它可以像处理 "configure" 一样处理它。这将在 debconf 的后续版本中用于让 postinst 知道它们何时被重新配置。

**其他脚本**  
除了配置脚本和 postinst，您可以在任何其他维护者脚本中使用 debconf。最常见的是，您将在 postrm 中使用 debconf，在软件包被清除时调用 PURGE 命令，以清理其在 debconf 数据库中的条目。（顺便说一下，这是由 dh_installdebconf(1) 自动为您设置的。）  
更复杂的使用 debconf 的情况是，如果您想在软件包被清除时的 postrm 中使用它，询问关于删除某些内容的问题。或者，您可能发现由于某些原因需要在 preinst 或 prerm 中使用它。所有这些用法都有效，尽管它们可能涉及在同一程序中提问并根据答案执行操作，而不是像配置和 postinst 脚本那样将两项活动分开。  
**注意**：如果您的软件包唯一使用 debconf 的地方是在 postrm 中，您应该让您的软件包的 postinst 导入 `/usr/share/debconf/confmodule`，以便 debconf 有机会将其模板文件加载到其数据库中。这样，在清除软件包时，模板将可用。  
您还可以在其他独立的程序中使用 debconf。这里要注意的问题是，debconf 不打算用作注册表，并且绝不能用作注册表。毕竟这是 Unix，程序由 `/etc` 中的文件配置，而不是由一些模糊的 debconf 数据库（这只是一个缓存，可能会被清除）配置。因此，在独立程序中使用 debconf 之前，请仔细考虑。有时这是有意义的，例如在 apt-setup 程序中，它使用 debconf 以与 Debian 安装过程其余部分一致的方式提示用户，并根据他们的答案立即设置 apt 的 sources.list。

**本地化**  
Debconf 支持模板文件的本地化。这是通过添加更多包含翻译文本的字段来实现的。任何字段都可以被翻译。例如，您可能想将描述翻译成西班牙语。只需创建一个名为 'Description-es' 的字段，其中包含翻译即可。如果翻译字段不可用，debconf 将回退到正常的英语字段。  
除了 'Description' 字段，您还应该翻译 select 或 multiselect 模板的 'Choices' 字段。请确保以与主 'Choices' 字段中相同的顺序列出翻译后的选项。  
您不需要翻译 select 或 multiselect 问题的 'Default' 字段，问题的值将自动以英语返回。  
如果您将翻译保存在单独的文件中（每个翻译一个文件），管理翻译会更容易。过去，使用 debconf-getlang(1) 和 debconf-mergetemplate(1) 程序来管理 debian/template.ll 文件。这已被 po-debconf(7) 软件包取代，它允许您像处理任何其他翻译一样，在 .po 文件中处理 debconf 翻译。您的翻译者会感谢您使用这个新的改进机制。有关 po-debconf 的详细信息，请参阅其手册页。  
如果您使用 debhelper，转换为 po-debconf 就像运行一次 debconf-gettextize(1) 命令一样简单，并在 Build-Dependency 中添加对 po-debconf 和 debhelper (>= 4.1.13) 的依赖。

**将所有内容整合在一起**  
所以您有了一个配置脚本、一个模板文件、一个使用 debconf 的 postinst 脚本等等。将这些部分组合成一个 Debian 软件包并不难。您可以手动完成，也可以使用 dh_installdebconf(1)，它将合并您翻译后的模板，将文件复制到正确的位置，甚至可以生成应该在 postrm 脚本中调用的 PURGE 命令。  
确保您的软件包依赖于 debconf (>= 0.5)，因为早期版本与本文档中描述的所有内容不完全兼容。  
然后您就完成了。嗯，除了测试、调试以及实际上将 debconf 用于比询问几个基本问题更有趣的事情。关于这些，请继续阅读..

**调试**  
您有一个应该使用 debconf 的软件包，但它不能完全正常工作。也许 debconf 就是没有询问您设置的那个问题。或者也许发生了更奇怪的事情；它陷入某种无限循环，或者更糟。幸运的是，debconf 提供了大量的调试功能。

**DEBCONF_DEBUG**  
首先应该使用的是 DEBCONF_DEBUG 环境变量。如果设置并导出 DEBCONF_DEBUG=developer，debconf 将在程序运行时将 debconf 协议的转储输出到标准错误。它将看起来像这样——拼写错误变得清晰：

```
debconf (developer): <-- input high debconf/frontand
debconf (developer): --> 10 "debconf/frontand" doesn't exist
debconf (developer): <-- go
debconf (developer): --> 0 ok
```

调试时使用 debconf 的 readline 前端非常有用（在作者看来），因为问题不会妨碍，而且所有调试输出都易于保存和记录。

**DEBCONF_C_VALUES**  
如果此环境变量设置为 'true'，前端将显示 select 和 multiselect 模板的 Choices-C 字段（如果存在）中的值，而不是描述性值。

**debconf-communicate**  
另一个有用的工具是 debconf-communicate(1) 程序。启动它，您可以交互地与 debconf 对话原始 debconf 协议。这是在飞行中尝试一些东西的好方法。

**debconf-show**  
如果用户报告问题，可以使用 debconf-show(1) 来转储您的软件包拥有的所有问题，显示它们的值以及用户是否已看过它们。

**.debconfrc**  
为了避免通常繁琐的构建/安装/调试周期，使用 debconf-loadtemplate(1) 加载模板并使用 debconf(1) 命令手动运行配置脚本会很有用。但是，您仍然必须以 root 身份执行，对吗？这不太好。理想情况下，您希望能够看到软件包的全新安装是什么样子，使用干净的 debconf 数据库。事实证明，如果您为普通用户设置一个 ~/.debconfrc 文件，指向用户个人的 config.dat 和 template.dat，您可以根据需要加载模板和运行配置脚本，而无需任何 root 权限。如果您想重新开始使用干净的数据库，只需删除 *.dat 文件。有关设置此功能的详细信息，请参阅 debconf.conf(5)，并注意 /etc/debconf.conf 可以作为个人 ~/.debconfrc 文件的良好模板。

**使用 DEBCONF 进行高级编程**  

**配置文件处理**  
你们中的许多人似乎想使用 debconf 来帮助管理作为软件包一部分的配置文件。也许配置文件中没有合适的默认值，因此您想使用 debconf 提示用户，并根据他们的答案写出配置文件。这似乎很容易做到，但随后您会考虑升级，以及当有人修改您生成的配置文件时该怎么做，以及 dpkg-reconfigure 等等……有很多方法可以做到这一点，但大多数都是错误的，并且常常会让您收到恼人的错误报告。  
这里有一种正确的方法。它假设您的配置文件实际上只是一系列设置的 shell 变量，中间有注释，因此您可以导入文件以“加载”它。如果您有更复杂的格式，读取（和写入）它就会变得有点棘手。  
您的配置脚本将大致如下所示：

```
#!/bin/sh
CONFIGFILE=/etc/foo.conf
set -e
. /usr/share/debconf/confmodule

# 加载配置文件（如果存在）。
if [ -e $CONFIGFILE ]; then
. $CONFIGFILE || true
# 将配置文件中的值存储到
# debconf 数据库。
db_set mypackage/foo "$FOO"
db_set mypackage/bar "$BAR"
fi

# 提问。
db_input medium mypackage/foo || true
db_input medium mypackage/bar || true
db_go || true
```

而 postinst 将大致如下所示：

```
#!/bin/sh
CONFIGFILE=/etc/foo.conf
set -e
. /usr/share/debconf/confmodule

# 生成配置文件（如果不存在）。
# 另一种方法是从其他地方复制模板文件。
if [ ! -e $CONFIGFILE ]; then
echo "# Config file for my package" > $CONFIGFILE
echo "FOO=" >> $CONFIGFILE
echo "BAR=" >> $CONFIGFILE
fi

# 用 debconf 数据库中的值替换。
# 这里可以进行明显的优化。
# sed 之前的 cp 确保我们不会弄乱
# 配置文件的所有权和权限。
db_get mypackage/foo
FOO="$RET"
db_get mypackage/bar
BAR="$RET"

cp -a -f $CONFIGFILE $CONFIGFILE.tmp
# 如果管理员删除或注释了一些变量，但随后通过 debconf 设置了它们，则（重新）添加到配置文件中。
test -z "$FOO" || grep -Eq '^ *FOO=' $CONFIGFILE || \
echo "FOO=" >> $CONFIGFILE
test -z "$BAR" || grep -Eq '^ *BAR=' $CONFIGFILE || \
echo "BAR=" >> $CONFIGFILE

sed -e "s/^ *FOO=.*/FOO=\"$FOO\"/" \
-e "s/^ *BAR=.*/BAR=\"$BAR\"/" \
< $CONFIGFILE > $CONFIGFILE.tmp
mv -f $CONFIGFILE.tmp $CONFIGFILE
```

考虑这两个脚本如何处理所有情况。在全新安装时，配置脚本会提出问题，postinst 会生成新的配置文件。在升级和重新配置时，会读取配置文件，并将其中的值用于更改 debconf 数据库中的值，因此管理员的手动更改不会丢失。再次询问问题（可能会显示，也可能不会）。然后 postinst 将这些值替换回配置文件，保持其余部分不变。

**允许用户后退**  
使用像 debconf 这样的系统时，几乎没有比问了一个问题，回答了它，然后转到另一个有新问题的屏幕，并意识到嘿，您在上一个问题中犯了一个错误，想要回到它，却发现无法返回更令人沮丧的了。由于 debconf 由您的配置脚本驱动，它本身无法跳回上一个问题，但在您的帮助下，它可以完成这个任务。  
第一步是让您的配置脚本让 debconf 知道它能够处理用户按下后退按钮。您使用 CAPB 命令来做到这一点，传递 backup 作为参数。然后，在每个 GO 命令之后，您必须测试用户是否要求后退（debconf 返回代码 30），如果是，则跳回到前一个问题。  
有几种方法可以编写程序的控制结构，以便在必要时可以跳回之前的问题。您可以编写充满 goto 的意大利面条代码。或者您可以创建几个函数并使用递归。但也许最干净、最简单的方法是构建一个状态机。  
以下是您可以填写和扩展的状态机骨架：

```
#!/bin/sh
set -e
. /usr/share/debconf/confmodule

db_capb backup
STATE=1
while true; do
case "$STATE" in
1)
# 两个不相关的问题。
db_input medium my/question || true
db_input medium my/other_question || true
;;
2)
# 仅当第一个问题得到肯定回答时才问此问题。
db_get my/question
if [ "$RET" = "true" ]; then
db_input medium my/dep_question || true
fi
;;
*)
# 默认情况处理 $STATE 大于最后一个实现状态的情况，并退出循环。
# 这要求状态从 1 开始连续编号，没有间隔，因为如果编号中断，也会进入默认情况
break # 退出外部的 "while" 循环
;;
esac
if db_go; then
STATE=$(($STATE + 1))
else
STATE=$(($STATE - 1))
fi
done

if [ $STATE -eq 0 ]; then
# 用户要求从第一个问题后退。
# 这种情况是有问题的。常规的 dpkg 和 apt 软件包安装无法在不同软件包之间后退问题，因此这将导致退出，留下软件包未配置 - 这可能是处理这种情况的最佳方式。
exit 10
fi
```

请注意，如果您的配置脚本所做的只是询问几个不相关的问题，则不需要状态机。只需全部提问，然后 GO；debconf 会尽力将它们全部显示在一个屏幕上，用户不需要后退。

**防止无限循环**  
如果在配置脚本中有循环，使用 debconf 时有一个陷阱。假设您请求输入并验证它，如果无效则循环：

```
ok=''
while [ ! "$ok" ]; do
db_input low foo/bar || true
db_go || true
db_get foo/bar
if [ "$RET" ]; then
ok=1
fi
done
```

乍一看这没问题。但考虑一下，如果 foo/bar 的值在此循环进入时为 ""，并且用户将其优先级设置得很高，或者正在使用非交互式前端，因此他们实际上没有被要求输入。foo/bar 的值不会被 db_input 更改，因此它无法通过测试并循环。然后循环……  
对此的一种修复方法是，确保在进入循环之前，将 foo/bar 的值设置为能够通过循环测试的内容。例如，如果 foo/bar 的默认值是 "1"，那么您可以在进入循环之前 RESET foo/bar。另一种修复方法是检查 INPUT 命令的返回代码。如果是 30，则不会向用户显示您询问他们的问题，您应该退出循环。

**在相关软件包中选择**  
有时一组相关的软件包可以安装，您想提示用户该集合中哪一个应作为默认使用。此类集合的示例是窗口管理器或 ispell 词典文件。虽然集合中的每个软件包都可以简单地提示“此软件包应为默认吗？”，但如果安装了多个软件包，这将导致许多重复的问题。使用 debconf 可以显示集合中所有软件包的列表，并允许用户在它们之间进行选择。方法如下。  
让集合中的所有软件包使用共享模板。类似这样：

```
Template: shared/window-manager
Type: select
Choices: ${choices}
Description: 选择默认窗口管理器。
选择当 X 启动时默认启动的窗口管理器。
```

每个软件包都应包含模板的副本。然后在其配置脚本中应包含类似以下的代码：

```
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

需要一点解释。在您的配置脚本运行时，debconf 已经读取了正在安装的所有软件包的模板。由于软件包集合共享一个问题，debconf 在 owners 字段中记录了这一事实。奇怪的是，owners 字段的格式与 choices 字段的格式相同（逗号和空格分隔的值列表）。METAGET 命令可用于获取 owners 列表和 choices 列表。如果它们不同，则安装了新软件包。因此使用 SUBST 命令将 choices 列表更改为与 owners 列表相同，并提问。  
当软件包被移除时，您可能希望检查该软件包是否是当前选定的选项，如果是，则提示用户选择其他软件包来替换它。这可以通过在所有相关软件包的 prerm 脚本中添加类似以下内容来实现（用软件包名称替换 `<package>`）：

```
if [ -e /usr/share/debconf/confmodule ]; then
. /usr/share/debconf/confmodule
# 我不再声明此问题。
db_unregister shared/window-manager
# 查看共享问题是否仍然存在。
if db_get shared/window-manager; then
db_metaget shared/window-manager owners
db_subst shared/window-manager choices $RET
db_metaget shared/window-manager value
if [ "<package>" = "$RET" ] ; then
db_fset shared/window-manager seen false
db_input high shared/window-manager || true
db_go || true
fi
# 现在执行 postinst 脚本所做的任何操作
# 来更新窗口管理器符号链接。
fi
fi
```

**HACKS**  
目前，debconf 尚未完全集成到 dpkg 中（但我希望在将来改变这一点），因此当前需要一些混乱的 hack。其中最糟糕的涉及让配置脚本运行。  
现在的工作方式是，配置脚本将在软件包预配置时运行。然后，当 postinst 脚本运行时，它会再次启动 debconf。Debconf 注意到它正被 postinst 脚本使用，因此它会去运行配置脚本。只有您的 postinst 加载了其中一个 debconf 库时，这才有效，因此 postinst 必须始终注意这样做。我们希望通过在 dpkg 中添加对 debconf 的显式支持来解决这个问题。debconf(1) 程序是朝着这个方向迈出的一步。  
一个相关的 hack 是在配置脚本、postinst 或其他使用它的程序启动时运行 debconf。毕竟，它们期望能够立即与 debconf 对话。目前实现这一点的方式是，当这样的脚本加载 debconf 库（如 `/usr/share/debconf/confmodule`）并且 debconf 尚未运行时，它将被启动，并且脚本的新副本会被重新执行。唯一明显的结果是，您需要将加载 debconf 库的行放在脚本的顶部，否则会发生奇怪的事情。我们希望通过更改 debconf 的调用方式来解决这个问题，并将其更改为更像瞬态守护进程的东西。  
debconf 如何确定加载哪些模板文件以及何时加载它们，这是相当 hackish 的。当配置、preinst 和 postinst 脚本调用 debconf 时，它会自动找出模板文件的位置并加载它。使用 debconf 的独立程序将导致 debconf 在 `/usr/share/debconf/templates/progname.templates` 中查找模板文件。如果 postrm 想在清除时使用 debconf，除非 debconf 有机会在其 postinst 中加载它们，否则模板将不可用。这很混乱，但不可避免。将来，其中一些程序可能能够手动使用 debconf-loadtemplate。  
`/usr/share/debconf/confmodule` 历史上会操作文件描述符并设置与 debconf 对话的 fd #3，当 postinst 运行守护进程时，这会导致各种麻烦，因为守护进程最终会与 debconf 对话，并且 debconf 无法确定脚本何时终止。STOP 命令可以解决这个问题。将来，我们考虑让 debconf 通信通过套接字或除 stdio 之外的某种机制进行。  
Debconf 在运行 postinst 脚本之前设置 DEBCONF_RECONFIGURE=1，因此需要避免在重新配置时执行某些昂贵操作的 postinst 脚本可以查看该变量。这是一个 hack，因为正确的方法应该是传递 $1 = "reconfigure"，但这样做而不破坏所有使用 debconf 的 postinst 是困难的。摆脱此 hack 的迁移计划是鼓励人们编写接受 "reconfigure" 的 postinst，一旦它们都这样做，就开始传递该参数。

**另请参阅**  
debconf(7) 是 debconf 用户指南。  
Debian 政策中的 debconf 规范是 debconf 协议的权威定义。 `/usr/share/doc/debian-policy/debconf_specification.txt.gz`  
debconf.conf(5) 包含许多有用信息，包括一些关于后端数据库的信息。

**作者**  
Joey Hess <joeyh@debian.org>
