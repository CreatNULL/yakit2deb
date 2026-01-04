https://www.debian.org/doc/packaging-manuals/copyright-format/1.0/
---

机器可读文件 debian/copyright
版本 1.0
Debian Policy 4.7.3.0, 2025-12-23

在保留本声明的前提下，允许在任何媒介中复制和分发本文件，无论是否经过修改。

摘要

为 Debian 软件包内的文件建立一种标准的机器可读格式，以便对软件包和软件包集合的许可证进行自动化检查和报告。本规范最初起草为 DEP-5。debian/copyright

目录

1. 引言
2. 缘由
3. 致谢
4. 文件语法
4.1. 单行值
4.2. 以空格分隔的列表
4.3. 基于行的列表
4.4. 格式化文本
5. 节
5.1. 头部节（一次）
5.2. 文件节（可重复）
5.3. 独立的许可证节（可选，可重复）
6. 字段
6.1. 格式
6.2. 上游名称
6.3. 上游联系
6.4. 来源
6.5. 免责声明
6.6. 注释
6.7. 许可证
6.8. 版权
6.9. 文件
7. 许可证规范
7.1. 简称
7.2. 语法
7.3. SPDX
8. 示例

1. 引言
本文档描述了一种标准的机器可解释的 `debian/copyright` 文件格式。这个文件是 Debian 打包中最重要的文件之一，但在此规范之前，没有为其定义标准格式，其内容在不同软件包中差异巨大。这使得自动提取许可信息变得困难。

本规范的使用是可选的。

本提案中的任何内容都不应取代或修改《Debian Policy》中关于在 `debian/copyright` 中记录版权和许可状态时应使用的适当详细程度或粒度。

2. 缘由
自由软件许可证的多样性意味着 Debian 不仅需要关注特定作品的自由程度，还需要关注其许可证与所使用的 Debian 其他部分的兼容性。

GPL 第 3 版的发布、其与第 2 版的不兼容性，以及我们无法识别哪些软件可能存在兼容性问题，是这一局限性的突出表现。

更早的先例也存在，例如 GPL/OpenSSL 不兼容性。除了通过 `grep` 命令查找 `debian/copyright` 文件（这容易产生许多误报（例如打包遵循 GPL 但软件使用其他许可证）或漏报（GPL 软件但带有“OpenSSL 特殊例外”双许可形式）），没有可靠的方法知道 Debian 中哪些软件可能有问题。

问题还会继续出现。例如，在 CDDL 操作系统（如 Nexenta）上分发仅遵循 GPLv2 的软件存在问题。GPL 第 3 版解决了这个问题，但并非所有 GPL 软件都能切换到第 3 版，而且我们无法知道应从此类系统中剥离多少 Debian 软件。

即使在许可证符合 DFSG 且彼此兼容的情况下，用户也可能希望有一种方法来识别遵循特定许可证的软件（例如，如果他们因特殊原因希望避免某些许可证）。

3. 致谢
多年来，许多人参与了本规范的制定。以下按字母顺序排列的列表并不完整；请建议遗漏的人员：Russ Allbery, Ben Finney, Sam Hocevar, Steve Langasek, Charles Plessy, Noah Slater, Jonas Smedegaard, Lars Wirzenius.

4. 文件语法
`debian/copyright` 文件必须是机器可解释的，同时人类可读，并能传达所有必需的上游信息、版权声明和许可详细信息。

该文件的语法与其他 Debian 控制文件相同，具体遵循《Debian Policy Manual》中的规定。详见其第 5.1 节。可以向任何节添加额外字段。无需也不建议添加前缀，但请避免使用与标准字段相似的名字，以便更容易发现错误。本规范的未来版本将尝试避免为广泛使用的额外字段指定冲突的规范。

文件由两个或更多节组成。文件必须至少包含一个头部节和一个文件节。

字段有四种类型。本文档中每个字段的定义指明了其取值类型。

