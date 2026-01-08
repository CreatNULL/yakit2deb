## 一、说明
这是一个用来测试，的项目，探索四个维护脚本执行逻辑。

测试代码，测试的时候，将代码分别放到不同的脚本，然后打包运行 helloword，启动，然后尝试 升级/卸载
```basb
check_some_run() {
    # Check if any Burp Suite processes are running
    if pgrep -f "python\s+-m\s+http.server.*" >/dev/null; then
        echo "python server is running"
        echo "-------------------------------"
        ps -aux | grep -E ".*python\s+-m\s+http.server.*" | grep -v grep
        echo "-------------------------------"
        
        read -p "Stop these processes? (y/N): " -n 1
        echo ""
        
        if [[ $REPLY =~ ^[Yy]$ ]]; then
            echo "Stopping processes..."
            
            # Attempt graceful termination
            pkill -f "python\s+-m\s+http.server.*" 2>/dev/null || true
            sleep 2
            
            # Check if any processes remain
            if pgrep -f "python\s+-m\s+http.server.*" >/dev/null; then
                echo "Some processes still running, attempting forceful termination..."
                pkill -9 -f "python\s+-m\s+http.server.*" 2>/dev/null || true
                sleep 1
            fi
            
            # Final check
            if pgrep -f "python\s+-m\s+http.server.*" >/dev/null; then
                echo "Unable to stop all processes. Please manually close Burp Suite."
                exit 1
            else
                echo "All processes terminated successfully."
            fi
        else
            echo "Please close Burp Suite manually before continuing."
            exit 1
        fi
    fi
}

check_some_run
```

确保每次都完全卸载，重新构建

正常升级：
- 先完全卸载（dpkg -P helloworld )防止以前的影响
- 构建老的包
- 安装
- 构建新的包：然后（安装前执行的脚本 - preinst -> $1: install）添加 new，代表新版本
- 构建
- 安装

# 一、模拟安装过程出现问题
> 安装过程中安装前脚本出现问题、安装过程中安装后执行脚本出现问题、安装过程中卸载前脚本出现问题、安装过程中卸载后脚本出现问题

## 实验一、正常安装 （首次安装）
dpkg -i helloworld.deb
- 安装前执行的脚本 - preinst -> $1: install
- 安装后执行的脚本 -> postinst $1: configure
输出:<br />
```bash
──(root㉿kali)-[/home/vbgaga/vscode-project/tool2deb/project]
└─# dpkg -i helloworld.deb                        
正在选中未选择的软件包 helloworld。
(正在读取数据库 ... 系统当前共安装有 463715 个文件和目录。)
准备解压 helloworld.deb  ...
安装前执行的脚本 - preinst -> $1: install
正在解压 helloworld (2025.12.2) ...
正在设置 helloworld (2025.12.2) ...
按装后执行的脚本 -> postinst $1: configure
正在处理用于 kali-menu (2023.2.3) 的触发器 ...
```

## 实验二、正常升级
dpkg -i 
- 卸载前执行的脚本 - prerm -> $1: upgrade
- 安装前执行的脚本 - preinst -> $1: upgrade new
- 卸载后执行的脚本 - postrm -> $1: upgrade
- 安装后执行的脚本 -> postinst $1: configure new

```bash                                                                                                                                                           ┌──(root㉿kali)-[/home/vbgaga/vscode-project/tool2deb/project]
└─# dpkg -i helloworld.deb                                                                  
(正在读取数据库 ... 系统当前共安装有 463716 个文件和目录。)
准备解压 helloworld.deb  ...
卸载前执行的脚本 - prerm -> $1: upgrade
安装前执行的脚本 - preinst -> $1: upgrade new
正在解压 helloworld (2025.12.2) 并覆盖 (2025.12.2) ...
卸载后执行的脚本 - postrm -> $1: upgrade
正在设置 helloworld (2025.12.2) ...
安装后执行的脚本 -> postinst $1: configure new
正在处理用于 kali-menu (2023.2.3) 的触发器 ...
```

