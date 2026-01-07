## 一、说明
这是一个用来测试的，模拟安装后，执行升级程序，程序仍在运行，用户输入了N，不停止，以此探索四个维护脚本执行逻辑。

## 二、实验一
### （一）、正常安装
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
### (二）正常卸载
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
### (三)、正常升级
dpkg -i 
- 卸载前执行的脚本 - prerm -> $1: upgrade
- 安装前执行的脚本 - preinst -> $1: upgrade
- 卸载后执行的脚本 - postrm -> $1: upgrade
- 安装后执行的脚本 -> postinst $1: configure

```bash                                                                                                                                                             
┌──(root㉿kali)-[/home/vbgaga/vscode-project/tool2deb/project]
└─# dpkg -i helloworld.deb
(正在读取数据库 ... 系统当前共安装有 463716 个文件和目录。)
准备解压 helloworld.deb  ...
卸载前执行的脚本 - prerm -> $1: upgrade
安装前执行的脚本 - preinst -> $1: upgrade
正在解压 helloworld (2025.12.2) 并覆盖 (2025.12.2) ...
卸载后执行的脚本 - postrm -> $1: upgrade
正在设置 helloworld (2025.12.2) ...
安装后执行的脚本 -> postinst $1: configure
正在处理用于 kali-menu (2023.2.3) 的触发器 ...
```

## 三、实验二 - 模拟出现升级、卸载时候出现程序未退出情况
### （一）、升级

### （二）、卸载
