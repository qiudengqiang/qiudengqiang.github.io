---
title: Linux Common Sense
date: 2020-11-21 22:51:51
tags: [Linux]
categories: [Linux]
---
# linux 的发展史

- Unix
- Minix
- Linux

# linux 的版本

- 内核版本
- 发行版本：redhat、ubuntu、centos

# linux 下常用的目录结构

- `/`：根目录
- `/home` ：系统默认的用户home目录，存放普通用户相关文件
- `/root`：管理员的home目录，存放root用户相关文件
- `/bin`：存放linux常用命令的目录，如ls、vi、cat等
- `/sbin`：需要一定权限才可以使用该目录下的命令
- `/etc`：存放系统配置相关文件
- `/tmp`：存放用户或正在执行的程序临时存放文件的目录
- `/usr`：安装一个软件的默认目录，相当于windows下的program files
- `/var`：存放经常变化的文件，如网络连接的sock文件、日志等
- `/boot`：存放引导系统启动的相关文件
- `/proc`：这是虚拟目录，不占磁盘空间。是系统内存的映射，访问这个目录来获取系统相关信息
- `/mnt`：默认挂在光驱和软驱的目录
- `/dev`：存放linux系统下的设备文件，访问该目录下某个文件，相当于访问某个设备
- `/srv`：service的缩写。该目录提供一些服务启动之后需要读取的数据
- `/sys`：linux 2.6内核之后一个变化，该目录下安装了一个2.6之后出现的新文件系统
- `/del`：类似于windows的设备管理器，把所有的硬件以文件的形式存储
- `/media`：用于临时挂载被的文件系统
- `/opt`：用于存放主机额外的软件，如oracle数据库就可以放在该目录下

# 终端命令格式

```
command [-opitons] [parameter]
```

说明：

- `command`：命令名
- `[-option]`：选项，可用来对命令进行控制和细化，也可省略
- `[parameter]`：传递给命令的参数，可以是0个、1个或多个
- `[]`：代表可选

# 查阅命令的帮助信息

- `command --help`：显示`command`命令的帮助信息
- `man command`：查阅`command`命令的使用手册
  - 使用`man`命令时的操作键：
  - `space`，显示下一页
  - `enter`，一次滚动手册页的一行
  - `b`，回滚一页
  - `f`，前滚一屏
  - `q`，退出
  - `/word`，搜索word字符串

# 常见的linux命令

- `ssh username@ip -p port`：链接远程服务器命令

- `exit`：退出登录

-  `ls `：查看当前文件下的内容

  - `ll`：`ls -l`的简写
  - `-a`：a=all，显示指定文件下所有子目录与文件，包括隐藏文件
  - `-l`：l=list，以列表形式显示详细信息
  - ` -h`：h=humanity，配合`-l`以人性化的方式展示文件大小

- `pwd`：查看当前所在文件夹

- `touch [文件名]`：新建文件

  - `touch file1 file2 ...`：创建多个文件

- `mkdir [目录名]`：创建目录

  - `mkdir -p dir1/dir2/dir3`：递归创建目录

- `rm [文件名]`：删除指定的文件名

  - `rmdir `/`rm -r`：删除文件夹
  - `-r `：recursion，表示删除目录
  - `-f `：force，强制
  - `-i`：interactive，以交互的方式删除

- `mv`：移动、重命名

  - `mv 1.txt 2.txt`
  - `-f`：强制移动，如有覆盖会给出提示
  - `-i`：交互式操作，如覆盖目标之前提示
  - `-v`：展示移动进度

- `cp`：文件或者目录的复制

  - `cp 1.txt 2.txt `
  - `-a`：保持文件原有属性
  - `-f`：覆盖已经存在的目标文件而不提示
  - `-i`：交互式复制，在覆盖目标文件之前提示
  - `-r`：目标文件必须为目录。会拷贝该目录下所有子目录和文件
  - `-v`：展示拷贝进度

- `find`：查找指定目录下的文件

  - 一般格式：`find 路径 -name 文件名`
  - `find . -name test.sh`：查找当前目录下名为test.sh的文件
  - `find . name '*.sh'`：查找当前目录下所有以.sh后缀结尾的文件

- `grep`：文本搜索

  - 一般格式：`grep [-options] '搜索内容' 文件名`
  - eg：`grep 'a' 1.txt`
  - `-v`：显示不包含匹配文本的所有行，即取反
  - `-n`：显示匹配行及行号
  - `-i`：忽略大小写
  - 搜索内容支持正则表达式
    - `^a`：表示以a开头。eg：`grep -n '^a' a.txt`
    - `b$`：表示以b结尾。eg：`grep -n 'b$' a.txt`
    - [a-z]：范围中的一个。eg：`grep -n [a-z][a-z][a-z].txt b.txt`