## 实验三、模拟升级 - 安装前脚本出现问题
- 卸载前执行的脚本 - prerm -> $1: upgrade
- 安装前执行的脚本 - preinst -> $1: upgrade new
- 卸载后执行的脚本 - postrm -> $1: abort-upgrade new
- 安装后执行的脚本 -> postinst $1: abort-upgrade

```bash
┌──(root㉿kali)-[/home/vbgaga/vscode-project/tool2deb/project]
└─# dpkg -i helloworld.deb                                                                  
(正在读取数据库 ... 系统当前共安装有 463716 个文件和目录。)
准备解压 helloworld.deb  ...
卸载前执行的脚本 - prerm -> $1: upgrade
安装前执行的脚本 - preinst -> $1: upgrade new
python server is running
-------------------------------
root      823404  0.5  0.4  32216 19048 pts/0    S    11:07   0:00 python -m http.server 65232
-------------------------------
Stop these processes? (y/N): n
Please close Burp Suite manually before continuing.
dpkg: 处理归档 helloworld.deb (--install)时出错：
 新的 helloworld 软件包 pre-installation 脚本 子进程返回错误状态 1
卸载后执行的脚本 - postrm -> $1: abort-upgrade new
安装后执行的脚本 -> postinst $1: abort-upgrade
在处理时有错误发生：
 helloworld.deb               
```

## 实验四、模拟升级 - 安装后脚本出现问题
- 卸载前执行的脚本 - prerm -> $1: upgrade
- 安装前执行的脚本 - preinst -> $1: upgrade new
- 卸载后执行的脚本 - postrm -> $1: upgrade
- 安装后执行的脚本 -> postinst $1: configure new

```bash
┌──(root㉿kali)-[/home/vbgaga/vscode-project/tool2deb/project]
└─# dpkg-deb --root-owner-group --build helloworld;dpkg -i helloworld.deb                   
dpkg-deb: 正在 'helloworld.deb' 中构建软件包 'helloworld'。
(正在读取数据库 ... 系统当前共安装有 463716 个文件和目录。)
准备解压 helloworld.deb  ...
卸载前执行的脚本 - prerm -> $1: upgrade
安装前执行的脚本 - preinst -> $1: upgrade new
正在解压 helloworld (2025.12.2) 并覆盖 (2025.12.2) ...
卸载后执行的脚本 - postrm -> $1: upgrade
正在设置 helloworld (2025.12.2) ...
安装后执行的脚本 -> postinst $1: configure new
python server is running
-------------------------------
root      827931  0.1  0.4  32216 19048 pts/0    S    11:10   0:00 python -m http.server 65232
-------------------------------
Stop these processes? (y/N): n
Please close Burp Suite manually before continuing.
dpkg: 处理软件包 helloworld (--install)时出错：
 已安装 helloworld 软件包 post-installation 脚本 子进程返回错误状态 1
正在处理用于 kali-menu (2023.2.3) 的触发器 ...
在处理时有错误发生：
 helloworld
```

## 三、实验五 - 模拟安装升级 - 卸载前脚本出现问题
### （N、N）
- 卸载前执行的脚本 - prerm -> $1: upgrade
- 卸载前执行的脚本 - prerm -> $1: failed-upgrade new
- 安装后执行的脚本 -> postinst $1: abort-upgrade

