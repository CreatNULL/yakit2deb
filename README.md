## 前言

Yakit-xxx-xxx.AppImage 自己从官网（https://www.yaklang.com/）下载，然后放入 yakit_install_package 路径
修改
- DEBIAN/control 文件中的版本信息
- DEBIAN/postinst 中的 YAKIT_VERSION
- 打包 `dpkg-deb --root-owner-group --build yakit`

安装
- dpkg -i yakit.deb

编写的时候，尽可能的规范了，但是可能还是有不规范地方，技术有限。

<img width="469" height="406" alt="image" src="https://github.com/user-attachments/assets/5cdc98ec-4eca-4c26-a057-613013cd732f" />

## 目录结构：
```bash
┌──(vbgaga㉿kali)-[~/vscode-project/deb_make/project]
└─$ tree ./yakit/
./yakit/
├── DEBIAN
│   ├── config  # Debian 软件包中处理配置的 shell 脚本文件，与 DEBIAN/templates​ 配合工作，我用来询问 存放 AppImage 解压后的目录
│   ├── control  # 一些包的基本信息
│   ├── postinst  # 安装脚本
│   ├── postrm   # 卸载脚本
│   └── templates # 于定义软件包安装和配置过程中的交互式对话框
├── tmp
│   └── yakit_install_package # 存放从官网下载的yakit ，如果后期版本更新，可以自己替换后重新打包
│       └── Yakit-1.4.5-1226-linux-amd64.AppImage  
└── usr
    └── share
        └── icons
            └── hicolor
                ├── 128x128
                │   └── apps
                │       └── yakit.png
                ├── 16x16
                │   └── apps
                │       └── yakit.png
                ├── 256x256
                ├── 32x32
                │   └── apps
                │       └── yakit.png
                ├── 48x48
                │   └── apps
                │       └── yakit.png
                ├── 512x512
                │   └── apps
                │       └── yakit.png
                ├── 64x64
                │   └── apps
                │       └── yakit.png
                └── apps
                    └── yakit.png

22 directories, 13 files
```

## 开发
- 发现 AppImage 特性是可以解压（xxx.AppImage --appimage-extract）就是为了解决在不支持FUSE的系统上使用AppImage
- 进入解压后的目录（squashfs-root) 
  - AppRun，是可以运行的 但是得设置一下环境变量 export APPDIR=解压的目录
  - chrome-sandbox 和 其他一些目录设置权限 755

## 安装逻辑 （dpkg -i xxx.deb)
- 检测程序是否已经在运行
- 指定**解压目录**，设置默认 /usr/share/yakit/ （请**不要指定系统已经存在的路径**，例如 /usr/share、/root、而是在其后添加新的目录，例如 /tools/yakit, 会自动创建)
- 解压
  - 把 AppImage 解压，修改权限，让其他用户可以访问，进入解压后的目录，修改一些目录权限和chrome-sandbox权限
- 创建启动脚本 /usr/bin/yakit，修改权限
- 添加 .desktop 文件，更新缓存

## 卸载 (dpkg -r xxxx 和 dpkg -P xxxx)
- 卸载前脚本判断一下进程是否在运行
- 删除 /user/bin/yakit 启动脚本、图标文件、.desktop 文件
- 删除所有 $HOME/.confg/yakit 和 $HOME/yakit-projects
- 完全卸载(dpkg -P xxx) ，执行一下清理 debconf，就把 debconf 之前配置的安装目录的配置也删掉，否则下次安装，他不会询问你解压的目录是什么。


## 更新 （dpkg -i xxx.deb)
- 不考虑使用 DEBAIN/watch 文件，因为我就压根没有上游支持，自己手动放进去，重新打包，然后安装一下，
- 如果觉的麻烦，可以修改 postinst 脚本，自动从官网先获取版本信息（官网有一个 version.txt 我记得是），然后拼接下载。
- 但是这样就不符合规范了（咱自己用无所谓了哈哈）
- dpkg -i 再次支持，也就是升级的时候
  - 也会检测是否运行，
  - 升级会使用首次安装的目录（安装后，想修改路径也行，但是请你确保清楚自己在做什么，因为**他会删除旧的目录，当然会弹窗确认是否删除，默认不删除旧的目录**）

## 修改解压目录
- 检测是否运行
- 弹窗指定新的目录
- 移动到新的目录
- 弹窗确认，删除旧的空的目录（防止误删）

## 效果展示
### （1）安装

<img width="1571" height="845" alt="image" src="https://github.com/user-attachments/assets/21008370-ff68-4f30-a81a-00dfbdb56e18" />

<img width="1571" height="845" alt="image" src="https://github.com/user-attachments/assets/7ad6a859-5078-45c1-a5c4-f887588e594b" />

<img width="1571" height="845" alt="image" src="https://github.com/user-attachments/assets/e06bd488-b540-418c-9584-2ae4b96e7d88" />

### （2）卸载
完全卸载，就下次的安装路径又得重新指定<br />
```bash
dpkg -P yakit
```
<img width="714" height="157" alt="image" src="https://github.com/user-attachments/assets/30c49527-bf89-47e7-a714-6784119aaab0" />

普通的<br />
```bash
dpkg -r yakit 或 apt-get remove yakit
```
<img width="723" height="129" alt="image" src="https://github.com/user-attachments/assets/2a817d99-9665-469a-8ae7-7766f05561ce" />

两种卸载都会删除 $HOME/yakit-projects 和 $/.config/yakit 两个目录，唯一区别就是下次安装是否还需要指定解压目录

### (3) 修改解压目录
```bash
dpkg-reconfigure yakit
```
<img width="891" height="103" alt="image" src="https://github.com/user-attachments/assets/3c11e459-ee7e-458a-85d5-07b66090b78a" />

