# 一、前言
 如cs之类的，有时候换个电脑，换个服务器，分享给朋友，还得配置权限，卸载的时候还得回忆一下自己装哪里了，找的都是破解版，又没有.deb 安装包，迁移卸载麻烦哦。要是有deb多好，哈基咪~ 
 
 一、基础知识
参考: https://leux.cn/doc/Debian%E5%88%B6%E4%BD%9CDEB%E5%8C%85%E7%9A%84%E6%96%B9%E6%B3%95.html
原文：
```
# DEB包就是把文件放到根目录对应的路径里，然后再在包管理器里注册一下
# DEB包通常遵循一定的文件结构，包括以下主要部分：

控制信息		DEBIAN/control	 包含软件包的的包名、版本、架构、作者、描述等
配置文件		DEBIAN/conffiles 软件包配置文件的路径，这些文件在软件包升级时会特别处理，以确保用户自定义的配置不会丢失
装前脚本		DEBIAN/preinst	 在解包 data.tar.gz 之前执行的脚本
装后脚本		DEBIAN/postinst	 在软件包安装后要执行的命令，可以用于执行清理工作、添加启动脚本等操作
卸前脚本		DEBIAN/prerm	 卸载时，在删除 实际数据 之前运行的脚本
卸后脚本		DEBIAN/postrm	 在软件包卸载后要执行的命令，可以用于执行卸载后的清理操作
实际数据		usr/ 		 软件包的实际文件和目录结构，这些文件会被释放到系统根目录对应的路径里
文档文件		usr/share/doc	 一些软件包可能包括文档文件，这些文件通常存储在该目录下

# 注意：必须要有一个DEBIAN文件夹(权限755)，最精简的状态下只需要有 control 文件就可以了，其余文件可以不创建
```

# 三、编写原则
## (一)、DEBIAN/control 文件
简单示例
参考: https://leux.cn/doc/Debian%E5%88%B6%E4%BD%9CDEB%E5%8C%85%E7%9A%84%E6%96%B9%E6%B3%95.html
原文：
```
# DEBIAN/control 文件内的属性信息必须以字母或者数字开头，最简单的模板如下：
Package: python3.10			# 包名，缺少会报错
Version: 3.10.9-1+leux			# 版本，缺少会报错
Architecture: amd64			# 架构，缺少会报错
Maintainer: leux			# 作者，缺少会警告
Description: Python 3.10 Version	# 描述，缺少会警告

# DEBIAN/control 文件内的其他可用属性信息：
Package: 				# 软件包名称
Version: 				# 软件包版本
Section: 				# 软件包所属的部分
Priority: 				# 软件包的优先级
Architecture: 				# 软件包的体系结构
Essential: 				# 是否是必要软件包
Depends: 				# 软件包的依赖关系
Pre-Depends: 				# 软件包的先决依赖
Recommends: 				# 建议安装的软件包
Suggests: 				# 建议但非必需的软件包
Conflicts: 				# 与其他软件包的冲突
Provides: 				# 软件包提供的功能
Replaces: 				# 替代其他软件包
Maintainer: 				# 维护者的信息
Description: 				# 软件包的简短描述
# 软件包的详细描述（可以跨多行）
```

部分字段描述
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

续行必须以空格或制表符开头
参考:
- https://www.debian.org/doc/debian-policy/ch-controlfields.html#syntax-of-control-files （编写control的语法）
- https://www.debian.org/doc/debian-policy/ch-controlfields.html#description
原文:
```
folded
 The value of a folded field is a logical line that may span several lines. The lines after the first are called continuation lines and must start with a space or a tab. Whitespace, including any newlines, is not significant in the field values of folded fields. 3
```
<br />

