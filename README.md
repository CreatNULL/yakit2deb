## 前言
我想要图标启动，本来我想着自己创建一个.desktop, 结果kali有个脚本，会自动检测是否为安装的软件，如果不是，就会清理掉你的.desktop。
好吧，那我干脆自己搞成.deb 安装一下好了。

## 我打包好的
https://github.com/CreatNULL/yakit2deb/releases/tag/000000
<br />

## 手动
Yakit-xxx-xxx.AppImage 自己从官网（https://www.yaklang.com/）下载，然后放入 项目：yakit2deb/project/yakit/tmp/yakit_install_package 路径(自己手动创建）

修改
- DEBIAN/control 文件中的版本信息
- DEBIAN/postinst 中的 YAKIT_VERSION
- 打包：进入project目录，执行 `sudo dpkg-deb --root-owner-group --build yakit`

如果想要自己打包其他的，例如尝试制作破解版的burpsuite.deb，可以看看https://github.com/CreatNULL/yakit2deb/blob/main/doc/README.md ,反正逻辑差不多吧感觉。


切换到 root ，确保以下权限（755），当然如果不修改，反正打包的时候会报错提示的，问题不大，到时候再修改就好了：
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
<span style="color: red">**两种卸载都会删除 $HOME/yakit-projects 和 $HOME/.config/yakit 两个目录，唯一区别就是下次安装是否还需要指定解压目录**</span>

这是我对$HOME/yakit-projects的观察
```
# 项目
# ~/ykait-project/default-yakit.db # 默认项目的数据库 (存储了一些插件信息、字典信息等)
# yakit 文件字典存储位置
# ～/yakit-project/payloads
# ~/yakit-project/project/* # 其他的自己创建的项目


# 这些应该和插件也有点关系的
# ~/yakit-project/yakit-profile-plugin.db
# ~/yakit-project/yakit-profile-plugin.db-shm
# ~/yakit-project/yakit-profile-plugin.sb-wal

# yaklang 脚本默认保存目录
# ～/yakit-project/code

# yakit 引擎
# ~/yakit-project/yak-engine/* 
```

### (3) 修改解压目录
```bash
dpkg-reconfigure yakit
```
Yakit运行的检测<br />
<img width="1668" height="845" alt="image" src="https://github.com/user-attachments/assets/c0d7e1aa-0f00-488b-8993-bc056b19e9dc" />

指定新的目录<br />
<img width="1668" height="845" alt="image" src="https://github.com/user-attachments/assets/cf1375de-dcbe-427d-a4fb-38f88f724302" />

移动成功，原理就是如果不存在新的目录，则创建新的目录，然后移动目录，然后删除原本的目录，是如果是默认目录 /usr/share/yakit 不会提示，如果是，防止用户指定了系统等其他重要的目录，所以让用户确认一下<br />
<img width="751" height="224" alt="image" src="https://github.com/user-attachments/assets/820d2c66-f1d2-4e6d-8c70-aa6c41773af1" />

执行脚本同步更新<br />
<img width="796" height="222" alt="image" src="https://github.com/user-attachments/assets/d9b664e2-ea1e-4c65-88da-e42566c0f304" />

### 启动后修复网卡权限
<img width="1397" height="642" alt="image" src="https://github.com/user-attachments/assets/5e3cf0dd-4c8b-478b-aedb-7227eaa1d058" />

退出后，重启yakit
<img width="906" height="652" alt="image" src="https://github.com/user-attachments/assets/79bb29e5-1e3b-49ce-bc37-a5eea273530e" />


