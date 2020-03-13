# 静默安装ORACLE

安装前先确定原数据库的字符集，为避免数据导入乱码
```
select * from nls_database_parameters;
 
NLS_CHARACTERSET	AL32UTF8
NLS_NCHAR_CHARACTERSET	AL16UTF16
```
记录这个两个字段的字符编码集,在设置后面的相应文件```oracle解压路径/***/response/dbca.rsp```时候将对应的两个字段改为上面查询出来的两个值
 


######  1.编辑hosts文件,将本地的ip加入到hosts文件中
```
[root@localhost ~]# vim /etc/hosts
 ```
完成后可以测试一下是否成功,
```
[root@localhost~]# ping ***.***.**.***
 ```
######  2.配置 /etc/sysctl.conf
添加并修改以下数据
```
fs.file-max = 6815744
fs.aio-max-nr = 3145728
kernel.shmall = 2097152
kernel.shmmax = 4294967295
kernel.shmmni = 4096
kernel.sem = 250 32000 100 128
net.ipv4.ip_local_port_range = 9000 65500
net.core.rmem_default = 262144
net.core.rmem_max= 4194304
net.core.wmem_default= 262144
net.core.wmem_max= 1048576
```
修改完后并是配置文件生效
```
[root@localhost~]# /sbin/sysctl -p
```
 
 
######  3.安装 zip 和 unzip
```
[root@localhost~]# yum install zip unzip 
 ```
######  4. 安装依赖包
```
[root@localhost~]# yum install -y binutils \
compat-libcap1 \
compat-libstdc++-33 \
compat-libstdc++-33.i686 \
glibc \
glibc.i686 \
glibc-devel \
glibc-devel.i686 \
ksh \
libaio \
libaio.i686 \
libaio-devel \
libaio-devel.i686 \
libX11 \
libX11.i686 \
libXau \
libXau.i686 \
libXi \
libXi.i686 \
libXtst \
libXtst.i686 \
libgcc \
libgcc.i686 \
libstdc++ \
libstdc++.i686 \
libstdc++-devel \
libstdc++-devel.i686 \
libxcb \
libxcb.i686 \
make \
nfs-utils \
net-tools \
smartmontools \
sysstat \
unixODBC \
unixODBC-devel \
gcc \
gcc-c++ \
libXext \
libXext.i686 \
zlib-devel \
zlib-devel.i686
 ```
######  5. 创建用户和用户组
```
[root@localhost~]# groupadd oinstall
[root@localhost~]# groupadd dba 
[root@localhost~]# useradd  -g  oinstall -G dba oracle
[root@localhost~]# passwd oracle
```
 
######  6. 配置/etc/security/limits.d/20-nproc.conf
[root@localhost~]# vim /etc/security/limits.d/20-nproc.conf 
```
#加入下面内容
oracle   soft    nofile    1024  
oracle   hard   nofile    65536  
oracle   soft    nproc    16384  
oracle   hard   nproc    16384  
oracle   soft    stack    10240  
oracle   hard   stack    32768  
oracle   hard   memlock    134217728  
oracle   soft    memlock    134217728
```
 
 
###### 7关闭防火墙及selinux,因为听说不关会有意想不到的事情发生
```
[root@localhost~]# vi /etc/selinux/config
将SELINUX属性设置为disabled
SELINUX=disabled
 
Linux防火墙firewalld 开放1521端口
查看防火墙状态
[root@localhost~]#systemctl status firewalld
查看防火墙所有开放的端口
[root@localhost~]#firewall-cmd --zone=public --list-ports
[root@localhost~]#firewall-cmd --zone=public --add-port=1521/tcp --permanent
重启防火墙firewalld
[root@localhost~]# firewall-cmd --reload
 ```
###### 8.创建目录修改权限
 ```
[root@localhost~]# mkdir -p /u01/app/oracle/product/11.2.0/db_1
[root@localhost~]# chown -R oracle:oinstall /u01
[root@localhost~]# chmod -R 777 /u01
 ```
并在/etc/profile末尾增加oracle相关限制
```
[root@localhost~]# vim  /etc/profile
##此段最好手敲,复制粘贴的情况可能会出现语法错误
#ORACLE判断
if [ \$USER = "oracle" ]; then 
if [ \$SHELL = "/bin/ksh" ]; then  
ulimit -p 16384 
ulimit -n 65536
else 
ulimit -u 16384 -n 65536
fi
umask 022
fi
 ```
###### 9.配置oracle环境变量
```
[root@localhost~]#vim /home/oracle/.bash_profile
export ORACLE_HOSTNAME=localhost  
export ORACLE_UNQNAME=orcl
export ORACLE_BASE=/u01/app/oracle  
export ORACLE_HOME=$ORACLE_BASE/product/11.2.0/db_1  
export ORACLE_SID=orcl
export PATH=/usr/sbin:$PATH  
export PATH=$ORACLE_HOME/bin:$PATH
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib
export CLASSPATH=$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib
 ```
