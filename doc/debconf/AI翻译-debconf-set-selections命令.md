https://manpages.debian.org/testing/debconf/debconf-set-selections.1.en.html
---

**DEBCONF-SET-SELECTIONS(1)**	Debconf	**DEBCONF-SET-SELECTIONS(1)**

**名称**
debconf-set-selections - 向 debconf 数据库插入新值

**概要**
 debconf-set-selections 文件
 debconf-get-selections | ssh 新主机 debconf-set-selections

**描述**
debconf-set-selections 可用于为 debconf 数据库预设答案，或更改数据库中的答案。每个问题将被标记为“已查看”，以防止 debconf 以交互方式询问。

如果给定文件名，则从该文件读取；否则从标准输入读取。

**警告**
仅将此命令用于那些**将要安装或已经安装**的软件包的 debconf 值进行预填充。否则，您可能会在数据库中留下未安装软件包的、不会自动清除的配置值，或者导致涉及共享值的更严重问题。建议**仅在源机器具有完全相同的安装环境时**，才使用此命令为数据库预填充值。

**数据格式**
数据由一系列行组成。以 `#` 字符开头的行是注释。空行被忽略。所有其他行用于设置一个问题的值，应包含四个字段，每个字段之间用一个空白字符分隔。第一个字段是拥有该问题的软件包的名称。第二个字段是问题的名称。第三个字段是此问题的类型。第四个字段（直到行尾）是用于该问题答案的值。

或者，第三个字段可以是“seen”；这种情况下，预填充行仅控制该问题是否在 debconf 的数据库中被标记为“已查看”。注意，预填充问题的值**默认**会将问题标记为“已查看”，因此，要覆盖默认值**而不**将问题标记为“已查看”，您需要两行。

行尾添加 `\` 字符可将行延续到下一行。

**示例**
 # 将 debconf 优先级强制设为 critical。
 debconf debconf/priority select critical
 # 将默认前端覆盖为 readline，但允许用户选择。
 debconf debconf/frontend select readline
 debconf debconf/frontend seen false

**选项**
--verbose, -v
详细输出
--checkonly, -c
仅检查输入文件格式，不将更改保存到数据库

**另见**
debconf-get-selections(1)（在 debconf-utils 软件包中提供）

**作者**
Petter Reinholdtsen <pere@hungry.com>

**日期**
2025年3月10日