```bash
──(root㉿kali)-[/home/vbgaga/vscode-project/tool2deb/project]
└─# dpkg -i helloworld.deb                         
(正在读取数据库 ... 系统当前共安装有 463716 个文件和目录。)
准备解压 helloworld.deb  ...
卸载前执行的脚本 - prerm -> $1: upgrade
python server is running
-------------------------------
root      772311  0.0  0.4  32216 19040 pts/0    S    10:23   0:00 python -m http.server 65232
-------------------------------
Stop these processes? (y/N): n
Please close Burp Suite manually before continuing.
dpkg: 警告: 旧的 helloworld 软件包 pre-removal 脚本 子进程返回错误状态 1
dpkg: 现在尝试使用新软件包所带的脚本...
卸载前执行的脚本 - prerm -> $1: failed-upgrade new
python server is running
-------------------------------
root      772311  0.0  0.4  32216 19040 pts/0    S    10:23   0:00 python -m http.server 65232
-------------------------------
Stop these processes? (y/N): n
Please close Burp Suite manually before continuing.
dpkg: 处理归档 helloworld.deb (--install)时出错：
 新的 helloworld 软件包 pre-removal 脚本 子进程返回错误状态 1
安装后执行的脚本 -> postinst $1: abort-upgrade
在处理时有错误发生：
 helloworld.deb
```

### （N、Y）
- 卸载前执行的脚本 - prerm -> $1: upgrade
- 卸载前执行的脚本 - prerm -> $1: failed-upgrade new
- 安装前执行的脚本 - preinst -> $1: upgrade new
- 安装后执行的脚本 -> postinst $1: configure new

```bash  
┌──(root㉿kali)-[/home/vbgaga/vscode-project/tool2deb/project]
└─# dpkg -i helloworld.deb
(正在读取数据库 ... 系统当前共安装有 463716 个文件和目录。)
准备解压 helloworld.deb  ...
卸载前执行的脚本 - prerm -> $1: upgrade
python server is running
-------------------------------
root      772311  0.0  0.4  32216 19040 pts/0    S    10:23   0:00 python -m http.server 65232
-------------------------------
Stop these processes? (y/N): n
Please close Burp Suite manually before continuing.
dpkg: 警告: 旧的 helloworld 软件包 pre-removal 脚本 子进程返回错误状态 1
dpkg: 现在尝试使用新软件包所带的脚本...
卸载前执行的脚本 - prerm -> $1: failed-upgrade new
python server is running
-------------------------------
root      772311  0.0  0.4  32216 19040 pts/0    S    10:23   0:00 python -m http.server 65232
-------------------------------
Stop these processes? (y/N): y
Stopping processes...
All processes terminated successfully.
dpkg: ... 它看起来没有问题
安装前执行的脚本 - preinst -> $1: upgrade new
正在解压 helloworld (2025.12.2) 并覆盖 (2025.12.2) ...
卸载后执行的脚本 - postrm -> $1: upgrade
正在设置 helloworld (2025.12.2) ...
安装后执行的脚本 -> postinst $1: configure new
正在处理用于 kali-menu (2023.2.3) 的触发器 ...
```
## 实验六、模拟安装升级 - 卸载后执行脚本出现问题

