## 前言
 每次都是 AppImage，没找到 .deb 的安装包，琢磨着给他搞成.deb <br/>
 - 发现 AppImage 特性是可以解压（xxx.AppImage --appimage-extract）就是为了解决在不支持FUSE的系统上使用AppImage </br>
 - 进入解压后的目录（squashfs-root) AppRun，是可以运行的 但是得设置一下环境变量 export APPDIR=解压的目录
 - Yakit-xxx-xxx.AppImage 都是自动从官网（https://www.yaklang.com/）下载的, 我只是大自然的搬运工o(=•ェ•=)m

## 原理：
- 安装： 先从官网获取最新的版本，创建安装目录 /usr/share/yakit/, 然后下载输出到目录，赋予执行权限，把 AppImage 解压，修改权限让其他用户可以访问，进入解压后的目录，修改一些目录权限和chrome-sandbox权限，创建启动脚本 /usr/bin/yakit，修改权限 , 添加 .desktop 文件。

编写参考文档：
 - https://leux.cn/doc/Debian%E5%88%B6%E4%BD%9CDEB%E5%8C%85%E7%9A%84%E6%96%B9%E6%B3%95.html  （deb 制作的方法）
 - https://blog.csdn.net/weixin_42267862/article/details/138808742 （deb包中preinst、postinst、prerm、postrm等脚本的执行顺序及参数）
 - https://www.debian.org/doc/manuals/packaging-tutorial/packaging-tutorial.zh_CN.pdf （官方文档）
 - https://www.cnblogs.com/swtjavaspace/p/18188551 （.desktop 的StartupWMClass 值的获取）
 - https://geek-blogs.com/blog/linux-run-appimage/ （AppImage的解压）
 
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

## 效果
### 安装：
<img width="931" height="618" alt="image" src="https://github.com/user-attachments/assets/4ebded96-67c9-4a0f-9c21-dd4be08ee2f0" />

<img width="890" height="638" alt="image" src="https://github.com/user-attachments/assets/80327a8e-b936-427e-a3ce-a84985416880" />

<img width="601" height="703" alt="image" src="https://github.com/user-attachments/assets/6bd31a93-8b2b-4872-96b2-8183c326c643" />

### 卸载：
程序正在运行，提示是否结束
<img width="1164" height="781" alt="image" src="https://github.com/user-attachments/assets/9cb8c50b-d571-4ce7-8cf9-26bd3abf4b09" />