但，又说不能用tab制表符，因为后果不可预测，好吧有点看不懂了，咱不用制表符就是了。
参考: https://www.debian.org/doc/debian-policy/ch-controlfields.html#description
原文:
```
The lines in the extended description can have these formats:

· Those starting with a single space are part of a paragraph. Successive lines of this form will be word-wrapped when displayed. The leading space will usually be stripped off. The line must contain at least one non-whitespace character.

· Those starting with two or more spaces. These will be displayed verbatim. If the display cannot be panned horizontally, the displaying program will line wrap them “hard” (i.e., without taking account of word breaks). If it can they will be allowed to trail off to the right. None, one or two initial spaces may be deleted, but the number of spaces deleted from each line will be the same (so that you can have indenting work correctly, for example). The line must contain at least one non-whitespace character.

· Those containing a single space followed by a single full stop character. These are rendered as blank lines. This is the only way to get a blank line. 9

· Those containing a space, a full stop and some more characters. These are for future expansion. Do not use them.

Do not use tab characters. Their effect is not predictable.
```

段落之间用 . 来分割
参考: https://www.debian.org/doc/debian-policy/ch-controlfields.html#syntax-of-control-files
原文:
```text
Stanza separators (empty lines), and lines consisting only of U+0020 SPACE and U+0009 TAB, are not allowed within field values or between fields. Empty lines in field values are usually escaped by representing them by a U+0020 SPACE followed by a U+002E ()..
```
<br />

文件必须用UTF-8编码
参考: https://www.debian.org/doc/debian-policy/ch-controlfields.html#syntax-of-control-files
原文:
```text
All control files must be encoded in UTF-8.
```
<br />

注释：用 # 开头且前置没有任何前置空格
参考: https://www.debian.org/doc/debian-policy/ch-controlfields.html#syntax-of-control-files
原文:
```text
Lines starting with U+0023 (), without any preceding whitespace, are comment lines that are only permitted in source package control files (). These comment lines are ignored, even between two continuation lines. They do not end logical lines.#debian/control
```
<br />


详细具体的参考：<br />
https://www.debian.org/doc/debian-policy/ch-controlfields.html#debian-binary-package-control-files-debian-control

## (二)、维护脚本相关的
### (1)、执行先后顺序
我在每个脚本的开头放入类似以下代码，观察执行先后顺序以及 $1 的值是什么，以此验证文档：https://zhuanlan.zhihu.com/p/439622402 中描述执行的先后顺序是否和我实验的一致。
```bash
#!/bin/bash
set -e

echo "-----------"
echo "安装后执行的脚本 -> postinst"
echo "\$1的值: $1"
echo "-----------"
```
set +e

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


这是从同一个包多次安装的角度来看，实际上还要复杂一点, 比如这个明显可以看到在原来的卸载脚本出现错误后，会启用新发布.deb 内的脚本去尝试卸载，具体详细流程还是得看官方的文档。
```bash
┌──(root㉿kali)-[/home/vbgaga/vscode-project/deb_make/project]
└─# dpkg -i yakit.deb
(正在读取数据库 ... 系统当前共安装有 465779 个文件和目录。)
准备解压 yakit.deb  ...
正在解压 yakit (1.4.5-1226) 并覆盖 (1.4.5-1226) ...
dpkg: 警告: 旧的 yakit 软件包 post-removal 脚本 子进程返回错误状态 10
dpkg: 现在尝试使用新软件包所带的脚本...
dpkg: ... 它看起来没有问题
正在设置 yakit (1.4.5-1226) ...
Copy /tmp/yakit_install_package/Yakit-1.4.5-1226-linux-amd64.AppImage to -> /usr/share/yakit
Extract files from AppImage.
squashfs-root/.DirIcon
squashfs-root/AppRun
squashfs-root/LICENSE.electron.txt
squashfs-root/LICENSE.md
squashfs-root/LICENSES.chromium.html
squashfs-root/bins
squashfs-root/bins/database
squashfs-root/bins/database/flag.txt

```

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