###### 10.重启linux  
```
[root@localhost~]#  reboot
```
 
###### 11.解压Linux oracle压缩包,确定解压路径,并执行unzip命令
```
[root@localhost database]#unzip linux.x64_11gR2_database_1of2.zip 
```
 
###### 12.切换到oracle用户再对oracle目录进行操作
切换命令 su 加上参数 -  (意为完全切换,包括环境变量)
```
[root@localhost database]# su - oracle
```
前往database/response路径下,备份oracle安装的响应文件db_install.rsp
```
[oracle@localhost reponse]$ cp db_install.rsp db_install.rsp.bak
```
修改oracle响应文件
```
[oracle@localhost database]$vim database/response/db_install.rsp
oracle.install.responseFileVersion=/oracle/install/rspfmt_dbinstall_response_schema_v12.2.0  
oracle.install.option=INSTALL_DB_SWONLY  
ORACLE_HOSTNAME=localhost
UNIX_GROUP_NAME=oinstall  
INVENTORY_LOCATION=/u01/app/oracle/oraInventory  
SELECTED_LANGUAGES=en,zh_CN
ORACLE_HOME=/u01/app/oracle/product/11.2.0/db_1  
ORACLE_BASE=/u01/app/oracle  
oracle.install.db.InstallEdition=EE
oracle.install.db.isCustomInstall=false
oracle.install.db.OSDBA_GROUP=dba
oracle.install.db.OSOPER_GROUP=dba 
oracle.install.db.config.starterdb.type=GENERAL_PURPOSE
#有的修改,没有的不管
oracle.install.db.OSBACKUPDBA_GROUP=dba  
oracle.install.db.OSDGDBA_GROUP=dba  
oracle.install.db.OSKMDBA_GROUP=dba  
oracle.install.db.OSRACDBA_GROUP=dba  
 
oracle.install.db.config.starterdb.type=GENERAL_PURPOSE  
oracle.install.db.config.starterdb.globalDBName=orcl
oracle.install.db.config.starterdb.SID=orcl
oracle.install.db.config.starterdb.characterSet=AL32UTF8 
oracle.install.db.config.starterdb.memoryOption=true 
oracle.install.db.config.starterdb.installExampleSchemas=false
oracle.install.db.config.starterdb.enableSecuritySettings=true
SECURITY_UPDATES_VIA_MYORACLESUPPORT=false 
DECLINE_SECURITY_UPDATES=true
 
 ```
开始静默安装oracle,并使用的是oracle用户
```
[oracle@localhost database]$ ./runInstaller -force -silent -noconfig -responseFile /data/database/database/response/db_install.rsp
 ```
如果警告
 
则需要在环境变量里面加入
```
export DISPLAY=(IP)***.**.**.**:1.0
```
若不行则执行命令unset DISPLAY
最后会显示successful complete 并说明另外连接一个终端,使用root用户执行两条脚本命令,然后再回到此连接敲回车
 ```
[root@localhost~]# /u01/app/oracle/oraInventory/orainstRoot.sh
[root@localhost~]# /u01/app/oracle/product/11.2.0/db_1/root.sh
```
至此,敲完Enter Oracle算是初步安装完成
接下来添加监听
```
[oracle @localhost response] $ cat netca.rsp | grep -Ev "^#|^$"
 
 
 
[oracle@localhost database]$netca -silent -responsefile /data/database/database/response/netca.rsp
 ```
 
 
若出现错误,根据提示在环境变量里面加入
```
Export DISPLAY=192.168.1.80:1.0
并执行命令 source /home/oracle/.bash_profile
```
使环境变量生效
 
查看监听状态
```
[oracle@localhost database]$lsnrctl status
 
 
创建数据库
首先备份响应文件
[oracle@localhost response]$ cp dbca.rsp dbca.rsp.bak
修改文件 /data/database/database/response/dbca.rsp
[oracle@localhost response]$ vim dbca.rsp
#修改参数配置如下
[CREATEDATABASE]
GDBNAME = "orcl"
SID = "orcl"
TEMPLATENAME = "General_Purpose.dbc"
SYSPASSWORD="121121"
SYSTEMPASSWORD="121121"
SYSMANPASSWORD="121121"
DBSNMPPASSWORD="121121"
DATAFILEDESTINATION="/u01/app/oracle"
STORAGETYPE="FS"
#根据第一步从数据库查询出来的编码集填写
CHARACTERSET="AL32UTF8"
NATIONALCHARACTERSET="AL16UTF16"
 ```
执行静默安装命令
 ```
[oracle@localhost response]$ dbca -createDatabase -silent 
-responseFile 
/u01/app/oracle/database/response/dbca.rsp
 
Oracle安装完成,最后连接数据库查看数据路状态
[oracle@localhost ~]$ sqlplus /nolog
SQL>conn /as sysdba
SQL>select open_mode from v$database;
 
SQL>select status from v$instance;
```
 