### (N,N,N)
- 卸载前执行的脚本 - prerm -> $1: upgrade
- 安装前执行的脚本 - preinst -> $1: upgrade new
- 卸载后执行的脚本 - postrm -> $1: upgrade
- 卸载后执行的脚本 - postrm -> $1: failed-upgrade new
- 安装前执行的脚本 - preinst -> $1: abort-upgrade
- 卸载后执行的脚本 - postrm -> $1: abort-upgrade new
```     
┌──(root㉿kali)-[/home/vbgaga/vscode-project/tool2deb/project]
└─# dpkg-deb --root-owner-group --build helloworld;dpkg -i helloworld.deb                   
dpkg-deb: 正在 'helloworld.deb' 中构建软件包 'helloworld'。
(正在读取数据库 ... 系统当前共安装有 463716 个文件和目录。)
准备解压 helloworld.deb  ...
卸载前执行的脚本 - prerm -> $1: upgrade
安装前执行的脚本 - preinst -> $1: upgrade new
正在解压 helloworld (2025.12.2) 并覆盖 (2025.12.2) ...
卸载后执行的脚本 - postrm -> $1: upgrade
python server is running
-------------------------------
root      834814  0.2  0.4  32216 18964 pts/0    S    11:16   0:00 python -m http.server 65232
-------------------------------
Stop these processes? (y/N): n
Please close Burp Suite manually before continuing.
dpkg: 警告: 旧的 helloworld 软件包 post-removal 脚本 子进程返回错误状态 1
dpkg: 现在尝试使用新软件包所带的脚本...
卸载后执行的脚本 - postrm -> $1: failed-upgrade new
python server is running
-------------------------------
root      834814  0.2  0.4  32216 18964 pts/0    S    11:16   0:00 python -m http.server 65232
-------------------------------
Stop these processes? (y/N): n
Please close Burp Suite manually before continuing.
dpkg: 处理归档 helloworld.deb (--install)时出错：
 新的 helloworld 软件包 post-removal 脚本 子进程返回错误状态 1
安装前执行的脚本 - preinst -> $1: abort-upgrade
卸载后执行的脚本 - postrm -> $1: abort-upgrade new
python server is running
-------------------------------
root      834814  0.2  0.4  32216 18964 pts/0    S    11:16   0:00 python -m http.server 65232
-------------------------------
Stop these processes? (y/N): n
Please close Burp Suite manually before continuing.
dpkg: 清理时出错:
 新的 helloworld 软件包 post-removal 脚本 子进程返回错误状态 1
正在处理用于 kali-menu (2023.2.3) 的触发器 ...
在处理时有错误发生：
 helloworld.deb              
```

### (N, N, Y)
- 安装前执行的脚本 - preinst -> $1: upgrade new
- 卸载后执行的脚本 - postrm -> $1: upgrade
- 卸载后执行的脚本 - postrm -> $1: failed-upgrade new
- 安装前执行的脚本 - preinst -> $1: abort-upgrade
- 卸载后执行的脚本 - postrm -> $1: abort-upgrade new

```
┌──(root㉿kali)-[/home/vbgaga/vscode-project/tool2deb/project]
└─# dpkg -i helloworld.deb
(正在读取数据库 ... 系统当前共安装有 463716 个文件和目录。)
准备解压 helloworld.deb  ...
安装前执行的脚本 - preinst -> $1: upgrade new
正在解压 helloworld (2025.12.2) 并覆盖 (2025.12.2) ...
卸载后执行的脚本 - postrm -> $1: upgrade
python server is running
-------------------------------
root      799783  0.0  0.4  32216 18984 pts/0    S    10:47   0:00 python -m http.server 65232
-------------------------------
Stop these processes? (y/N): n
Please close Burp Suite manually before continuing.
dpkg: 警告: 旧的 helloworld 软件包 post-removal 脚本 子进程返回错误状态 1
dpkg: 现在尝试使用新软件包所带的脚本...
卸载后执行的脚本 - postrm -> $1: failed-upgrade new
python server is running
-------------------------------
root      799783  0.0  0.4  32216 18984 pts/0    S    10:47   0:00 python -m http.server 65232
-------------------------------
Stop these processes? (y/N): n
Please close Burp Suite manually before continuing.
dpkg: 处理归档 helloworld.deb (--install)时出错：
 新的 helloworld 软件包 post-removal 脚本 子进程返回错误状态 1
安装前执行的脚本 - preinst -> $1: abort-upgrade
卸载后执行的脚本 - postrm -> $1: abort-upgrade new
python server is running
-------------------------------
root      799783  0.0  0.4  32216 18984 pts/0    S    10:47   0:00 python -m http.server 65232
-------------------------------
Stop these processes? (y/N): y
Stopping processes...
All processes terminated successfully.
正在处理用于 kali-menu (2023.2.3) 的触发器 ...
在处理时有错误发生：
 helloworld.deb
```
### (N,Y)
- 卸载前执行的脚本 - prerm -> $1: upgrade
- 安装前执行的脚本 - preinst -> $1: upgrade new
- 卸载后执行的脚本 - postrm -> $1: upgrade
- 卸载后执行的脚本 - postrm -> $1: failed-upgrade new
- 安装后执行的脚本 -> postinst $1: configure new

