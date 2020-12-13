---
layout:      post                                   # 使用的布局（不需要改）
title:       Linux下FTP服务器快速搭建指南          # 标题 
subtitle:   Ubuntu 20.04                              # 副标题
date:        2020-12-212                            # 时间
author:      zhaiyunfan                           # 作者
header-img: img/post-bg-everyday.jpg           #这篇文章标题背景图片
catalog: true                                       # 是否归档
tags:                                               # 标签
    - Linux
    - Ubuntu
    - FTP
---



## 1.用户创建

> 使用系统原有root用户也可以，但安全性较差

这里所说的用户创建指创建一个Linux下的用户，以后访问FTP服务器时将使用这个用户来登录，这个用户同样可以用于SSH

在新建用户之前，先新建该用户的家目录

```bash
sudo mkdir /home/ftpfile    
sudo chmod -R 777 /home/ftpfile
# 更改一下目录权限，否则无法向其中写入内容
```

然后创建Ubuntu系统用户

```bash
# -d参数：指定登入时的起始目录	-s参数：指定用户登录后所使用的默认shell
sudo useradd -d /home/ftpfile  -s /bin/bash ftpuser
sudo passwd ftpuser
# 接下来连续两次输入密码并确认
```

## 2.安装vsftpd服务器

直接使用apt进行安装

```bash
sudo apt install vsftpd
```

## 3.FTP配置

### 修改配置文件

使用任意文本编辑器修改vsftpd的配置文件

```bash
sudo vim /etc/vsftpd.conf
```

写入以下内容

