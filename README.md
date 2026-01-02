## 前言
 每次都是 AppImage，没找到 .deb 的安装包，琢磨着给他搞成.deb <br/>
 - 发现 AppImage 特性是可以解压（xxx.AppImage --appimage-extract）就是为了解决在不支持FUSE的系统上使用AppImage </br>
 - 进入解压后的目录（squashfs-root) AppRun，是可以运行的 但是得设置一下环境变量 export APPDIR=解压的目录
 - Yakit-xxx-xxx.AppImage 都是自动从官网（https://www.yaklang.com/）下载的, 我只是大自然的搬运工o(=•ェ•=)m

## 原理：
主要涉及 项目/DEBIAN/* 下面涉及的几个重要的脚本就是 control、preinst、postinst、prerm、postrm  ，control 是一些包的信息，preinst、postinst、prerm、postrm 四个是维护脚本，项目/DEBIAN 的同级目录下的其他文件夹，就映射Linux真实的路径

<img width="230" height="366" alt="image" src="https://github.com/user-attachments/assets/9956ae3e-beeb-4c44-b62f-6032e07687b6" />

编写主要涉及：preinst、postinst、prerm、postrm 脚本，以及他们执行的先后顺序
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


#### 安装逻辑：
  1. 检测程序是否已经在运行
  2. 先从官网获取最新的版本信息
  3. 创建安装目录 /usr/share/yakit/, 然后下载输出到目录，赋予执行权限
  4. 把 AppImage 解压，修改权限让其他用户可以访问，进入解压后的目录，修改一些目录权限和chrome-sandbox权限
  5. 创建启动脚本 /usr/bin/yakit，修改权限 ,
  6. 添加 .desktop 文件，更新缓存

#### 卸载
  卸载前判断一下进程是否在运行
  AppImage 

依据这个执行顺序，检测进程是否在运行，只需要在 prerm 脚本中编写，因为如果没有安装，就不存在进程在运行的情况，如果已经安装，再次执行，会被判定位更新，则第一个调用的就是 prerm 脚本
而 preinst 我就用来检查一些必要的依赖。


## 部分内容展示
### 请求获取版本信息
<img width="1577" height="878" alt="image" src="https://github.com/user-attachments/assets/bf101232-5ba7-4918-bb34-d71ec224aca6" />

### 请求下载
<img width="935" height="367" alt="image" src="https://github.com/user-attachments/assets/7e5b51fe-2b1a-4448-a2aa-2c42c1f1675a" />

### 解压
<img width="1577" height="878" alt="image" src="https://github.com/user-attachments/assets/060b822a-100d-4640-997a-cd74608b34e9" />

### 创建启动脚本
<img width="1134" height="504" alt="image" src="https://github.com/user-attachments/assets/846b3186-6094-47b3-a7d9-4d4f1d57986d" />

### 创建图标 .desktop 
<img width="1482" height="729" alt="image" src="https://github.com/user-attachments/assets/3376efd2-3222-400b-842c-57cdf1ab74ce" />

## 效果展示
### 安装：
<img width="931" height="618" alt="image" src="https://github.com/user-attachments/assets/4ebded96-67c9-4a0f-9c21-dd4be08ee2f0" />

<img width="890" height="638" alt="image" src="https://github.com/user-attachments/assets/80327a8e-b936-427e-a3ce-a84985416880" />

<img width="601" height="703" alt="image" src="https://github.com/user-attachments/assets/6bd31a93-8b2b-4872-96b2-8183c326c643" />

### 卸载：
程序正在运行，提示是否结束
<img width="1164" height="781" alt="image" src="https://github.com/user-attachments/assets/9cb8c50b-d571-4ce7-8cf9-26bd3abf4b09" />


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
 - 
## 其他
### 编写原则
#### (1)、DEBIAN/control 文件
- https://www.debian.org/doc/debian-policy/ch-controlfields.html
- 续行必须以空格或制表符开头
- 空行看着得用 .

#### (2)、维护脚本幂等性
https://www.debian.org/doc/debian-policy/ch-maintainerscripts.html -> 6.2.Maintainer scripts idempotency
✅ 成功后再运行：保持现状，不报错
✅ 失败后重新运行：继续完成剩余工作
❌ 不能假设：这是第一次运行或环境是干净的
❌ 不能重复：已经完成的操作

### (3)、必须设计为能在无终端环境下工作、必须支持非交互式回退
https://www.debian.org/doc/debian-policy/ch-maintainerscripts.html -> 6.3 Controlling terminal for maintainer scripts
丢给AI，让他理解一下：

您提供的这段文本是 Debian Policy Manual 6.3 节，关于维护脚本的控制终端要求。这非常重要，因为它明确了维护脚本与用户交互的限制条件。
核心规定解析
**1. 基本限制**
原文："Maintainer scripts are not guaranteed to run with a controlling terminal and may not be able to interact with the user."
含义：
维护脚本不能保证有控制终端可用
可能无法与用户进行交互
必须设计为能在无终端环境下工作
**2. 必须支持非交互式回退**
原文："They must be able to fall back to noninteractive behavior if no controlling terminal is available."
要求：
脚本必须检测终端可用性
无终端时必须自动切换到非交互模式
不能因为无法交互而卡住或失败
**3. 使用 Debconf 的例外**
原文："Maintainer scripts that prompt via a program conforming to the Debian Configuration Management Specification... may assume that program will handle falling back to noninteractive behavior."
含义：
如果使用 Debconf​ 进行提示，可以依赖它处理非交互情况
Debconf 会自动根据环境选择适当的行为
**4. 高优先级提示的特殊处理**
原文："For high-priority prompts without a reasonable default answer, maintainer scripts may abort if there is no controlling terminal. However, this situation should be avoided..."
限制：
只有没有合理默认值的高优先级提示才允许在无终端时中止
但这种情况应该尽量避免
用户通常认为这是包的bug

那我就用 debconf

#### (4)、尽量减少需要提示的次数
> https://www.debian.org/doc/debian-policy/ch-binary.html#s-maintscriptprompt -> 3.9.1
> 包应尽量减少需要提示的次数， 并且他们应确保用户**只会被问到每一个 问一次。升级时不应再问同样的问题**， 除非用户已经移除了包的 配置。配置问题的答案应被存储 放置在合适的位置，方便用户修改它们， 以及这些做法都应有记录

> https://wiki.debian.org/debconf
> 简单来说，debconf 就是“正确安装 Shield Wizards Wizards”，这是基于 Debian 发行版的主要优势之一。
> 当你安装或升级包时，debconf会一次性问所有配置问题，并将答案存储在数据库中。然后当每个包安装自己时，脚本会利用数据库中的偏好设置。这样可以省去手动编辑配置文件的麻烦，也省去了等待每个软件包安装完再回答某些配置问题的麻烦。

- http://www.fifi.org/doc/debconf-doc/tutorial.html
> 看不懂，丢给ai


看着，感觉 defconf 
- 在升级的时候的作用类似Windows安装的时候，设置安装路径，然后后续升级安装的时候，无需再次配置路径，路径显示的就是软件安装的路径

#### (5)、脚本应该保持安静，避免不必要的输出
https://www.debian.org/doc/debian-policy/ch-binary.html#s-maintscriptprompt -> 3.9.Maintainer Scripts