```bash
┌──(root㉿kali)-[/home/vbgaga/vscode-project/tool2deb/project]
└─# dpkg-deb --root-owner-group --build helloworld;dpkg -i helloworld.deb
dpkg-deb: 正在 'helloworld.deb' 中构建软件包 'helloworld'。
(正在读取数据库 ... 系统当前共安装有 463716 个文件和目录。)
准备解压 helloworld.deb  ...
卸载前执行的脚本 - prerm -> $1: upgrade
安装前执行的脚本 - preinst -> $1: upgrade new
正在解压 helloworld (2025.12.2) 并覆盖 (2025.12.2) ...
卸载后执行的脚本 - postrm -> $1: upgrade
python server is running
-------------------------------
root      840408  0.1  0.4  32216 18960 pts/0    S    11:20   0:00 python -m http.server 65232
-------------------------------
Stop these processes? (y/N): n
Please close Burp Suite manually before continuing.
dpkg: 警告: 旧的 helloworld 软件包 post-removal 脚本 子进程返回错误状态 1
dpkg: 现在尝试使用新软件包所带的脚本...
卸载后执行的脚本 - postrm -> $1: failed-upgrade new
python server is running
-------------------------------
root      840408  0.1  0.4  32216 18960 pts/0    S    11:20   0:00 python -m http.server 65232
-------------------------------
Stop these processes? (y/N): y
Stopping processes...
All processes terminated successfully.
dpkg: ... 它看起来没有问题
正在设置 helloworld (2025.12.2) ...
安装后执行的脚本 -> postinst $1: configure new
正在处理用于 kali-menu (2023.2.3) 的触发器 ...                                                        
```


# 二、卸载
### 实验一、正常卸载
dpkg -r 或 apt-get remove
- 卸载前执行的脚本 - prerm -> $1: remove
- 卸载后执行的脚本 - postrm -> $1: remove

dpkg -P 
- 卸载前执行的脚本 - prerm -> $1: remove
- 卸载后执行的脚本 - postrm -> $1: remove
- 卸载后执行的脚本 - postrm -> $1: purge
输出:<br />
```bash                                                                                                                                                                                                                                                                                                                             
┌──(root㉿kali)-[/home/vbgaga/vscode-project/tool2deb/project]
└─# dpkg -r helloworld                            
(正在读取数据库 ... 系统当前共安装有 463704 个文件和目录。)
正在卸载 helloworld (2025.12.2) ...
卸载前执行的脚本 - prerm -> $1: remove
卸载后执行的脚本 - postrm -> $1: remove
卸载中
正在处理用于 kali-menu (2023.2.3) 的触发器 ...
                                                                                                                                                                            
┌──(root㉿kali)-[/home/vbgaga/vscode-project/tool2deb/project]
└─# dpkg -i helloworld.deb
正在选中未选择的软件包 helloworld。
(正在读取数据库 ... 系统当前共安装有 463715 个文件和目录。)
准备解压 helloworld.deb  ...
安装前执行的脚本 - preinst -> $1: install
正在解压 helloworld (2025.12.2) ...
正在设置 helloworld (2025.12.2) ...
安装后执行的脚本 -> postinst $1: configure
正在处理用于 kali-menu (2023.2.3) 的触发器 ...
                                                                                                                                                                            
┌──(root㉿kali)-[/home/vbgaga/vscode-project/tool2deb/project]
└─# dpkg -P helloworld    
(正在读取数据库 ... 系统当前共安装有 463704 个文件和目录。)
正在卸载 helloworld (2025.12.2) ...
卸载前执行的脚本 - prerm -> $1: remove
卸载后执行的脚本 - postrm -> $1: remove
卸载中
正在清除 helloworld (2025.12.2) 的配置文件 ...
卸载后执行的脚本 - postrm -> $1: purge
卸载中
正在处理用于 kali-menu (2023.2.3) 的触发器 ...
```
### 卸载中 - 卸载前脚本遇到问题
- 卸载前执行的脚本 - prerm -> $1: remove
- 安装后执行的脚本 -> postinst $1: abort-remove
```
┌──(root㉿kali)-[/home/vbgaga/vscode-project/tool2deb/project]
└─# dpkg -r helloworld                                                                      
(正在读取数据库 ... 系统当前共安装有 463704 个文件和目录。)
正在卸载 helloworld (2025.12.2) ...
卸载前执行的脚本 - prerm -> $1: remove
python server is running
-------------------------------
root      847842  0.6  0.4  32216 18960 pts/0    S    11:26   0:00 python -m http.server 65232
-------------------------------
Stop these processes? (y/N): n
Please close Burp Suite manually before continuing.
dpkg: 处理软件包 helloworld (--remove)时出错：
 已安装 helloworld 软件包 pre-removal 脚本 子进程返回错误状态 1
安装后执行的脚本 -> postinst $1: abort-remove
在处理时有错误发生：
 helloworld
```