4.1. 单行值
单行字段的整个值必须位于一行。例如，`Format` 字段具有一个指定所用机器可读格式版本的单行值。

4.2. 以空格分隔的列表
定义为以空格分隔列表的字段值可以位于一行或多行。列表中的值由一个或多个空白字符（空格、制表符或换行符）分隔。例如，`Files` 字段包含一个以空格分隔的文件名模式列表。

4.3. 基于行的列表
基于行的列表每行一个值。例如，`Upstream-Contact` 字段包含一个基于行的联系地址列表。

4.4. 格式化文本
格式化文本字段使用与 Debian 控制文件中软件包的 `Description` 字段的详细描述相同的规则。

在某些情况下，第一行可能具有特殊含义作为概要，类似于 `Description` 字段使用第一行作为简短描述。详见《Debian Policy》第 5.6.13 节“描述”。例如，`Disclaimer` 是一个没有特殊第一行的格式化文本字段，而 `License` 是一个格式化文本字段，其第一行指示许可证的简称。

5. 节
有三种类型的节。文件中的第一个节称为头部节。其他每个节要么是文件节，要么是独立的许可证节。这类似于 `debian/control` 文件中的源码和二进制包节。

5.1. 头部节（一次）
以下字段可以出现在头部节中。

Format: 必需。
Upstream-Name: 可选。
Upstream-Contact: 可选。
Source: 可选。
Disclaimer: 可选。
Comment: 可选。
License: 可选。
Copyright: 可选。

头部节中的 `Copyright` 和 `License` 字段可以补充但不能替代文件节中的字段。如果存在，它们总结整个软件包的版权声明或再分发条款。

例如，当一个作品在宽松许可证和 Copyleft 许可证下都有授权时，`License` 可用于阐明组合的许可条款。`Copyright` 和 `License` 一起也可用于记录汇编作品的版权和许可证。

在头部节中使用 `License` 字段而不伴随 `Copyright` 字段是有效的，但仅有 `Copyright` 是不够的。

5.1.1. 头部节示例
```
Format: https://www.debian.org/doc/packaging-manuals/copyright-format/1.0/
Source: https://www.example.com/software/project
Upstream-Name: SOFTware
Upstream-Contact: John Doe <john.doe@example.com>
```

5.2. 文件节（可重复）
文件的版权和许可证声明可以由一个或多个节组成。在最简单的情况下，可以使用一个带有 `Files: *` 的节来声明整个软件包的许可证和版权。只有 Debian 存档所需的许可证和版权信息需要在此列出。

以下字段可以出现在文件节中。

Files: 必需。
Copyright: 必需。
License: 必需。
Comment: 可选。

5.2.1. 文件节示例
```
Files:
 *
Copyright: 1975-2010 Ulla Upstream
License: GPL-2+

Files:
 debian/*
Copyright: 2010 Daniela Debianizer
License: GPL-2+

Files:
 debian/patches/fancy-feature
Copyright: 2010 Daniela Debianizer
License: GPL-3+

Files:
 */*.1
Copyright: 2010 Manuela Manpager
License: GPL-2+
```
在此示例中，所有文件的版权都由上游持有，并且该版权持有者根据 GPL 第 2 版或更高版本授予许可。有三个例外。所有 Debian 打包文件的版权由打包者持有，此外，一个提供新功能的特定文件具有不同的许可授权。最后，包中添加了一些手册页，其版权由第三方持有。

由于手册页的许可证与包中大多数其他文件相同，上面的最后一个节可以与第一个节合并，在一个 `Copyright` 字段中列出两个版权声明。是否将具有相同许可授权的节合并，由 `debian/copyright` 文件的作者自行决定。

5.3. 独立的许可证节（可选，可重复）
独立的 `License` 节可用于为给定许可证提供完整的许可证文本，而不是在每个引用它的 `Files` 节中重复。

`License` 字段的概要（第一行）必须是单个许可证简称，或简称后跟一个许可证例外。

以下字段可以出现在独立的许可证节中。

