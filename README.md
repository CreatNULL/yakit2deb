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
 - https://wiki.debian.org/MaintainerScripts (重复安装，正常安装，安装失败 执行流程图）
 - https://www.debian.org/doc/manuals/packaging-tutorial/packaging-tutorial.zh_CN.pdf （官方文档）
 - https://www.cnblogs.com/swtjavaspace/p/18188551 （.desktop 的StartupWMClass 值的获取）
 - https://geek-blogs.com/blog/linux-run-appimage/ （AppImage的解压）
 - https://www.oryoy.com/news/ubuntu-debconf-quan-gong-lve-qing-song-jie-jue-xi-tong-pei-zhi-nan-ti.html (debconf)


## 其他
### 编写原则
#### (1)、维护脚本幂等性
https://www.debian.org/doc/debian-policy/ch-maintainerscripts.html -> 6.2.Maintainer scripts idempotency
✅ 成功后再运行：保持现状，不报错
✅ 失败后重新运行：继续完成剩余工作
❌ 不能假设：这是第一次运行或环境是干净的
❌ 不能重复：已经完成的操作
<img width="925" height="246" alt="image" src="https://github.com/user-attachments/assets/54d55eaf-884a-48d8-9e22-e3a444294d32" />


#### (2)、脚本应该保持安静，避免不必要的输出
https://www.debian.org/doc/debian-policy/ch-binary.html#s-maintscriptprompt -> 3.9.Maintainer Scripts

#### (3)、允许交互，但有严格条件
https://www.debian.org/doc/debian-policy/ch-binary.html#s-maintscriptprompt -> 3.9.1
包应尽量减少需要提示的次数， 并且他们应确保用户只会被问到每一个 问一次。升级时不应再问同样的问题， 除非用户已经移除了包的 配置。配置问题的答案应被存储 放置在合适的位置，方便用户修改它们， 以及这些做法都应有记录