```
# Example config file /etc/vsftpd.conf
#
# The default compiled in settings are fairly paranoid. This sample file
# loosens things up a bit, to make the ftp daemon more usable.
# Please see vsftpd.conf.5 for all compiled in defaults.
#
# READ THIS: This example file is NOT an exhaustive list of vsftpd options.
# Please read the vsftpd.conf.5 manual page to get a full idea of vsftpd's
# capabilities.
#
#
# Run standalone?  vsftpd can run either from an inetd or as a standalone
# daemon started from an initscript.
listen=NO # 服务器监听  
#
# This directive enables listening on IPv6 sockets. By default, listening
# on the IPv6 "any" address (::) will accept connections from both IPv6
# and IPv4 clients. It is not necessary to listen on *both* IPv4 and IPv6
# sockets. If you want that (perhaps because you want to listen on specific
# addresses) then you must run two copies of vsftpd with two configuration
# files.
listen_ipv6=YES
#
# Allow  anonymous FTP? (Disabled by default).
anonymous_enable=NO # 匿名访问允许，默认不要开启。yes的话就可以不登录访问  
#
# Uncomment this to allow local users to log in.
local_enable=YES # 是否允许本地用户访问  
#
# Uncomment this to enable any form of FTP write command.
write_enable=YES # 是否允许上传文件，不开启会报 550 permission denied  
#
# Default umask for local users is 077. You may wish to change this to 022,
# if your users expect that (022 is used by most other ftpd's)
local_umask=022 # FTP上本地的文件权限，默认是077  022?
#
# Uncomment this to allow the anonymous FTP user to  upload files. This only
# has an effect if the above global write enable is activated. Also, you will
# obviously need to create a directory writable by the FTP user.
#anon_upload_enable=YES # 匿名上传允许，默认是NO  
#
# Uncomment this if you want the anonymous FTP user to be able to create
# new directories.
#anon_mkdir_write_enable=YES  # 匿名创建文件夹允许  
#
# Activate directory messages - messages given to remote users when they
# go into a certain directory.
dirmessage_enable=YES # 进入文件夹允许  
#
# If enabled, vsftpd will display directory listings with the time
# in  your  local  time  zone.  The default is to display GMT. The
# times returned by the MDTM FTP command are also affected by this
# option.
use_localtime=YES
#
# Activate logging of uploads/downloads.
xferlog_enable=YES # ftp 日志记录允许  
#
# Make sure PORT transfer connections originate from port 20 (ftp-data).
connect_from_port_20=YES # 启用20号端口作为数据传送的端口  
#
# If you want, you can arrange for uploaded anonymous files to be owned by
# a different user. Note! Using "root" for uploaded files is not
# recommended!
#chown_uploads=YES
#chown_username=whoever
#
# You may override where the log file goes if you like. The default is shown
# below.
xferlog_file=/var/log/vsftpd.log
#
# If you want, you can have your log file in standard ftpd xferlog format.
# Note that the default log file location is /var/log/xferlog in this case.
xferlog_std_format=YES
#
# You may change the default value for timing out an idle session.
#idle_session_timeout=600
#
# You may change the default value for timing out a data connection.
#data_connection_timeout=120
#
# It is recommended that you define on your system a unique user which the
# ftp server can use as a totally isolated and unprivileged user.
#nopriv_user=ftpsecure
#
# Enable this and the server will recognise asynchronous ABOR requests. Not
# recommended for security (the code is non-trivial). Not enabling it,
# however, may confuse older FTP clients.
#async_abor_enable=YES
#
# By default the server will pretend to allow ASCII mode but in fact ignore
# the request. Turn on the below options to have the server actually do ASCII
# mangling on files when in ASCII mode.
# Beware that on some FTP servers, ASCII support allows a denial of service
# attack (DoS) via the command "SIZE /big/file" in ASCII mode. vsftpd
# predicted this attack and has always been safe, reporting the size of the
# raw file.
# ASCII mangling is a horrible feature of the protocol.
#ascii_upload_enable=YES
#ascii_download_enable=YES
#
# You may fully customise the login banner string:
ftpd_banner=Welcome to blah FTP service.# 欢迎信息  
#
# You may specify a file of disallowed anonymous e-mail addresses. Apparently
# useful for combatting certain DoS attacks.
#deny_email_enable=YES
# (default follows)
#banned_email_file=/etc/vsftpd.banned_emails
#
# You may restrict local users to their home directories.  See the FAQ for
# the possible risks in this before using chroot_local_user or
# chroot_list_enable below.
#chroot_local_user=YES  # 用于指定用户列表文件中的用户是否允许切换到上级目录。默认值为NO。  
#
# You may specify an explicit list of local users to chroot() to their home
# directory. If chroot_local_user is YES, then this list becomes a list of
# users to NOT chroot().
# (Warning! chroot'ing can be very dangerous. If using chroot, make sure that
# the user does not have write access to the top level directory within the
# chroot)
chroot_local_user=YES
chroot_list_enable=YES # 设置是否启用chroot_list_file配置项指定的用户列表文件。默认值为NO。
local_root=/home/ftpfile # 设置一个本地用户登录后进入到的目录，可以定义其他目录，与刚才用户创建的目录无关，但最好放这里
# (default follows)
chroot_list_file=/etc/vsftpd.chroot_list # 格式为一行一个用户，用于指定用户列表文件，该文件用于控制哪些用户可以切换到用户家目录的上级目录。
#
# You may activate the "-R" option to the builtin ls. This is disabled by
# default to avoid remote users being able to cause excessive I/O on large
# sites. However, some broken FTP clients such as "ncftp" and "mirror" assume
# the presence of the "-R" option, so there is a strong case for enabling it.
#ls_recurse_enable=YES
#
# Customization
#
# Some of vsftpd's settings don't fit the filesystem layout by
# default.
#
# This option should be the name of a directory which is empty.  Also, the
# directory should not be writable by the ftp user. This directory is used
# as a secure chroot() jail at times vsftpd does not require filesystem
# access.
secure_chroot_dir=/var/run/vsftpd/empty
#
# This string is the name of the PAM service vsftpd will use.
# !!!!!!!!!!!****************************!!!!!!!! 
pam_service_name=ftp
# !!!!!!!!!!!****************************!!!!!!!! 一定要把上句改好，这一句非常重要
#
# This option specifies the location of the RSA certificate to use for SSL
# encrypted connections.
rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
ssl_enable=NO

#
# Uncomment this to indicate that vsftpd use a utf8 filesystem.
utf8_filesystem=YES
```

接下来编辑/etc/vsftpd.chroot_list文件，将ftpuser的帐户名添加进去，保存退出。注意使用你刚创建的账户即可

```bash
sudo vim /etc/vsftpd.chroot_list
# 用vim操作不存在的文件，即创建文件时，需要sudo提权
# 在这个文件里写上刚刚创建的用户名即可
```

### 重新启动vsftpd并查看服务状态

使用service命令

```bash
sudo service vsftpd restart	# 重启服务
service vsftpd status		# 查看状态
```

如果显示类似以下文本，则启动成功，按Q退出

```bash
● vsftpd.service - vsftpd FTP server
     Loaded: loaded (/lib/systemd/system/vsftpd.service; enabled; vendor preset>
     Active: active (running) since Sun 2020-12-13 22:19:04 CST; 9s ago
    Process: 46618 ExecStartPre=/bin/mkdir -p /var/run/vsftpd/empty (code=exite>
   Main PID: 46620 (vsftpd)
      Tasks: 1 (limit: 9347)
     Memory: 660.0K
     CGroup: /system.slice/vsftpd.service
             └─46620 /usr/sbin/vsftpd /etc/vsftpd.conf

12月 13 22:19:04 zhaiyunfan systemd[1]: Starting vsftpd FTP server...
12月 13 22:19:04 zhaiyunfan systemd[1]: Started vsftpd FTP server.

```

### 打开防火墙

```bash
sudo ufw disable
```

## 享受你的FTP服务器