License: 必需。
Comment: 可选。

示例 1. 三重许可文件
```
Files: src/js/editline/*
Copyright: 1993, John Doe
           1993, Joe Average
License: MPL-1.1 or GPL-2 or LGPL-2.1

License: MPL-1.1
 [LICENSE TEXT]

License: GPL-2
 [LICENSE TEXT]

License: LGPL-2.1
 [LICENSE TEXT]
```

示例 2. 重复使用的许可证
```
Files:
 src/js/editline/*
Copyright: 1993, John Doe
           1993, Joe Average
License: MPL-1.1

Files:
 src/js/fdlibm/*
Copyright: 1993, J-Random Corporation
License: MPL-1.1

License: MPL-1.1
 [LICENSE TEXT]
```

6. 字段
以下字段是为 `debian/copyright` 定义的。

6.1. 格式
单行：格式规范的 URI。对于本文档的当前版本，应使用的字段是：

```
Format: https://www.debian.org/doc/packaging-manuals/copyright-format/1.0/
```

本规范的最初版本使用此 URL 的非 HTTPS 版本作为其 URI：

```
Format: http://www.debian.org/doc/packaging-manuals/copyright-format/1.0/
```
两个版本都有效，并指向同一规范，解析器应将两者都解释为引用同一格式。优先使用 https URI。

6.2. 上游名称
单行：上游为软件使用的名称。

6.3. 上游联系
基于行的列表：联系上游项目的首选地址。可以是自由格式的文本，但按照惯例通常写为 RFC5322 地址或 URI 的列表。

6.4. 来源
格式化文本，无概要：关于上游源码来源的说明。通常这是一个 URL，但也可能是自由格式的说明。《Debian Policy》第 12.5 节要求提供此信息，除非没有上游源码，这主要适用于本机 Debian 软件包。如果上游源码已被修改以移除非自由部分，则应在此字段中说明。

6.5. 免责声明
格式化文本，无概要：此字段用于非自由或 contrib 软件包，以声明它们不属于 Debian 并说明原因（参见《Debian Policy》第 12.5 节）。

6.6. 注释
格式化文本，无概要：此字段可以提供额外信息。例如，可以引用上游的电子邮件，说明为何许可证可被主存档接受；或者解释此版本的软件包如何从已知符合 DFSG 的版本分叉而来，尽管当前上游版本不符合。

6.7. 许可证
格式化文本，带概要。

在头部节中，此字段给出整个软件包的许可信息，可能与所有按文件许可信息的组合不同或更简化。在文件节中，此字段给出此节 `Files` 字段所列文件的许可条款。在独立的许可证节中，它为引用它的节提供许可条款。

第一行（概要）：许可证的缩写名称，或给出替代方案的表达式（有关标准缩写的列表，请参阅“简称”部分）。如果软件包中存在没有标准简称的许可证，可以为这些许可证分配任意的简称。这些任意名称仅保证在单个版权文件中是唯一的。

如果没有剩余的行，则概要中的所有简称或简称后跟许可证例外必须在独立的许可证节中进行描述。否则，此字段应包含许可证的完整文本，或包含指向 `/usr/share/common-licenses` 下许可证文件的指针。此字段应包含履行《Debian Policy》关于包含软件分发许可证副本（12.5）要求以及任何许可证要求（如包含免责声明或其他通知）所需的所有文本。

6.8. 版权
格式化文本，无概要：一个或多个自由格式的版权声明。允许任何格式；请参见下面的示例，了解如何构建此字段以使其更易于阅读的一些思路。在头部节中，此字段给出整个软件包的版权信息，可能与所有按文件版权信息的组合不同或更简化。在文件节中，它为与 `Files` 模式匹配的文件提供适用的版权信息。如果作品没有版权持有者（即属于公共领域），则应在此处记录。

