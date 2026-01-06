## 前言
我想要图标启动，本来我想着自己创建一个.desktop, 结果kali有个脚本，会自动检测是否为安装的软件，如果不是，就会清理掉你的.desktop。
好吧，那我干脆自己搞成.deb 安装一下好了。

Yakit-xxx-xxx.AppImage 自己从官网（https://www.yaklang.com/）下载，然后放入 yakit_install_package 路径

修改
- DEBIAN/control 文件中的版本信息
- DEBIAN/postinst 中的 YAKIT_VERSION
- 打包：进入project目录，执行 `sudo dpkg-deb --root-owner-group --build yakit`

如果想要自己打包其他的，例如破解版的burpsuite可以看看https://github.com/CreatNULL/yakit2deb/blob/main/doc/README.md ,反正逻辑差不多

确保以下权限（755），当然如果不修改，反正打包的时候会报错提示的，问题不大，到时候再修改就好了：
```
/home/createnull/vscode-project/deb_make/project/yakit/DEBIAN

-rwxr-xr-x 1 createnull createnull 4499  1月 6日 16:36 config
-rw-r--r-- 1 createnull createnull  347  1月 5日 01:48 control
-rwxr-xr-x 1 createnull createnull 4758  1月 6日 20:28 postinst
-rwxr-xr-x 1 createnull createnull 3832  1月 6日 20:59 postrm
-rw-r--r-- 1 createnull createnull 1101  1月 5日 23:21 templates
```

安装
- dpkg -i yakit.deb

编写的时候，尽可能的规范了，但是可能还是有不规范地方，技术有限。

<img width="469" height="406" alt="image" src="https://github.com/user-attachments/assets/5cdc98ec-4eca-4c26-a057-613013cd732f" />

## 目录结构：
```
yakit
├── DEBIAN
│   ├── config # Debian 软件包中处理配置的 shell 脚本文件，与 DEBIAN/templates​ 配合工作，我用来询问 存放 AppImage 解压后的目录, 以及确保配置、安装 时候未运行yakit
│   ├── control # 一些包的基本信息
│   ├── postinst  # 安装后执行的脚本
│   ├── postrm # 卸载后执行的脚本
│   └── templates # 于定义软件包安装和配置过程中的交互式对话框
├── tmp
│   └── yakit_install_package # 存放从官网下载的yakit ，如果后期版本更新，可以自己替换后重新打包
│       └── Yakit-1.4.5-1226-linux-amd64.AppImage
└── usr
    └── share
        ├── doc  # 从 yakit的GitHub项目那拷贝下来的
        │   ├── LICENSE.md
        │   ├── README-EN.md
        │   ├── README_LEGACY.md
        │   └── README.md
        └── icons # 从 .AppIamge 中提取的
            └── hicolor
                ├── 128x128
                │   └── apps
                │       └── yakit.png
                ├── 16x16
                │   └── apps
                │       └── yakit.png
                ├── 256x256
                ├── 32x32
                │   └── apps
                │       └── yakit.png
                ├── 48x48
                │   └── apps
                │       └── yakit.png
                ├── 512x512
                │   └── apps
                │       └── yakit.png
                ├── 64x64
                │   └── apps
                │       └── yakit.png
                └── apps
                    └── yakit.png
```

## 开发
- 发现 AppImage 特性是可以解压（xxx.AppImage --appimage-extract）就是为了解决在不支持FUSE的系统上使用AppImage
- 进入解压后的目录（squashfs-root) 
  - AppRun，是可以运行的 但是得设置一下环境变量 export APPDIR=解压的目录
  - chrome-sandbox 和 其他一些目录设置权限 755
- 确保解压后的目录其他用户，是有权限访问的

## 安装逻辑 （dpkg -i xxx.deb)
- 检测程序是否已经在运行 （在 config 中实现了）
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
> 不考虑使用 DEBAIN/watch 文件，因为我就压根没有上游支持，自己手动放进去，重新打包，然后安装一下，如果觉的麻烦，可以修改 postinst 脚本，自动从官网先获取版本信息（官网有一个 version.txt 我记得是），然后拼接下载。但是这样就不符合规范了（咱自己用无所谓了哈哈）
<br />

- dpkg -i .deb 被认定为升级操作
  - 也会检测是否运行 （依赖卸载后的脚本实现）
  - 升级会使用首次安装的目录 （安装后，想修改路径也行，但是请你确保清楚自己在做什么，因为**他会删除旧的目录，当然会弹窗确认是否删除，非交互模式下，不删除旧的目录**）

## 修改解压目录
- 检测是否运行 （config确保）
- 弹窗指定新的目录
    - 移动到新的目录
    - 弹窗确认，删除旧的空的目录（防止误删）

## 效果展示
### （1）首次安装
#### 1. 指定AppImage解压路径
<img width="1668" height="845" alt="image" src="https://github.com/user-attachments/assets/6adfa179-5064-4563-aacd-86cc4c80b7f5" />
#### 2. 图标搜索展示
<img width="758" height="764" alt="image" src="https://github.com/user-attachments/assets/5212d009-fb1c-4403-8538-5e0a7a7ee4de" />

### （2）卸载
#### 1. 完全卸载，就下次的安装路径又得重新指定
```bash
dpkg -P yakit
```
检测到运行<br />
<img width="1668" height="845" alt="image" src="https://github.com/user-attachments/assets/ac933d7f-7f98-4185-93eb-260fc8f26ada" />
否<br />
<img width="658" height="205" alt="image" src="https://github.com/user-attachments/assets/1f757ae7-5b8d-4b06-99d3-3374b96240fe" />
是<br />
<img width="655" height="115" alt="image" src="https://github.com/user-attachments/assets/dafb6d88-41bb-481e-a420-dcf9dc8c673a" />

#### 2. 普通的
```bash
dpkg -r yakit 或 apt-get remove yakit
```
<img width="575" height="112" alt="image" src="https://github.com/user-attachments/assets/573b683f-29ad-4bff-8211-39c6a77e8698" />
当然也会检测是否在运行<br />
<img width="1668" height="845" alt="image" src="https://github.com/user-attachments/assets/41895ca3-22ce-4238-94bc-17286fb2277c" />
#### 3. 我自己对于这两种卸载的处理的区别：
两种卸载都会删除 $HOME/yakit-projects 和 $/.config/yakit 两个目录，唯一区别就是下次安装是否还需要指定解压目录

### (3) 修改解压目录
```bash
dpkg-reconfigure yakit
```
Yakit运行的检测<br />
<img width="1668" height="845" alt="image" src="https://github.com/user-attachments/assets/c0d7e1aa-0f00-488b-8993-bc056b19e9dc" />
指定新的目录<br />
<img width="1668" height="845" alt="image" src="https://github.com/user-attachments/assets/cf1375de-dcbe-427d-a4fb-38f88f724302" />
移动成功，原理就是如果不存在新的目录，则创建新的目录，然后移动目录<br />
<img width="751" height="224" alt="image" src="https://github.com/user-attachments/assets/820d2c66-f1d2-4e6d-8c70-aa6c41773af1" />
执行脚本同步更新<br />
<img width="796" height="222" alt="image" src="https://github.com/user-attachments/assets/d9b664e2-ea1e-4c65-88da-e42566c0f304" />



