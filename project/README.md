## 前言
发现 AppImage 特性是可以解压（xxx.AppImage --appimage-extract）就是为了解决在不支持FUSE的系统上使用AppImage
进入解压后的目录（squashfs-root) AppRun，是可以运行的 但是得设置一下环境变量 export APPDIR=解压的目录
Yakit-xxx-xxx.AppImage 都是自动从官网（https://www.yaklang.com/）下载的, 我只是大自然的搬运工o(=•ェ•=)m

## 安装逻辑：
- 升级安装的话，检测程序是否已经在运行
- 安装前脚本检测 命令 wget 命令是否以及安装下载（用来下载yakit)
- 创建提示指定解压目录，设置默认 /usr/share/yakit/, 然后下载输出到目录，赋予执行权限
- 把 AppImage 解压，修改权限让其他用户可以访问，进入解压后的目录，修改一些目录权限和chrome-sandbox权限
- 创建启动脚本 /usr/bin/yakit，修改权限 ,
- 添加 .desktop 文件，更新缓存

## 卸载
- 卸载前脚本判断一下进程是否在运行

依据这个执行顺序，检测进程是否在运行，只需要在 prerm 脚本中编写。<br />
因为如果没有安装，就不存在进程在运行的情况<br />
如果已经安装，再次执行，会被判定位更新，则第一个调用的就是prerm 脚本<br />
对于 preinst 我就用来检查一些必要的依赖。<br />

最后在卸载后的脚本中，执行一下清理 debconf


## 更新
不考虑使用 DEBAIN/watch 文件，因为我就压根没有上游。