`Copyright` 字段收集此节文件的所有相关版权声明。并非所有版权声明都适用于每个单独的文件，并且一个版权所有者的出版年份可以汇总在一起。例如，如果文件 A 有：
```
Copyright 2008 John Smith
Copyright 2009 Angela Watts
```
文件 B 有：
```
Copyright 2010 Angela Watts
```
则仍可以使用单个节涵盖两个文件。该节的 `Copyright` 字段将包含：
```
Copyright 2008 John Smith
Copyright 2009, 2010 Angela Watts
```

`Copyright` 字段可以包含完全复制的原始版权声明（包括“Copyright”一词），也可以按照上述方式缩短文本或与其他版权声明合并，只要不丢失信息即可。本规范中的示例使用了两种形式。

6.9. 文件
以空格分隔的列表：指示此节指定的许可证和版权所涵盖的文件模式列表。

`Files` 字段中的文件名模式使用简化的 shell glob 语法指定。模式由空白字符分隔。

只有通配符 `*` 和 `?` 适用；前者匹配任意数量的字符（包括零个），后者匹配单个字符。两者都匹配斜杠 (`/`) 和前导点，这与 shell glob 不同。因此，模式 `*.in` 匹配源树中任何位置以 `.in` 结尾的任何文件，而不仅仅是顶层目录。

模式匹配从源树根目录开始的路径名。因此，“Makefile.in”仅匹配树根目录下的文件，但“*/Makefile.in”匹配任意深度。