- 卸载前执行的脚本 - prerm -> $1: remove
- 安装后执行的脚本 -> postinst $1: abort-remove
```bash
┌──(root㉿kali)-[/home/vbgaga/vscode-project/tool2deb/project]
└─# dpkg -P helloworld                                                                      
(正在读取数据库 ... 系统当前共安装有 463704 个文件和目录。)
正在卸载 helloworld (2025.12.2) ...
卸载前执行的脚本 - prerm -> $1: remove
python server is running
-------------------------------
root      855172  0.7  0.4  32216 19024 pts/0    S    11:32   0:00 python -m http.server 65232
-------------------------------
Stop these processes? (y/N): n
Please close Burp Suite manually before continuing.
dpkg: 处理软件包 helloworld (--purge)时出错：
 已安装 helloworld 软件包 pre-removal 脚本 子进程返回错误状态 1
安装后执行的脚本 -> postinst $1: abort-remove
在处理时有错误发生：
 helloworld
```


### 卸载中 - 卸载后脚本遇到问题
- 卸载前执行的脚本 - prerm -> $1: remove
- 卸载后执行的脚本 - postrm -> $1: remove
```
┌──(root㉿kali)-[/home/vbgaga/vscode-project/tool2deb/project]
└─# dpkg -r helloworld                                                                      
(正在读取数据库 ... 系统当前共安装有 463704 个文件和目录。)
正在卸载 helloworld (2025.12.2) ...
卸载前执行的脚本 - prerm -> $1: remove
卸载后执行的脚本 - postrm -> $1: remove
python server is running
-------------------------------
root      850570  0.3  0.4  32216 19020 pts/0    S    11:28   0:00 python -m http.server 65232
-------------------------------
Stop these processes? (y/N): n
Please close Burp Suite manually before continuing.
dpkg: 处理软件包 helloworld (--remove)时出错：
 已安装 helloworld 软件包 post-removal 脚本 子进程返回错误状态 1
正在处理用于 kali-menu (2023.2.3) 的触发器 ...
在处理时有错误发生：
 helloworld
```

- 卸载后执行的脚本 - postrm -> $1: remove
```bash                                                                                                                                                                     
┌──(root㉿kali)-[/home/vbgaga/vscode-project/tool2deb/project]
└─# dpkg -P helloworld                                                                      
(正在读取数据库 ... 系统当前共安装有 463703 个文件和目录。)
正在卸载 helloworld (2025.12.2) ...
卸载后执行的脚本 - postrm -> $1: remove
python server is running
-------------------------------
root      850570  0.0  0.4  32216 19020 pts/0    S    11:28   0:00 python -m http.server 65232
-------------------------------
Stop these processes? (y/N): n
Please close Burp Suite manually before continuing.
dpkg: 处理软件包 helloworld (--purge)时出错：
 已安装 helloworld 软件包 post-removal 脚本 子进程返回错误状态 1
在处理时有错误发生：
 helloworld
```
