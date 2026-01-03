## 前言
 每次都是 AppImage，没找到 .deb 的安装包，琢磨着给他搞成.deb <br/>
 - 发现 AppImage 特性是可以解压（xxx.AppImage --appimage-extract）就是为了解决在不支持FUSE的系统上使用AppImage </br>
 - 进入解压后的目录（squashfs-root) AppRun，是可以运行的 但是得设置一下环境变量 export APPDIR=解压的目录
 - Yakit-xxx-xxx.AppImage 都是自动从官网（https://www.yaklang.com/）下载的, 我只是大自然的搬运工o(=•ェ•=)m

## 原理：
主要涉及 项目/DEBIAN/* 下面涉及的几个重要的脚本就是 control、preinst、postinst、prerm、postrm  ，control 是一些包的信息，preinst、postinst、prerm、postrm 四个是维护脚本，项目/DEBIAN 的同级目录下的其他文件夹，就映射Linux真实的路径

<img width="230" height="366" alt="image" src="https://github.com/user-attachments/assets/9956ae3e-beeb-4c44-b62f-6032e07687b6" />

编写主要涉及：

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

# 二、编写原则
## (一)、DEBIAN/control 文件
- https://www.debian.org/doc/debian-policy/ch-controlfields.html#syntax-of-control-files （编写control的语法）
- 续行必须以空格或制表符开头
```
Description: This is a single logical line that happens
 to span multiple physical lines. The newlines are
 converted to spaces.
```
- 段落之间的空行用 . 来替代
- 文件必须用UTF-8编码
<br />

具体的字段参考：<br />
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
#### 1. 尽量减少需要提示的次数， 并且他们应确保用户只会被问到每一个 问一次。
https://www.debian.org/doc/debian-policy/ch-binary.html#prompting-in-maintainer-scripts -> 3.9.1. Prompting in maintainer scripts<br />

#### 2. 必须设计为能在无终端环境下工作、必须支持非交互式回退
https://www.debian.org/doc/debian-policy/ch-maintainerscripts.html -> 6.3 Controlling terminal for maintainer scripts<br />
原文：
- Maintainer scripts are not guaranteed to run with a controlling terminal and may not be able to interact with the user. They must be able to fall back to noninteractive behavior if no controlling terminal is available. Maintainer scripts that prompt via a program conforming to the Debian Configuration Management Specification (see Prompting in maintainer scripts) may assume that program will handle falling back to noninteractive behavior.

- For high-priority prompts without a reasonable default answer, maintainer scripts may abort if there is no controlling terminal. However, this situation should be avoided if at all possible, since it prevents automated or unattended installs. In most cases, users will consider this to be a bug in the package.

#### 3. 用 set -e 
https://www.debian.org/doc/debian-policy/ch-maintainerscripts.html -> 6.1. Introduction to package maintainer scripts<br />
原文:
- The package management system looks at the exit status from these scripts. It is important that they exit with a non-zero status if there is an error, so that the package management system can stop its processing. For shell scripts this means that you almost always need to use (this is usually true when writing shell scripts, in fact). It is also important, of course, that they exit with a zero status if everything went well.set -e

### (4)、脚本应该保持安静，避免不必要的输出
https://www.debian.org/doc/debian-policy/ch-binary.html#s-maintscriptprompt -> 3.9.Maintainer Scripts<br />
原文:
- The package installation scripts should avoid producing output which is unnecessary for the user to see and should rely on to stave off boredom on the part of a user installing many packages. This means, amongst other things, not passing the option to .dpkg--verboseupdate-alternatives

### (7)、defconf 使用
#### 1. 介绍
- https://wiki.debian.org/debconf
> 简单来说，debconf 就是“正确安装 Shield Wizards Wizards”，这是基于 Debian 发行版的主要优势之一。
> 当你安装或升级包时，debconf会一次性问所有配置问题，并将答案存储在数据库中。然后当每个包安装自己时，脚本会利用数据库中的偏好设置。这样可以省去手动编辑配置文件的麻烦，也省去了等待每个软件包安装完再回答某些配置问题的麻烦。

> 看着，感觉 defconf 设置在升级的时候挺有用的，设置一个安装路径等配置，再次安装的时候，类似Windows安装的时候，设置安装路径，然后后续升级安装的时候，无需再次配置路径，路径显示的就是软件安装的路径。

#### 2. template 文件的编写，主要设置交互的提示信息，以及选项，可以设置支持多语言
格式参考: https://manpages.debian.org/jessie/debconf-doc/debconf-devel.7.en.html#THE_TEMPLATES_FILE
> 使用debconf的软件包可能会想问一些问题。这些 问题以模板形式存储在模板文件中。 和配置文件脚本一样，模板文件放在control.tar.gz部分 一个Deb。其格式类似于 Debian 控制文件;一组诗节 以空白行分隔，每节采用类似RFC822的形式

#### 3. 看看，有啥命令 `/usr/share/debconf/confmodule`, 看到 db_set、db_input 等等
对于这些命令的解释：
- https://manpages.debian.org/jessie/debconf-doc/debconf-devel.7.en.html#THE_DEBCONF_PROTOCOL 文档中有详细描述 

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

<img width="912" height="224" alt="image" src="https://github.com/user-attachments/assets/ee4a2aa2-5974-4493-a842-bf754b9d3e97" />


#### 4. /var/cache/debconf/config.dat（存储所的回答） /var/cache/debconf/templates.dat （存储问题的模板定义）
https://stackoverflow.com/questions/10885177/how-to-read-input-while-installing-debian-package-on-debian-systems<br />

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




## 编写参考文档
 - https://leux.cn/doc/Debian%E5%88%B6%E4%BD%9CDEB%E5%8C%85%E7%9A%84%E6%96%B9%E6%B3%95.html  （deb 制作的方法）
 - https://blog.csdn.net/weixin_42267862/article/details/138808742 （deb包中preinst、postinst、prerm、postrm等脚本的执行顺序及参数）
 - https://www.debian.org/doc/debian-policy/ch-controlfields.html （DEBIAN/control)
 - https://wiki.debian.org/MaintainerScripts (重复安装，正常安装，安装失败 执行流程图）
 - https://www.debian.org/doc/manuals/packaging-tutorial/packaging-tutorial.zh_CN.pdf （官方文档）
 - https://www.cnblogs.com/swtjavaspace/p/18188551 （.desktop 的StartupWMClass 值的获取）
 - https://geek-blogs.com/blog/linux-run-appimage/ （AppImage的解压）
 - https://www.oryoy.com/news/ubuntu-debconf-quan-gong-lve-qing-song-jie-jue-xi-tong-pei-zhi-nan-ti.html (debconf)
 - https://wiki.debian.org/debconf (debconf 介绍)
 - https://www.oryoy.com/news/ubuntu-xin-shou-bi-kan-qing-song-zhang-wo-debconf-pei-zhi-ji-qiao-gao-bie-xi-tong-she-zhi-nan-ti.html (defconf 配置）
 - http://www.fifi.org/doc/debconf-doc/tutorial.html ( defconf 配置教程）
 - https://linux.extremeoverclocking.com/man/3/confmodule ( defconf模块的 基本的简介）
 - https://www.tecmint.com/dpkg-reconfigure-installed-package-in-ubuntu-debian/ （dpkg-reconfigure 命令）
 - https://manpages.debian.org/testing/debconf/debconf-communicate.1.en.html （debconf-communicate  命令）