反斜杠 (`\`) 用于移除下一个字符的特殊含义；参见下表。

转义序列	匹配
`\*`	星号 (asterisk)
`\?`	问号
`\\`	反斜杠
反斜杠后跟任何其他字符都是错误的。

这是与不设置 `FNM_PATHNAME` 标志的 fnmatch(3) 相同的模式语法，或 GNU find 命令的 `-path` 测试参数，只是不识别 `[...]` 通配符。

允许多个 `Files` 节。最后一个与特定文件匹配的节适用于该文件。因此，应先给出更通用的节，然后是更具体的覆盖。

仅通过添加 `Files` 节来覆盖先前的匹配，从而支持排除。

此语法不区分文件名和目录名；模式中的尾部斜杠永远不会匹配任何实际路径。可以使用像“foo/*”这样的模式选择整个目录树。

用于分隔模式的空格字符不能通过反斜杠转义。可以使用像“foo?bar”这样的模式选择“foo bar”这样的路径。

7. 许可证规范
7.1. 简称
机器可解析的版权文件的大部分价值在于能够关联多个软件的许可证。为此，本规范为许多常用许可证定义了标准简称，可用于 `License` 字段的概要（第一行）中。

这些简称在所有使用此文件格式的地方都具有指定的含义，不得用于指代任何其他许可证。因此，解析器可以依赖这些简称无论出现在何处都指代相同的许可证，而无需解析或比较完整的许可证文本。

许可证可能会不时地添加到标准简称列表中或从中移除。此类更改总是伴随着本标准的版本和推荐的 `Format` 值的更改。解析版权文件的实现者应注意不要对未知版本的许可证简称的含义做任何假设。

使用标准简称不覆盖《Debian Policy》关于在 `debian/copyright` 中包含完整许可证文本的要求，也不覆盖作品许可证中关于复制法律声明的任何要求。此信息仍必须包含在 `License` 字段中，无论是在独立的许可证节中，还是在相关的文件节中。

对于有多个使用版本的许可证，简称由许可证系列的通用简称，后跟连字符和版本号组成。如果省略版本号，则暗示最低版本号。当许可授予允许使用该许可证任何更高版本的条款时，在简称末尾添加加号。例如，简称 `GPL` 指 GPL 第 1 版，等同于 `GPL-1`，但后者更清晰，因此更受推荐。如果软件包可以根据 GPL 第 1 版或任何更高版本分发，则使用简称 `GPL-1+`。

为了与 SPDX 兼容，带尾随点零的版本被视为等同于不带尾随点零的版本（例如，“2.0.0”等同于“2.0”和“2”）。

目前，许可证的完整文本仅在 SPDX 开源许可证注册表中提供。

关键词	含义
`public-domain`	任何用途都不需要许可证；作品在任何司法管辖区均不受版权保护。
`Apache`	Apache 许可证 1.0, 2.0。
`Artistic`	Artistic 许可证 1.0, 2.0。
`BSD-2-clause`	伯克利软件分发许可证，2-clause 版本。
`BSD-3-clause`	伯克利软件分发许可证，3-clause 版本。
`BSD-4-clause`	伯克利软件分发许可证，4-clause 版本。
`ISC`	Internet 软件联盟，有时也称为 OpenBSD 许可证。
`CC-BY`	Creative Commons 署名许可证 1.0, 2.0, 2.5, 3.0。
`CC-BY-SA`	Creative Commons 署名-相同方式共享许可证 1.0, 2.0, 2.5, 3.0。
`CC-BY-ND`	Creative Commons 署名-禁止演绎许可证 1.0, 2.0, 2.5, 3.0。
`CC-BY-NC`	Creative Commons 署名-非商业性使用许可证 1.0, 2.0, 2.5, 3.0。
`CC-BY-NC-SA`	Creative Commons 署名-非商业性使用-相同方式共享许可证 1.0, 2.0, 2.5, 3.0。
`CC-BY-NC-ND`	Creative Commons 署名-非商业性使用-禁止演绎许可证 1.0, 2.0, 2.5, 3.0。
`CC0`	Creative Commons Zero 1.0 Universal。构成简称时省略许可证版本号中的“Universal”。
`CDDL`	通用开发与分发许可证 1.0。
`CPL`	通用公共许可证。
`EFL`	Eiffel 论坛许可证 1.0, 2.0。
`Expat`	Expat 许可证。
`GPL`	GNU 通用公共许可证 1.0, 2.0, 3.0。
`LGPL`	GNU 较宽松通用公共许可证 2.1, 3.0，或 GNU 库通用公共许可证 2.0。
`GFDL`	GNU 自由文档许可证 1.0, 1.1, 1.2, 1.3。如果没有封面文本、封底文本或不变章节，请使用 `GFDL-NIV`。
`GFDL-NIV`	GNU 自由文档许可证，无封面文本、封底文本或不变章节。使用与 `GFDL` 相同的版本号。
`LPPL`	LaTeX 项目公共许可证 1.0, 1.1, 1.2, 1.3c。
`MPL`	Mozilla 公共许可证 1.1。
`Perl`	Perl 许可证（改用“GPL-1+ or Artistic-1”）。
`Python`	Python 许可证 2.0。
`QPL`	Q 公共许可证 1.0。
`W3C`	W3C 软件许可证。更多信息，请参阅 W3C 知识产权常见问题。
`Zlib`	zlib/libpng 许可证。
`Zope`	Zope 公共许可证 1.0, 1.1, 2.0, 2.1。
MIT 许可证有许多版本。当匹配时，请使用 Expat。

通过将“with 关键字 exception”附加到简称上来表示许可证的例外或澄清。本文档提供了一个关键字列表，在提及最常见的例外时必须使用。当适用于通用许可证的例外是授予额外权限而非添加限制时，可以使用未从以下关键字列表中选取的任意关键字。当许可证由于添加了限制而不同于通用许可证时，应使用一个不同的简称，而不是“with 关键字 exception”。

在一个许可证规范内，每个许可证只能指定一个例外。如果有多个例外适用于单个许可证，则必须使用指示该多个例外组合的任意简称。

GPL 的“字体”例外指添加到每个文件许可证声明中的文本，具体规定见“GPL 如何适用于字体”。与此例外对应的精确文本是：
```
As a special exception, if you create a document which uses this font,
and embed this font or unaltered portions of this font into the
document, this font does not by itself cause the resulting document to
be covered by the GNU General Public License. This exception does not
however invalidate any other reasons why the document might be covered
by the GNU General Public License. If you modify this font, you may
extend this exception to your version of the font, but you are not
obligated to do so. If you do not wish to do so, delete this exception
statement from your version.
```

GPL 的“OpenSSL”例外授予将遵循 GPL 的代码与包含不兼容 GPL 条款的 OpenSSL 库链接的权限。更多信息，请参阅 Mark McLoughlin 的《OpenSSL 许可证和 GPL》以及 Mark McLoughlin 在 debian-legal 邮件列表上的消息“中间软件许可与 OpenSSL 冲突”。与此例外对应的文本是：
```
In addition, as a special exception, the copyright holders give
permission to link the code of portions of this program with the
OpenSSL library under certain conditions as described in each
individual source file, and distribute linked combinations including
the two.

You must obey the GNU General Public License in all respects for all
of the code used other than OpenSSL. If you modify file(s) with this
exception, you may extend this exception to your version of the
file(s), but you are not obligated to do so. If you do not wish to do
so, delete this exception statement from your version. If you delete
this exception statement from all source files in the program, then
also delete it here.
```

7.1.1. 公共领域
简称 `public-domain` 不指代一组许可条款。有些作品在任何司法管辖区均不受版权保护，因此对于版权法所涵盖的任何目的都不需要许可证。此简称是对相关文件“属于公共领域”的明确声明。

对版权和公共领域的普遍误解导致一种常见的情况，即错误地声称作品属于公共领域。关于公共领域的维基百科文章是此主题的有用参考。

当节中的 `License` 字段的简称为 `public-domain` 时，该字段的剩余行必须准确解释该节对应的文件在默认版权限制下具有何种豁免。

7.2. 语法
许可证名称不区分大小写，且不能包含空格。

在多许可证的情况下，当用户可以在不同许可证之间选择时，许可证简称用“or”分隔；当作品的使用必须同时遵守多个许可证的条款时，用“and”分隔。

例如，这是一个简单的“GPL 第 2 版或更高版本”字段：
```
License: GPL-2+
```
这是像 Perl 这样的 GPL/Artistic 双重许可作品：
```
License: GPL-1+ or Artistic
```
这是用于一个同时包含 GPL 和经典 BSD 代码的文件：
```
License: GPL-2+ and BSD-3-clause
```
对于最复杂的情况，使用逗号来消除“or”和“and”的优先级。“and”的优先级高于“or”，除非前面有逗号。例如：
`A or B and C` 表示 `A or (B and C)`。
`A or B, and C` 表示 `(A or B) and C`。

这是用于一个包含 Perl 代码和经典 BSD 代码的文件：
```
License: GPL-2+ or Artistic-2.0, and BSD-3-clause
```
具有“OpenSSL 例外”的作品实际上是一个双重许可作品，可以根据“GPL-2+”分发，也可以根据带有“OpenSSL 例外”的“GPL-2+”分发。因此表达为“GPL-2+ with OpenSSL exception”。此类许可证可能的 `License` 字段是：
```
License: GPL-2+ with OpenSSL exception
 This program is free software; you can redistribute it and/or modify
 it under the terms of the GNU General Public License as published by
 the Free Software Foundation; either version 2 of the License, or
 (at your option) any later version.
 .
 In addition, as a special exception, the author of this program gives
 permission to link the code of its release with the OpenSSL project's
 "OpenSSL" library (or with modified versions of it that use the same
 license as the "OpenSSL" library), and distribute the linked executables.
 You must obey the GNU General Public License in all respects for all of
 the code used other than "OpenSSL".  If you modify this file, you may
 extend this exception to your version of the file, but you are not
 obligated to do so.  If you do not wish to do so, delete this exception
 statement from your version.
 .
 This program is distributed in the hope that it will be useful,
 but WITHOUT ANY WARRANTY; without even the implied warranty of
 MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 GNU General Public License for more details.
 .
 You should have received a copy of the GNU General Public License
 along with this package; if not, see <https://www.gnu.org/licenses/>.
Comment:
 On Debian systems, the full text of the GNU General Public License
 version 2 can be found in the file '/usr/share/common-licenses/GPL-2'.
```

7.3. SPDX
SPDX 是一种标准化格式的尝试，用于传达与软件包相关的组件、许可证和版权。它与机器可读的 `debian/copyright` 格式试图保持一定程度的兼容。然而，这两种格式有不同的目标，因此格式也不同。DEP5 维基页面将用于跟踪这些差异。

8. 示例
示例 3. 简单

一个程序“X Solitaire”（在 Debian 源码包 `xsol` 中分发）可能的 `debian/copyright` 文件（这不是实际软件包的完整或正确的版权文件）：
```
Format: https://www.debian.org/doc/packaging-manuals/copyright-format/1.0/
Source: ftp://ftp.example.com/pub/games
Upstream-Name: X Solitaire

Files:
 *
Copyright: 1998 John Doe <jdoe@example.com>
   1998 Jane Smith <jsmith@example.net>
License: GPL-2+
 This program is free software; you can redistribute it and/or modify
 it under the terms of the GNU General Public License as published by
 the Free Software Foundation; either version 2 of the License, or
 (at your option) any later version.
 .
 This program is distributed in the hope that it will be useful,
 but WITHOUT ANY WARRANTY; without even the implied warranty of
 MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 GNU General Public License for more details.
 .
 You should have received a copy of the GNU General Public License
 along with this package; if not, see <https://www.gnu.org/licenses/>.
Comment:
 On Debian systems, the full text of the GNU General Public License
 version 2 can be found in the file '/usr/share/common-licenses/GPL-2'.
```

示例 4. 复杂

一个程序“Planet Venus”（在 Debian 源码包 `planet-venus` 中分发）可能的 `debian/copyright` 文件（这不是实际软件包的完整或正确的版权文件）：
```
Format: https://www.debian.org/doc/packaging-manuals/copyright-format/1.0/
Source: https://www.example.com/code/venus
Upstream-Name: Planet Venus
Upstream-Contact: John Doe <jdoe@example.com>

Files:
 *
Copyright: 2008, John Doe <jdoe@example.com>
           2007, Jane Smith <jsmith@example.org>
           2007, Joe Average <joe@example.org>
           2007, J. Random User <jr@users.example.com>
License: PSF-2

Files:
 debian/*
Copyright: 2008, Dan Developer <dan@debian.example.com>
License: permissive
 Copying and distribution of this package, with or without modification,
 are permitted in any medium without royalty provided the copyright notice
 and this notice are preserved.

Files:
 debian/patches/theme-diveintomark.patch
Copyright: 2008, Joe Hacker <hack@example.org>
License: GPL-2+

Files:
 planet/vendor/compat_logging/*
Copyright: 2002, Mark Smith <msmith@example.org>
License: MIT
 [LICENSE TEXT]

Files:
 planet/vendor/httplib2/*
Copyright: 2006, John Brown <brown@example.org>
License: MIT2
 Unspecified MIT style license.

Files:
 planet/vendor/feedparser.py
Copyright: 2007, Mike Smith <mike@example.org>
License: PSF-2

Files:
 planet/vendor/htmltmpl.py
Copyright: 2004, Thomas Brown <coder@example.org>
License: GPL-2+

License: PSF-2
 [LICENSE TEXT]

License: GPL-2+
 This program is free software; you can redistribute it and/or modify
 it under the terms of the GNU General Public License as published by
 the Free Software Foundation; either version 2 of the License, or
 (at your option) any later version.
 .
 This program is distributed in the hope that it will be useful,
 but WITHOUT ANY WARRANTY; without even the implied warranty of
 MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 GNU General Public License for more details.
 .
 You should have received a copy of the GNU General Public License
 along with this package; if not, see <https://www.gnu.org/licenses/>.
Comment:
 On Debian systems, the full text of the GNU General Public License
 version 2 can be found in the file '/usr/share/common-licenses/GPL-2'.
```