参考mysql的
```bash
┌──(root㉿DESKTOP-865OAE3)-[/mnt/c/Users/hajimi/Downloads/mysql_extract_dir/DEBIAN]
└─# cat postrm
#!/bin/bash

# Copyright (c) 2014, 2023, Oracle and/or its affiliates. All rights reserved.
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; version 2 of the License.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301 USA

set -e

if [ -f "/etc/apt/sources.list.d/mysql.list" ];
then
        rm -f /etc/apt/sources.list.d/mysql.list*
fi

if [ -f "/usr/share/keyrings/mysql-apt-config.gpg" ];
then
    rm -f /usr/share/keyrings/mysql-apt-config.gpg
fi

if [ "$1" = purge ] && [ -e /usr/share/debconf/confmodule ]; then
        . /usr/share/debconf/confmodule
        db_purge
fi

set +e
```
nginx的:
```bash

┌──(hajimi㉿DESKTOP-865OAE3)-[~/debian_packages/nginx/DEBIAN]
└─$ cat postinst
#!/bin/sh
set -e

case "$1" in
  abort-upgrade|abort-remove|abort-deconfigure|configure)
    ;;
  triggered)
    if invoke-rc.d --quiet nginx status >/dev/null; then
      echo "Triggering nginx reload ..."
      invoke-rc.d nginx reload || true
    fi
    exit 0
    ;;
  *)
    echo "postinst called with unknown argument \`$1'" >&2
    exit 1
    ;;
esac

if invoke-rc.d --quiet nginx status >/dev/null; then
  invoke-rc.d nginx upgrade || invoke-rc.d nginx restart
  exit $?
else
  if ! invoke-rc.d nginx start; then
    echo "Failed to start NGINX in postinst script, please check the logs" >&2
    exit 0
  fi
fi



exit 0
```

### (4)、defconf 使用相关 - 不要滥用它，并不是所有的需要用到它
#### 1. 介绍
https://wiki.debian.org/debconf<br />
原文翻译：<br />
- 简单来说，debconf 就是“正确安装 Shield Wizards Wizards”，这是基于 Debian 发行版的主要优势之一。
- 当你安装或升级包时，debconf会一次性问所有配置问题，并将答案存储在数据库中。然后当每个包安装自己时，脚本会利用数据库中的偏好设置。这样可以省去手动编辑配置文件的麻烦，也省去了等待每个软件包安装完再回答某些配置问题的麻烦。

> 看着，感觉 defconf 设置在升级的时候挺有用的，设置一个安装路径等配置，再次安装的时候，类似Windows安装的时候，设置安装路径，然后后续升级安装的时候，无需再次配置路径，路径显示的就是软件安装的路径。

#### 2. template 文件的编写（主要用来设置交互的提示信息，以及选项，可以设置支持多语言）
格式参考: https://manpages.debian.org/jessie/debconf-doc/debconf-devel.7.en.html#THE_TEMPLATES_FILE<br />
原文翻译：<br />
- 使用debconf的软件包可能会想问一些问题。这些 问题以模板形式存储在模板文件中。 和配置文件脚本一样，模板文件放在control.tar.gz部分 一个Deb。其格式类似于 Debian 控制文件;一组诗节 以空白行分隔，每节采用类似RFC822的形式

文件路径 DEBIAN/templates<br />
参考自: http://www.fifi.org/doc/debconf-doc/tutorial.html#AEN34<br />
原文:<br />
- Start writing a debian/templates file. Each time you find a piece of output or a question, add it to the file as a new template. The format of this file is simple and quite similar to a Debian control file:

