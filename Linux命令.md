### Linux命令
@(Linux)[linux]
#### 1. 查看端口
lsof -i:端口号

#### 2. vim tab设置为4个空格
在.vimrc中添加以下代码后，重启vim即可实现按TAB产生4个空格：
```vim
set ts=4  (注：ts是tabstop的缩写，设TAB宽4个空格)
set expandtab
```

对于已保存的文件，可以使用下面的方法进行空格和TAB的替换：
TAB替换为空格：
```vim
:set ts=4
:set expandtab
:%retab!
```

空格替换为TAB：
```vim
:set ts=4
:set noexpandtab
:%retab!
```

加!是用于处理非空白字符之后的TAB，即所有的TAB，若不加!，则只处理行首的TAB。


#### 3.scp
`语法`
> scp(选项)(参数)

`选项`
> -1：使用ssh协议版本1； 
> -2：使用ssh协议版本2； 
> -4：使用ipv4； 
> -6：使用ipv6； 
> -B：以批处理模式运行；
>  -C：使用压缩； 
>  -F：指定ssh配置文件； 
>  -l：指定宽带限制； 
>  -o：指定使用的ssh选项； 
>  -P：指定远程主机的端口号； 
>  -p：保留文件的最后修改时间，最后访问时间和权限模式； 
>  -q：不显示复制进度； 
>  -r：以递归方式复制。

`参数`
>源文件：指定要复制的源文件。 
>目标文件：目标文件。格式为user@host：filename（文件名为目标文件的名称）。

`实例`
从远程复制到本地的scp命令与上面的命令雷同，只要将从本地复制到远程的命令后面2个参数互换顺序就行了。
 
- 从远处复制文件到本地目录 

```bash
scp root@10.10.10.10:/opt/soft/nginx-0.5.38.tar.gz /opt/soft/ 
#从10.10.10.10机器上的/opt/soft/的目录中下载nginx-0.5.38.tar.gz 文件到本地/opt/soft/目录中。
```

- 从远处复制到本地 

```bash
scp -r root@10.10.10.10:/opt/soft/mongodb /opt/soft/ 
#从10.10.10.10机器上的/opt/soft/中下载mongodb目录到本地的/opt/soft/目录来。  
```
- 上传本地文件到远程机器指定目录

```bash
scp /opt/soft/nginx-0.5.38.tar.gz root@10.10.10.10:/opt/soft/scptest 
#复制本地/opt/soft/目录下的文件nginx-0.5.38.tar.gz到远程机器10.10.10.10的opt/soft/scptest目录。
```
- 上传本地目录到远程机器指定目录 

```bash
scp -r /opt/soft/mongodb root@10.10.10.10:/opt/soft/scptest
#上传本地目录/opt/soft/mongodb到远程机器10.10.10.10上/opt/soft/scptest的目录中去。
```

#### 4.grep命令反转去掉配置文件注释内容
``` bash
grep -v '#' /etc/vsftpd/vsftpd.conf_bak > /etc/vsftpd/vsftpd.conf
```

 






