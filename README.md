# 一、前言
 每次都是 AppImage，没找到 .deb 的安装包，琢磨着给他搞成.deb <br/>
 - 发现 AppImage 特性是可以解压（xxx.AppImage --appimage-extract）就是为了解决在不支持FUSE的系统上使用AppImage </br>
 - 进入解压后的目录（squashfs-root) AppRun，是可以运行的 但是得设置一下环境变量 export APPDIR=解压的目录
 - Yakit-xxx-xxx.AppImage 都是自动从官网（https://www.yaklang.com/）下载的, 我只是大自然的搬运工o(=•ェ•=)m

# 二、大概的流程：
主要涉及 项目/DEBIAN/* 下面涉及的几个重要的脚本就是 control、preinst、postinst、prerm、postrm  ，control 是一些包的信息，preinst、postinst、prerm、postrm 四个是维护脚本，项目/DEBIAN 的同级目录下的其他文件夹，就映射Linux真实的路径

<img width="230" height="366" alt="image" src="https://github.com/user-attachments/assets/9956ae3e-beeb-4c44-b62f-6032e07687b6" />

#### 安装逻辑：
  1. 检测程序是否已经在运行
  2. 先从官网获取最新的版本信息
  3. 创建安装目录 /usr/share/yakit/, 然后下载输出到目录，赋予执行权限
  4. 把 AppImage 解压，修改权限让其他用户可以访问，进入解压后的目录，修改一些目录权限和chrome-sandbox权限
  5. 创建启动脚本 /usr/bin/yakit，修改权限 ,
  6. 添加 .desktop 文件，更新缓存

#### 卸载
  卸载前判断一下进程是否在运行
 
```
依据这个执行顺序，检测进程是否在运行，只需要在 prerm 脚本中编写，
因为如果没有安装，就不存在进程在运行的情况
如果已经安装，再次执行，会被判定位更新，则第一个调用的就是 prerm 脚本
对于 preinst 我就用来检查一些必要的依赖。
```

# 三、编写原则
## (一)、DEBIAN/control 文件
- https://www.debian.org/doc/debian-policy/ch-controlfields.html#syntax-of-control-files （编写control的语法）
- https://www.debian.org/doc/debian-policy/ch-controlfields.html#description

- 续行必须以空格或制表符开头
```
folded
 The value of a folded field is a logical line that may span several lines. The lines after the first are called continuation lines and must start with a space or a tab. Whitespace, including any newlines, is not significant in the field values of folded fields. 3
```
<br />

- 但，又说不能用tab制表符，因为后果不可预测。https://www.debian.org/doc/debian-policy/ch-controlfields.html#description
```
The lines in the extended description can have these formats:

· Those starting with a single space are part of a paragraph. Successive lines of this form will be word-wrapped when displayed. The leading space will usually be stripped off. The line must contain at least one non-whitespace character.

· Those starting with two or more spaces. These will be displayed verbatim. If the display cannot be panned horizontally, the displaying program will line wrap them “hard” (i.e., without taking account of word breaks). If it can they will be allowed to trail off to the right. None, one or two initial spaces may be deleted, but the number of spaces deleted from each line will be the same (so that you can have indenting work correctly, for example). The line must contain at least one non-whitespace character.

· Those containing a single space followed by a single full stop character. These are rendered as blank lines. This is the only way to get a blank line. 9

· Those containing a space, a full stop and some more characters. These are for future expansion. Do not use them.

Do not use tab characters. Their effect is not predictable.
```

- 段落之间用 . 来分割 （用换行 + 空格、换行 + 制表符，实测是不行的，也符合上面说的）
```text
Stanza separators (empty lines), and lines consisting only of U+0020 SPACE and U+0009 TAB, are not allowed within field values or between fields. Empty lines in field values are usually escaped by representing them by a U+0020 SPACE followed by a U+002E ()..
```
<br />

- 文件必须用UTF-8编码
```text
All control files must be encoded in UTF-8.
```
<br />

- 注释：用 # 开头且前置没有任何前置空格
```text
Lines starting with U+0023 (), without any preceding whitespace, are comment lines that are only permitted in source package control files (). These comment lines are ignored, even between two continuation lines. They do not end logical lines.#debian/control
```
<br />


一个简单的：
```text
Package: fish
Version: 1.4.5-1
Architecture: amd64
Maintainer: 姓名 <email@address>
Section: net
Priority: optional
Depends: libcurl4t64 (= 8.18.0~rc3-1), libc6 (>= 2.34), zlib1g (>= 1:1.1.4)
Homepage: https://www.baidu.com
Description: 这是一个测试的软件包
 它并没有实际的的意义，取名为fish，专门用于测试 defconf配置脚本
 结束还必须回车换行，否则打包时候报错，在字段 Description 的值中间发有 EOF 字符(缺失结尾的换行符)

```

字段描述
```
 Package: 
 参考: https://www.debian.org/doc/debian-policy/ch-controlfields.html#package
 参考: https://www.debian.org/doc/debian-policy/ch-controlfields.html#s-f-source
 命名必须规范：小写字母、数字、+、-、.，至少2字符
 .
 Version: 版本号，有规范要求
 参考: https://www.debian.org/doc/debian-policy/ch-controlfields.html#version
 .
 Architecture: 一个唯一单一词用于标识 Debian 机器架构
 参考: https://www.debian.org/doc/debian-policy/ch-controlfields.html#architecture
 - any 匹配所有 Debian 机器架构
 - all 表示一个架构无关的封装。
 .
 Maintainer: 
 参考: https://www.debian.org/doc/debian-policy/ch-controlfields.html#maintainer
 格式: 姓名 <email@address> 格式符合 RFC822 标准
 .
 Sections: 
 参考 https://www.debian.org/doc/debian-policy/ch-archive.html#s-subsections 
 Debian 存档维护者提供了权威的节列表，我的理解就是官方的一些自带的分类
 .
 priorities: 
 参考 https://www.debian.org/doc/debian-policy/ch-archive.html#priorities
 分为：
    - required: 
        对于系统正常运行所必需的软件包（通常这意味着 dpkg 的功能依赖于这些软件包）。
        移除一个 required软件包可能导致你的系统完全损坏，
        甚至可能无法使用 dpkg来恢复，因此只有在你清楚自己在做什么的情况下才这样做。
        仅安装了 required软件包的系统至少具有足够的功能，使系统管理员能够启动系统并安装更多软件。
    - important: 
        重要的程序，包括那些在任何类 Unix 系统上都会期望找到的程序。
        如果一个有经验的 Unix 用户发现某个程序缺失时可能会说"这到底是怎么回事，foo在哪里？"，
        那么这个程序必须是一个 important软件包。其他没有它们系统就无法良好运行或使用的软件包也必须具有 important优先级。
        这不包括 Emacs、X Window 系统、TeX 或其他大型应用程序。important软件包只是一个常见期望和必要工具的最小集合。
    - standard: 
        这些软件包提供了一个相当精简但不过分受限的字符模式系统。
        如果用户没有选择其他选项，这将是默认安装的内容。它不包括许多大型应用程序。
    - optional (默认): 
        这是大多数归档软件包的默认优先级。
        除非一个软件包应该在标准 Debian 系统上默认安装，否则它应该具有 optional优先级。
        优先级为 optional的软件包可能会相互冲突
  Depends: 依赖
  参考: https://www.debian.org/doc/debian-policy/ch-controlfields.html#package-interrelationship-fields-depends-pre-depends-recommends-suggests-breaks-conflicts-provides-replaces-enhances
  参考: https://www.debian.org/doc/debian-policy/ch-relationships.html
```


详细具体的参考：<br />
https://www.debian.org/doc/debian-policy/ch-controlfields.html#debian-binary-package-control-files-debian-control

## (二)、维护脚本相关的
### (1)、执行先后顺序
我在每个脚本的开头放入类似以下代码，观察执行先后顺序以及 $1 的值是什么：
```bash
#!/bin/bash
set -e

echo "-----------"
echo "安装后执行的脚本 -> postinst"
echo "\$1的值: $1"
echo "-----------"
```

preinst、postinst、prerm、postrm 脚本，以及他们执行的先后顺序
- 安装前执行的脚本 -> preinst
- 安装后执行的脚本 -> postinst
- 卸载前执行的脚本 -> prerm
- 卸载后执行的脚本 -> postrm

正常安装流程：
- 安装前执行的脚本 -> preinst ($1的值: install)
- 安装后执行的脚本 -> postinst ($1的值: configure)

已经安装的情况下，又执行了一次 dpkg -i (顺序看着很奇怪，但是确实这样）
 - 卸载前执行的脚本 -> prerm ($1的值：upgrade)
 - 安装前执行的脚本 -> preinst ($1的值：upgrade)
 - 卸载后执行的脚本 -> postrm ($1的值：upgrade)
 - 安装后执行的脚本 -> postinst ($1的值: configure)

卸载
- 卸载前执行的脚本 -> prerm  ($1的值: remove)
- 卸载后执行的脚本 -> postrm ($1的值: remove)


### (2)、维护脚本的幂等性<br />
https://www.debian.org/doc/debian-policy/ch-maintainerscripts.html#maintainer-scripts-idempotency -> 6.2.Maintainer scripts idempotency<br />
✅ 成功后再运行：保持现状，不报错<br />
✅ 失败后重新运行：继续完成剩余工作<br />
❌ 不能重复：已经完成的操作<br />

### (3)、终端交互相关的
#### 1. 脚本应该保持安静，避免不必要的输出， 升级时不应再问同样的问题， 除非用户已经移除了包的 配置。
https://www.debian.org/doc/debian-policy/ch-binary.html#prompting-in-maintainer-scripts -> 3.9.1. Prompting in maintainer scripts<br />
原文:
- Packages should try to minimize the amount of prompting they need to do, and they should ensure that the user will only ever be asked each question once. This means that packages should try to use appropriate shared configuration files (such as and ), and shared debconf variables rather than each prompting for their own list of required pieces of information./etc/papersize/etc/news/server

- It also means that an upgrade should not ask the same questions again, unless the user has used to remove the package’s configuration. The answers to configuration questions should be stored in an appropriate place in so that the user can modify them, and how this has been done should be documented.dpkg --purge/etc

#### 2. 必须设计为能在无终端环境下工作、必须支持非交互式回退
https://www.debian.org/doc/debian-policy/ch-maintainerscripts.html -> 6.3 Controlling terminal for maintainer scripts<br />
原文：
- Maintainer scripts are not guaranteed to run with a controlling terminal and may not be able to interact with the user. They must be able to fall back to noninteractive behavior if no controlling terminal is available. Maintainer scripts that prompt via a program conforming to the Debian Configuration Management Specification (see Prompting in maintainer scripts) may assume that program will handle falling back to noninteractive behavior.

- For high-priority prompts without a reasonable default answer, maintainer scripts may abort if there is no controlling terminal. However, this situation should be avoided if at all possible, since it prevents automated or unattended installs. In most cases, users will consider this to be a bug in the package.

#### 3. 用 set -e 
https://www.debian.org/doc/debian-policy/ch-maintainerscripts.html -> 6.1. Introduction to package maintainer scripts<br />
原文:
- The package management system looks at the exit status from these scripts. It is important that they exit with a non-zero status if there is an error, so that the package management system can stop its processing. For shell scripts this means that you almost always need to use (this is usually true when writing shell scripts, in fact). It is also important, of course, that they exit with a zero status if everything went well.set -e

### (4)、defconf 使用相关
#### 1. 介绍
- https://wiki.debian.org/debconf<br />
原文翻译：<br />
- 简单来说，debconf 就是“正确安装 Shield Wizards Wizards”，这是基于 Debian 发行版的主要优势之一。
- 当你安装或升级包时，debconf会一次性问所有配置问题，并将答案存储在数据库中。然后当每个包安装自己时，脚本会利用数据库中的偏好设置。这样可以省去手动编辑配置文件的麻烦，也省去了等待每个软件包安装完再回答某些配置问题的麻烦。

> 看着，感觉 defconf 设置在升级的时候挺有用的，设置一个安装路径等配置，再次安装的时候，类似Windows安装的时候，设置安装路径，然后后续升级安装的时候，无需再次配置路径，路径显示的就是软件安装的路径。

#### 2. template 文件的编写（主要用来设置交互的提示信息，以及选项，可以设置支持多语言）
格式参考: https://manpages.debian.org/jessie/debconf-doc/debconf-devel.7.en.html#THE_TEMPLATES_FILE<br />
原文翻译：<br />
- 使用debconf的软件包可能会想问一些问题。这些 问题以模板形式存储在模板文件中。 和配置文件脚本一样，模板文件放在control.tar.gz部分 一个Deb。其格式类似于 Debian 控制文件;一组诗节 以空白行分隔，每节采用类似RFC822的形式

- 文件路径 DEBAIN/templates
参考: http://www.fifi.org/doc/debconf-doc/tutorial.html#AEN34<br />
原文:<br />
- Start writing a debian/templates file. Each time you find a piece of output or a question, add it to the file as a new template. The format of this file is simple and quite similar to a Debian control file:

#### 3. config 文件 (用来设置提问的问题）
- 提问的问题需要在config 文件中，而不是在 postinst 脚本中
参考: http://www.fifi.org/doc/debconf-doc/tutorial.html#AEN113<br />
原文:<br />
- Next, decide what order the questions should be asked and the messages to the user should be displayed, figure out what tests you'll make before asking the questions and displaying the messages, and start writing a debian/config file to ask and display them.
  - Note: These questions are asked by a separate config script, not by the postinst, so the package can be configured before it is installed, or reconfigured after it is installed. Do not make your postinst use debconf to ask questions.
 


#### 4. 设置支持多语言(国际化）
参考： http://www.fifi.org/doc/debconf-doc/tutorial.html <br />
AI翻译：https://github.com/CreatNULL/yakit-deb/blob/main/debconf/AI%E7%BF%BB%E8%AF%91-Debconf%20%E7%A8%8B%E5%BA%8F%E5%91%98%E6%95%99%E7%A8%8B-debconf-doc-tutorial.md#%E6%9C%AC%E5%9C%B0%E5%8C%96%E6%A8%A1%E6%9D%BF%E6%96%87%E4%BB%B6《br />


当使用 debconf-getlang 会提示：
```bash
# debconf-getlang：此实用程序已弃用；您应该切换到使用po-debconf包
debconf-getlang: This utility is deprecated; you should switch to using the po-debconf package.
```

```bash
apt-get install po-debconf
```

```bash
┌──(root㉿kali)-[/home/…/generate_deb/project/fishf/DEBIAN]
└─# po2debconf 
Usage: po2debconf [options] master
Options:
  -h,  --help             display this help message
  -v,  --verbose          enable verbose mode
  -o,  --output=FILE      specify output file (Default: stdout)
  -e,  --encoding=STRING  convert encoding, STRING is chosen between
                        po: no conversion
                      utf8: convert to UTF-8
                   popular: change encoding according to file map found
                            in PODEBCONF_ENCODINGS environment variable
                            (Default, map is /usr/share/po-debconf/encodings)
               traditional: obsolete, replaced by popular
       --podir=DIR        specify PO output directory
                          (Default: <master directory>/po)
```

生成 POTFILES.in ,该文件告诉在所有程序源代码中，哪些文件有需要翻译的标记字符串 <br />
参考: https://www.gnu.org/software/gettext/manual/html_node/po_002fPOTFILES_002ein.html


#### 3. 看看，有啥命令 `/usr/share/debconf/confmodule`, 看到 db_set、db_input 等等

```bash
┌──(vbgaga㉿kali)-[~]
└─$ tail -n 35  /usr/share/debconf/confmodule

db_capb ()      { _db_cmd "CAPB $@"; }
db_set ()       { _db_cmd "SET $@"; }
db_reset ()     { _db_cmd "RESET $@"; }
db_title ()     { _db_cmd "TITLE $@"; }
db_input ()     { _db_cmd "INPUT $@"; }
db_beginblock () { _db_cmd "BEGINBLOCK $@"; }
db_endblock ()  { _db_cmd "ENDBLOCK $@"; }
db_go ()        { _db_cmd "GO $@"; }
db_get ()       { _db_cmd "GET $@"; }
db_register ()  { _db_cmd "REGISTER $@"; }
db_unregister () { _db_cmd "UNREGISTER $@"; }
db_subst ()     { _db_cmd "SUBST $@"; }
db_fset ()      { _db_cmd "FSET $@"; }
db_fget ()      { _db_cmd "FGET $@"; }
db_purge ()     { _db_cmd "PURGE $@"; }
db_metaget ()   { _db_cmd "METAGET $@"; }
db_version ()   { _db_cmd "VERSION $@"; }
db_clear ()     { _db_cmd "CLEAR $@"; }
db_settitle ()  { _db_cmd "SETTITLE $@"; }
db_previous_module () { _db_cmd "PREVIOUS_MODULE $@"; }
db_info ()      { _db_cmd "INFO $@"; }
db_progress ()  { _db_cmd "PROGRESS $@"; }
db_data ()      { _db_cmd "DATA $@"; }
db_x_loadtemplatefile ()        { _db_cmd "X_LOADTEMPLATEFILE $@"; }

# An old alias for input.
db_text () {
        db_input $@
}

# Cannot read a return code, since there is none and it would block.
db_stop () {
        echo STOP >&3
}
```
对于这些命令的详细解释：<br />
- https://manpages.debian.org/jessie/debconf-doc/debconf-devel.7.en.html#THE_DEBCONF_PROTOCOL 文档中有详细描述


#### 4. /var/cache/debconf/config.dat（存储所的回答） /var/cache/debconf/templates.dat （存储问题的模板定义）
https://stackoverflow.com/questions/10885177/how-to-read-input-while-installing-debian-package-on-debian-systems<br />

#### 5. 一些相关的命令
```
# 查看已保存的配置
debconf-get-selections

# 查看特定包的配置
debconf-get-selections | grep openssh-server

# 导出配置用于自动化部署
debconf-get-selections > debconf-selections.txt

# 重新配置软件包
https://www.tecmint.com/dpkg-reconfigure-installed-package-in-ubuntu-debian/
sudo dpkg-reconfigure package-name

# 设置 debconf 值
# https://askubuntu.com/questions/381593/how-to-use-debcondf-show-results-with-debconf-set-selections/557837#557837
echo "package-name question-name value" | sudo debconf-set-selections

# 删除一个配置从数据库 （debconf-communicate 相关的命令仅仅用于调试，禁止使用在维护脚本中）
https://manpages.debian.org/testing/debconf/debconf-communicate.1.en.html （命令手册）
https://serverfault.com/questions/332459/how-do-i-delete-values-from-the-debconf-database （使用示例）
echo PURGE | sudo debconf-communicate packagename
```

## (三)、.desktop 编写
官方文档:  https://specifications.freedesktop.org/desktop-entry/latest/recognized-keys.html <br />


StartupWMClass 获取：https://www.cnblogs.com/swtjavaspace/p/18188551<br />
`xprop | grep WM_CLASS` 然后点击对应窗口，终端会显示


## 编写参考文档
 - https://leux.cn/doc/Debian%E5%88%B6%E4%BD%9CDEB%E5%8C%85%E7%9A%84%E6%96%B9%E6%B3%95.html
 - https://blog.csdn.net/weixin_42267862/article/details/138808742
 - https://www.debian.org/doc/debian-policy/ch-controlfields.html
 - https://wiki.debian.org/MaintainerScripts (重复安装，正常安装，安装失败 执行流程图）
 - https://www.debian.org/doc/manuals/packaging-tutorial/packaging-tutorial.zh_CN.pdf
 - https://www.cnblogs.com/swtjavaspace/p/18188551 （.desktop 的StartupWMClass 值的获取）
 - https://geek-blogs.com/blog/linux-run-appimage/ （AppImage的解压）
 - https://www.oryoy.com/news/ubuntu-debconf-quan-gong-lve-qing-song-jie-jue-xi-tong-pei-zhi-nan-ti.html
 - https://wiki.debian.org/debconf
 - https://www.oryoy.com/news/ubuntu-xin-shou-bi-kan-qing-song-zhang-wo-debconf-pei-zhi-ji-qiao-gao-bie-xi-tong-she-zhi-nan-ti.html
 - http://www.fifi.org/doc/debconf-doc/tutorial.html
 - https://linux.extremeoverclocking.com/man/3/confmodule
 - https://www.tecmint.com/dpkg-reconfigure-installed-package-in-ubuntu-debian
 - https://manpages.debian.org/testing/debconf/debconf-communicate.1.en.html