- `tar`：归档管理

  - 一般格式：`tar [参数] 打包文件名 文件`
  - 打包 ：`tar cvf 包名.tar 文件名`
  - 解包 ：`tar xvf 包名.tar -C 路径`
  - `-c`：生成档案文件，创建打包文件
  - `-v`：列出归档解档的详细过程，显示进度
  - `-f`：指定档案文件名称
  - `-t`：列出档案中包含的文件
  - `-x`：解开档案文件
  - `-z` ：归档压缩。调用gzip的压缩功能，先打包后压缩。默认后缀名为 file.tar.gz
    - 打包：`tar cvzf 压缩包包名 文件1 文件2 ...`
    - 解包：`tar zxvf 包名 -C 路径`
  - 注意：`tar` 命令`[参数]`前面可以使用`-`，也可以省略

- `gzip`：文件压缩解压，用gzip压缩tar打包后的文件扩展名一般用`xxx.tar.gz`

  - `gzip [选项] 被压缩文件`
  - `-d`：解压
  - `-r`：压缩所有子目录

- `chmod`：修改文件权限

  ```
  一个文件创建完成后，默认会有三个组：属主，属组，其他。每个组默认会有三个权限
  属主owner，属组group，其他other
  rwx，       rwx，     rwx
  ```

  - 字母法：`chmod u/g/o/a +/-/= rwx filename`
    - `u`：user，表示文件的所有者
    - `g`：group，用户组
    - `o`：other，其他
    - `a`：all，以上三者皆是
    - `+`：增加权限
    - `-`：撤销权限
    - `=`：设定权限
    - `r`：read，读取权限
    - `w`：write，写入权限
    - `x`：excute，执行权限
  - 数字法：`chmod 751 filename`
    - `r`：读取权限，数字代号为`4`
    - `w`：写入权限，数字代号为`2`
    - `x`：执行权限，数字代号为`1`
    - `-`：不具有任何权限，数字代号为`0`
  - eg：`chmod u=rwx,g=rx,o=r filename` 等同于 `chmod u=7,g=6,o=1 filename`
  - `-R`：递归所有目录加上相同权限，如`chmod 777 test/ -R` 递归test目录下所有文件加777权限

- `who`：用于查看当前所有登录系统的用户信息
  - `-q`或`--count`：只显示用户的登录账号和登录用户的数量
  - `-u`或`--heading`：显示列标题
- `which`：查看命令所在位置，例如`which pwd`
- `passwd`：修改用户登录密码
- `clear`：清屏
- `shutdown`：关机
- `reboot`：重启
- `more`：分屏展示
  - `more text.txt`
  - `q`：退出
  - `space`：下一页
  - `h`：获取帮助
- `cat`：查看并合并文件内容
  - `cat file1 file2 ...`：合并多个文件查看
- 软连接 / 硬链接
  - 命令格式：`ln [-s] 源文件 链接文件`
  - `ln -s a.txt a_sl`：软连接，相当于win下面的快捷方式
    - 尽量使用绝对路径
    - 当源文件大小改变时，链接文件不受影响
    - 链接文件几乎不占用磁盘空间
    - 当源文件删除，链接文件失效
    - 软连接可以链接目录
  - `ln b.txt b_sl`：硬链接
    - 只能链接普通文件，不能链接目录
    - 链接文件几乎与源文件占用空间一致
    - 当源文件删除时候，链接文件不受影响
- 重定向
  - `ls > test.txt`：讲查询结果重定向输出到文件中，若文件不存在则创建。若存在则覆盖
  - `>`：输出重定向为覆盖
  - `>>`：输出重定向为追加至文件的尾部
- `cd [目录名]`：切换文件夹
  - `cd .`：进入当前路径
  - `cd ..`：返回上一级目录
  - `cd /`：返回根目录
  - `cd ~`：返回home目录
  - `cd -`：进入上次所在的目录

# 通配符

- `*` ：表示所有字符
- `?`：只代表任意一个字符
- `[a-z]`：表示a-z中任意一个字符
- `\`：反斜杠，转义字符

# head & trail

- `head -n filename`：查看文件的前n行
- `trail -n filename`：查看文件的后n行
- `trail -f filename`：动态查看文件的内容

# 查看进程和端口号

- 进程：`ps aux |grep ***`
  - `a`：all
  - `u`：user
- 端口号：`netstat -tnulpa | grep ***`
  - `t`：tcp
  - `n`：no
  - `u`：user
  - `l`：listen
  - `p`：port
  - `a`：all

# Reference

[1] https://zh.wikipedia.org/wiki/Linux

