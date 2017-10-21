## SAMBA实现文件共享
@(Linux)[samba]
#### 修改SAMBA配置文件
vim /etc/samba/smb.cnf

``` dsconfig
[global]
 workgroup = MYGROUP
 server string = Samba Server Version %v
 log file = /var/log/samba/log.%m
 max log size = 50
 security = user
 passdb backend = tdbsam
 load printers = yes
 cups options = raw
[database]
 comment = Do not arbitrarily modify the database file
 path = /home/database
 public = no
 writable = yes
```

#### 创建用于访问共享资源的帐户信息
``` bash
[root@linuxprobe ~]# id linuxprobe
uid=1000(linuxprobe) gid=1000(linuxprobe) groups=1000(linuxprobe)
[root@linuxprobe ~]# pdbedit -a -u linuxprobe
new password:此处输入该用户在Samba服务数据库中的密码
retype new password:再次输入密码进行确认
Unix username: linuxprobe
NT username: 
Account Flags: [U ]
User SID: S-1-5-21-507407404-3243012849-3065158664-1000
Primary Group SID: S-1-5-21-507407404-3243012849-3065158664-513
Full Name: linuxprobe
Home Directory: \\localhost\linuxprobe
HomeDir Drive: 
Logon Script: 
Profile Path: \\localhost\linuxprobe\profile
Domain: LOCALHOST
Account desc: 
Workstations: 
Munged dial: 
Logon time: 0
Logoff time: Wed, 06 Feb 2036 10:06:39 EST
Kickoff time: Wed, 06 Feb 2036 10:06:39 EST
Password last set: Mon, 13 Mar 2017 04:22:25 EDT
Password can change: Mon, 13 Mar 2017 04:22:25 EDT
Password must change: never
Last bad password : 0
Bad password count : 0
Logon hours : FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF
```

#### 创建用于共享资源的文件目录
``` bash
[root@linuxprobe ~]# mkdir /home/database
[root@linuxprobe ~]# chown -Rf linuxprobe:linuxprobe /home/database
[root@linuxprobe ~]# semanage fcontext -a -t samba_share_t /home/database
[root@linuxprobe ~]# restorecon -Rv /home/database
restorecon reset /home/database context unconfined_u:object_r:home_root_t:s0->unconfined_u:object_r:samba_share_t:s0
```

#### 参考文章
[使用Samba或NFS实现文件共享。](http://www.linuxprobe.com/chapter-12.html)