其中type 支持多种类型，title 可以设置交互界面的标题 (没找到示例，就用我自己的好了）<br />
```
Template: yakit/title
Type: title
Description: Yakit
```
<img width="1571" height="845" alt="image" src="https://github.com/user-attachments/assets/ae5f5c06-af7d-42a6-aecd-83b7ef0b5412" />



设置error错误提示消息，可以参考 ufw 的：<br />
```
Template: ufw/existing_configuration
Type: error
Description: Existing configuration found
 An existing configuration for ufw has been found.
 Existing rules must be managed manually.
 .
 You should read the ufw(8) manpage for details about ufw configuration.
Description-cs.UTF-8: Nalezena existující konfigurace
 Pro ufw byla nalezena existující konfigurace. Existující pravidla musí být spravována ručně.
 .
 Pro podrobnosti o konfiguraci ufw si přečtěte manuálovou stránku ufw(8).
Description-da.UTF-8: Eksisterende konfiguration fundet
 Der er fundet en eksisterende konfiguration for ufw. Eksisterende regler skal håndteres manuelt.
 .
 Du bør læse manualsiden ufw(8) for detaljer om ufw-konfiguration.
Description-de.UTF-8: Bestehende Konfiguration gefunden
 Eine existierende Konfiguration fÃ¼r Ufw wurde gefunden. Existierende Regeln mÃ¼ssen manuell verwaltet werden.
 .
 Sie sollten die Handbuchseite ufw(8) fÃ¼r weitere Hinweise zur Konfiguration von Ufw lesen.
Description-es.UTF-8: Se ha encontrado la configuración existente
 Se ha encontrado una configuración de ufw existente. Las reglas existentes se deberán gestionar manualmente.
 .
 Debería leer la página del manual ufw(8) para los detalles sobre la configuración de ufw.
Description-eu.UTF-8: Konfigurazioa aurkituta
 Aurreko ufw konfigurazio bat aurkitu da. Dauden arauak eskuz kudeatu behar dira.
 .
 ufw(8) manual orria irakurri beharko zenuke ufw konfigurazio xehetasunetarako.
Description-fi.UTF-8: Asetustiedosto löytyi
 Järjestelmästä löytyi ufw:n asetustiedosto. Olemassa olevia sääntöjä täytyy pitää yllä käsin.
 .
 Lisätietoja ufw:n asetuksista löytyy man-ohjesivulta ufw(8).
Description-fr.UTF-8: Configuration existante trouvée
 Une configuration existante a été trouvée pour ufw. Les règles qui y sont utilisées doivent être gérées manuellement.
 .
 Vous devriez lire la page de manuel ufw(8) pour plus de détails sur la configuration de ufw.
Description-gl.UTF-8: Achouse unha configuraciÃ³n xa existente
 Atopouse unha configuraciÃ³n de ufw preexistente. As regras xa existentes deben xestionarse manualmente.
 .
 DeberÃ­a ler a pÃ¡xina de manual de ufw(8) para coÃ±ecer mÃ¡is detalles acerca da configuraciÃ³n de ufw.
Description-it.UTF-8: Trovata una configurazione già esistente
 È stata trovata una configurazione per ufw già esistente. Le regole esistenti devono essere gestite manualmente.
 .
 Si veda la pagina man di ufw(8) per i dettagli sulla configurazione di ufw.
Description-ja.UTF-8: 既存の設定が見つかりました
 ufw の既存の設定が見つかりました。既存のルールは手動で管理する必要があります。
 .
 ufw の設定の詳細については、ufw(8) の man ページを読んでください。
Description-nl.UTF-8: Bestaande configuratie gevonden
 Er is een bestaande configuratie voor ufw gevonden. De bestaande regels moeten handmatig beheerd worden.
 .
 Lees de man-pagina van ufw(8) voor details over de configuratie van ufw.
Description-pl.UTF-8: Znaleziono istniejącą konfigurację
 Znaleziono istniejącą konfigurację ufw. Konieczne jest ręczne zarządzanie istniejącymi regułami.
 .
 Proszę zapoznać się ze stroną man ufw(8), aby poznać szczegóły na temat konfiguracji ufw.
Description-pt.UTF-8: Foi encontrada configuração existente
 Foi encontrada uma configuração existente para o ufw. As regras existentes terão que ser geridas manualmente.
 .
 Você deverá ler o manual do ufw(8) para detalhes acerca da configuração do ufw.
Description-pt_BR.UTF-8: Configuração existente encontrada
 Uma configuração existente para o ufw foi encontrada. Regras existentes devem ser gerenciadas manualmente.
 .
 Você deveria ler a página de manual ufw(8) para detalhes sobre a configuração do ufw.
Description-ro.UTF-8: S-a găsit o configurație existentă
 O configurație existentă pentru «ufw» a fost găsită. Regulile existente trebuie gestionate manual.
 .
 Ar trebui să citiți pagina de manual ufw(8) pentru detalii despre configurarea „ufw”.
Description-ru.UTF-8: Найдены предыдущие настройки программы
 Найдены предыдущие настройки ufw. Существующие правила нужно изменять вручную.
 .
 Подробней о настройке ufw можно прочитать в справочной странице ufw(8).
Description-sk.UTF-8: Nájdená existujúca konfigurácia
 Našla sa existujúca konfigurácia ufw. Existujúce pravidlá je potrebné spravovať manuálne.
 .
 Mali by ste si prečítať podrobnosti o konfigurácii v manuálovej stránke ufw(8).
Description-sv.UTF-8: Äldre inställningar funna
 Inställningar för en tidigare version av ufw har hittats. Existerande regler måste hanteras manuellt.
 .
 Du bör läsa manualsidan ufw(8) för detaljerad information om ufws inställningar.
Description-vi.UTF-8: Tìm thấy cấu hình đã có
 Một cấu hình ufw đã tồn tại đã được tìm. Bạn cần tự thao tác những quy tắc đã có.
 .
 Hãy đọc trang hướng dẫn ufw(8) để tìm chi tiết về cấu hình ufw.
```

我自己的 DEBIAN/templates 部分：<br />
```
Template: yakit/path_empty_error
Type: error
Description: The path cannot be empty!!!
Description-zh_CN: 路径不能为空!!!
```
<img width="1571" height="845" alt="image" src="https://github.com/user-attachments/assets/056db662-f54b-4a22-ae41-8eab051c7824" />

我的 DEBIAN/config 部分代码：
```bash
# 验证3：不能是根目录
if [ "$INSTALL_DIR" = "/" ]; then
    db_input critical yakit/path_root_error || true
    db_go || true
    db_fset yakit/install_dir seen false
    continue
fi
```


#### 3. 设置支持多语言(国际化）
参考： http://www.fifi.org/doc/debconf-doc/tutorial.html <br />
AI翻译：https://github.com/CreatNULL/yakit-deb/blob/main/debconf/AI%E7%BF%BB%E8%AF%91-Debconf%20%E7%A8%8B%E5%BA%8F%E5%91%98%E6%95%99%E7%A8%8B-debconf-doc-tutorial.md#%E6%9C%AC%E5%9C%B0%E5%8C%96%E6%A8%A1%E6%9D%BF%E6%96%87%E4%BB%B6《br />

作为开发者，如果熟悉多国语言，可以这样编写（注意，fishf 为项目名称，install_dir 这是算是给模板取个名字，用来表示设置安装路径的）：
```
Template: fishf/install_dir
Type: string
Default: "/usr/share/fishf"
Description: Please enter the installation path
Description-zh_CN: 请输入安装路径
```
这个我没有深究，反正用不到, 下面的描述不一定正确<br />
如果需要翻译者协助，请参相关文档，是需要用到gettext 这个东西，文档内说使用 debconf-getlang , 生成用来翻译的模板，实际当使用 debconf-getlang 会提示：
```bash
# debconf-getlang：此实用程序已弃用；您应该切换到使用po-debconf包
debconf-getlang: This utility is deprecated; you should switch to using the po-debconf package.
```
安装
```bash
apt-get install po-debconf
```
命令
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
先得创建  debian/po/POTFILES.in ,该文件告诉在所有程序源代码中，哪些文件有需要翻译的标记字符串 <br />
参考: https://www.gnu.org/software/gettext/manual/html_node/po_002fPOTFILES_002ein.html <br />

而且看着，似乎还需要创建一个文件 LINGUAS 来指他支持什么语言<br />
参考：https://www.gnu.org/software/gettext/manual/html_node/po_002fLINGUAS.html <br />

还可以看看这个文档: <br />
https://manpages.debian.org/buster/po-debconf/po-debconf.7.en.html <br />


#### 4. config 文件 (用来设置提问的问题）
路径：DEBIAN/config
提问的问题需要在config 文件中，而不是在 postinst 脚本中<br />

参考自: http://www.fifi.org/doc/debconf-doc/tutorial.html#AEN113<br />
原文:<br />
- Next, decide what order the questions should be asked and the messages to the user should be displayed, figure out what tests you'll make before asking the questions and displaying the messages, and start writing a debian/config file to ask and display them.
  - Note: These questions are asked by a separate config script, not by the postinst, so the package can be configured before it is installed, or reconfigured after it is installed. Do not make your postinst use debconf to ask questions.
 
#### 5. 如果使用 debconf ，在control 文件中得指定依赖Depands条件： debconf (>= 0.2.17)
http://www.fifi.org/doc/debconf-doc/tutorial.html#AEN22<br />
原文:
```
First, your package must depend on debconf (or pre-depend on it if it uses debconf in its preinst[1]). This is necessary since debconf isn't essential.

The first thing to do is look at your postinst, plus any program your postinst calls (like a "packageconfig" program), plus your preinst, and even your prerm and postrm. Take note of all output they can generate and all input they prompt the user for. All this output and input must be eliminated for your package to use debconf. (Output to stderr can be left as is.)

Note: If your preinst uses debconf, you must make your package Pre-Depend on debconf (>= 0.2.17).
```

参考mysql的:
```
┌──(root㉿DESKTOP-865OAE3)-[/mnt/c/Users/hajimi/Downloads/mysql_extract_dir/DEBIAN]
└─# cat control
Package: mysql-apt-config
Version: 0.8.36-1
Section: database
Priority: optional
Architecture: all
Pre-depends: debconf (>= 0.2.17), dpkg (>= 1.15), lsb-release, wget, bash (>= 4.0), gnupg
Homepage: http://dev.mysql.com/
Maintainer: MySQL Release Engineering <mysql-build@oss.oracle.com>
Installed-Size: 35
Description: Auto configuration for MySQL APT Repo.
 MySQL is a fast, stable and true multi-user, multi-threaded SQL database
 server. SQL (Structured Query Language) is the most popular database query
 language in the world. The main goals of MySQL are speed, robustness and
 ease of use.
```
vim 没有使用, 就无需指定
```
┌──(vbgaga㉿DESKTOP-865OAE3)-[/mnt/c/Users/hajimi/Downloads/vim_extract_DEBIAN]
└─$ cat control
Package: vim
Version: 2:9.1.1882-1
Architecture: amd64
Maintainer: Debian Vim Maintainers <team+vim@tracker.debian.org>
Installed-Size: 4077
Depends: vim-common (= 2:9.1.1882-1), vim-runtime (= 2:9.1.1882-1), libacl1 (>= 2.2.23), libc6 (>= 2.38), libgpm2 (>= 1.20.7), libselinux1 (>= 3.1~), libsodium23 (>= 1.0.14), libtinfo6 (>= 6)
Suggests: ctags, vim-doc, vim-scripts
Provides: editor
Section: editors
Priority: optional
Homepage: https://www.vim.org/
Description: Vi IMproved - enhanced vi editor
 Vim is an almost compatible version of the UNIX editor Vi.
 .
 Many new features have been added: multi level undo, syntax
 highlighting, command line history, on-line help, filename
 completion, block operations, folding, Unicode support, etc.
 .
 This package contains a version of vim compiled with a rather
 standard set of features.  This package does not provide a GUI
 version of Vim.  See the other vim-* packages if you need more
 (or less).
```

#### 6. 如果使用了 debconf 在卸载指定 purge 也就是 -P 的时候得清理保存的问题模板和问题的答案，咱需要修改 postrm 维护脚本
参考: http://www.fifi.org/doc/debconf-doc/tutorial.html#AEN158 <br />
原文: <br />
```
There is one other alteration you need to make to the postrm script. When your package is purged, it should get rid of all the questions and templates it was using in the database. Accomplishing this is simple; use the "purge" command (make sure you don't fail if debconf has already been removed, though):


if [ "$1" = "purge" -a -e /usr/share/debconf/confmodule ]; then
    # Source debconf library.
    . /usr/share/debconf/confmodule
    # Remove my changes to the db.
    db_purge
fi
```

#### 7. 还有一点值得关注，在编写循环的时候
http://www.fifi.org/doc/debconf-doc/tutorial.html#AEN217 -> Letting the User Back Up

#### 8. 如果你设置的问题不显示，首先确保不是升级安装操作，或者同一个包安装第二次，其次得先看看自己的 debconf/priority 的配置
这里可以看到我展示的级别为：debconf/priority: critical

```
┌──(root㉿kali)-[/home/…/generate_deb/project/fishf/DEBIAN]
└─# debconf-show debconf
  debconf-apt-progress/title:
  debconf-apt-progress/info:
  debconf-apt-progress/preparing:
* debconf/frontend: Dialog
  debconf-apt-progress/media-change:
* debconf/priority: critical
```
所以如果我希望我的问题，会被展示那级别要 >= critical 
db_input critical xxx/xxxx || true


从未设置过的情况下：
```bash
┌──(root㉿DESKTOP-865OAE3)-[/usr/share/doc/openssl]
└─# debconf-show debconf
  debconf-apt-progress/info:
  debconf/priority: high
  debconf/frontend: Dialog
  debconf-apt-progress/media-change:
  debconf-apt-progress/title:
  debconf-apt-progress/preparing:
```

如果希望手动修改，使用命令:`dpkg-reconfigure debconf`, 这会启动一个交互的界面，让您重新配置
<img width="1571" height="845" alt="image" src="https://github.com/user-attachments/assets/a71562f1-3427-463e-b7f1-7d41d8bea28e" />

重置: 
```
echo "RESET debconf/priority" | debconf-communicate
echo "RESET debconf/frontend" | debconf-communicate
```

再看看效果：
```bash
┌──(root㉿kali)-[/home/…/generate_deb/project/fishf/DEBIAN]
└─# debconf-show debconf
  debconf-apt-progress/title:
  debconf/frontend: Dialog
  debconf-apt-progress/media-change:
  debconf-apt-progress/info:
  debconf/priority: high
  debconf-apt-progress/preparing:
```

#### 9. `/usr/share/debconf/confmodule` 模块涉及的相关命令 db_set、db_input...

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

里面 db_input 设置问题优先级：
```
low
 Very trivial items that have defaults that will work in the vast majority of cases; only control freaks see these.
medium
 Normal items that have reasonable defaults.
high
 Items that don't have a reasonable default.
critical
 Items that will probably break the system without user intervention.

翻译:
low：非常琐碎的项目，其默认值在绝大多数情况下都有效；只有控制狂才能看到这些。
medium：具有合理默认值的普通项目。
high：没有合理默认值的项目。
critical：没有用户干预可能会破坏系统的项目。
Debconf 基于问题的优先级、用户是否已看过它以及正在使用的前端来决定是否实际显示该问题。如果问题不显示，debconf 以代码 30 回复。
```

#### 9. /var/cache/debconf/config.dat（存储所的回答） /var/cache/debconf/templates.dat （存储问题的模板定义）
https://stackoverflow.com/questions/10885177/how-to-read-input-while-installing-debian-package-on-debian-systems<br />

#### 10. 一些其他的相关的命令
```
# 查看已保存的配置
debconf-get-selections （apt install debconf-utils)

# 查看特定包的配置
debconf-get-selections | grep openssh-server

# 这个感觉简单点哈哈
debconf-show <package-name>

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
 
