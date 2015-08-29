# redhat5.5 x64 安装oracle 11g

 - redhat5.5 x64 (rhel5.5 x64)
 - oracle 11.2.0.1 x64
## 一.准备工作
"#"后跟命令表示以操作系统下root用户操作;
"$"后跟命令表示以操作系统下Oracle用户操作;
 下载软件包
官网下载：http://www.oracle.com/technetwork/database/enterprise-edition/downloads/index.htm
linux_11gR2_database_1of2.zip 
linux_11gR2_database_2of2.zip

### 禁用selinux：
```
vi /etc/sysconfig/seliunx
   SELINUX=disabled 
```
### 安装包检查，用下面的语句如果出现“没有那个文件或目录”是不行的
```
rpm -qa | grep 	binutils-2.17.50.0.6
rpm -qa | grep 	compat-libstdc++-33-3.2.3
rpm -qa | grep 	compat-libstdc++-33-3.2.3
rpm -qa | grep 	elfutils-libelf-0.125
rpm -qa | grep 	elfutils-libelf-devel-0.125
rpm -qa | grep 	gcc-4.1.2
rpm -qa | grep 	gcc-c++-4.1.2
rpm -qa | grep 	glibc-2.5-24
rpm -qa | grep 	glibc-2.5-24
rpm -qa | grep 	glibc-common-2.5
rpm -qa | grep 	glibc-devel-2.5
rpm -qa | grep 	glibc-devel-2.5
rpm -qa | grep 	glibc-headers-2.5
rpm -qa | grep 	pdksh-5.2.14-36.el5
rpm -qa | grep 	libaio-0.3.106
rpm -qa | grep 	libaio-0.3.106
rpm -qa | grep 	libaio-devel-0.3.106
rpm -qa | grep 	libaio-devel-0.3.106
rpm -qa | grep 	libgcc-4.1.2
rpm -qa | grep 	libgcc-4.1.2 
rpm -qa | grep 	libstdc++-4.1.2
rpm -qa | grep 	libstdc++-4.1.2
rpm -qa | grep 	libstdc++-devel-4.1.2
rpm -qa | grep 	make-3.81
rpm -qa | grep 	sysstat-7.0.2
rpm -qa | grep 	unixODBC-2.2.11
rpm -qa | grep 	unixODBC-2.2.11
rpm -qa | grep 	unixODBC-devel-2.2.11
rpm -qa | grep 	unixODBC-devel-2.2.11

```

 
 
### 安装依赖包
需要用光盘制作yum源
#### 配置yum源
     如果有包没有安装就必须得先安装，但安装前要配置好yum源，用rpm方式安装太累。
配置yum源参考：按步骤做，也是我写的啦。
http://blog.csdn.net/jameshadoop/article/details/48054897

```
yum -y install make-3.81 compat-db* openmotif* libXp* binutils-2.17.50.0.6 gcc-4.1.2 libaio-0.3.106* glibc-2.5-24* compat-libstdc++-33-3.2.3* elfutils-libelf-0.125 elfutils-libelf-devel-0.125 glibc-common-2.5 glibc-devel-2.5* glibc-headers-2.5 gcc-c++-4.1.2 libaio-devel-0.3.106* libgcc-4.1.2* libstdc++-4.1.2* libstdc++-devel-4.1.2 sysstat-7.0.2 unixODBC-2.2.11* unixODBC-devel-2.2.11* ksh-20060214
```
安装完记得用上面的语句再检查一下啦
## 二.系统配置工作

### 创建用户和组
```
[root@red5v5 /]# groupadd oinstall 
[root@red5v5 /]# groupadd dba 
[root@red5v5 /]# useradd -g oinstall -G dba oracle 
[root@red5v5 /]# passwd oracle 
[root@red5v5 /]# id oracle 
uid=501(oracle) gid=501(oinstall) groups=501(oinstall),502(dba) context=root:system_r:unconfined_t:SystemLow-SystemHigh 
```
### 创建相应安装目录
不要忘了切换用户哦，仔细看下面命令
```
[root@red5v5 /]# chmod -R 777 /opt 
[root@red5v5 /]# su - oracle 
[oracle@red5v5 ~]$ cd /opt 
[oracle@red5v5 opt]$ mkdir oracle 
[oracle@red5v5 opt]$ mkdir oraInventory
```

### 修改系统内核参数
```
[root@red5v5 ~]vim /etc/sysctl.conf
fs.aio-max-nr = 1048576 
fs.file-max = 6815744 
kernel.shmall = 2097152                     //此行默认已有，确认不低于此数即可 
kernel.shmmax = 536870912                   //此行默认已有，确认不低于此数即可 
kernel.shmmni = 4096 
kernel.sem = 250 32000 100 128 
net.ipv4.ip_local_port_range = 9000 65500 
net.core.rmem_default = 262144 
net.core.rmem_max = 4194304 
net.core.wmem_default = 262144 
net.core.wmem_max = 1048576
```
### 执行生效命令
```[root@red5v5 ~]# sysctl -p```

### 调整会话限制
```
[root@red5v5 ~]# vim /etc/pam.d/login 文件最后添加：
session     required    pam_limits.so 

```
### 调整oracle用户的进程数与最大文件句柄数
```
[root@red5v5 ~]# vim /etc/security/limits.conf  文件的最后添加：
oracle         soft    nproc    10000
oracle         hard    nproc    16384
oracle         soft    nofile   52768
oracle         hard    nofile   6553
```

soft是最小值，hard是最大值，nofile是文件句柄，也就是这个用户能打开的文件数，nproc是进程数

### 增加环境变量
##### oracle用户变量
```
[root@red5v5 ~]# vim /home/oracle/.bash_profile
umask 022  
export ORACLE_BASE=/opt/oracle  
export ORACLE_HOME=/opt/oracle/product/11.2.0/dbhome_1 
export ORACLE_SID=orcl  
export LANG=en_US.UTF-8  
```
##### 添加全局变量
```

[root@red5v5 ~]# vi /etc/profile   
 
export PATH=$PATH:/opt/oracle/product/11.2.0/dbhome_1/bin 
 
```
### 上传软件
上传到/home/oracle/ 解压Oracle安装文件
```
# cd /home/oracle/  
# unzip linux*_11gR2_database_1of2.zip -d /home/oracle  
# unzip linux*_11gR2_database_2of2.zip -d /home/oracle  
# chown -R oracle:oinstall /opt/oracle /home/oracle 
```
到这里前期的准备工作全部完成啦，建议重启下系统让各项配置生效。当然也可以不重启
## 三. 安装oracle
#### 说明：
此种应答文件是安装并配置数据库，直接产生orcl数据库实例 
应答文件模版在解压后的文件夹中。 
/home/oracle/database/response/db_install.rsp 
模版中重要的配置项含义如下 
ORACLE_HOSTNAME 主机名 
ORACLE_BASE oracle的基础安装目录 
ORACLE_HOME oracle的安装目录 
INVENTORY_LOCATION oracle日志目录 
oracle.install.db.config.starterdb.globalDBName 
oracle.install.db.config.starterdb.password.ALL 

如果想知道每个选项的含义，请看：
http://blog.csdn.net/jameshadoop/article/details/48086933
### 3.1 填写安装选项
db_install.rsp文件中追加:
```
oracle.install.option=INSTALL_DB_AND_CONFIG 
ORACLE_HOSTNAME=red5v5 
UNIX_GROUP_NAME=oinstall 
INVENTORY_LOCATION=/opt/oraInventory 
SELECTED_LANGUAGES=en 
ORACLE_HOME=/opt/oracle/product/11.2.0/dbhome_1 
ORACLE_BASE=/opt/oracle 
oracle.install.db.InstallEdition=EE 
oracle.install.db.isCustomInstall=false 
oracle.install.db.DBA_GROUP=dba 
oracle.install.db.OPER_GROUP=oinstall 
oracle.install.db.CLUSTER_NODES= 
oracle.install.db.config.starterdb.type=GENERAL_PURPOSE 
oracle.install.db.config.starterdb.globalDBName=orcl 
oracle.install.db.config.starterdb.SID=orcl 
oracle.install.db.config.starterdb.characterSet=AL32UTF8 
oracle.install.db.config.starterdb.memoryLimit=503 
oracle.install.db.config.starterdb.memoryOption=true 
oracle.install.db.config.starterdb.installExampleSchemas=true 
oracle.install.db.config.starterdb.enableSecuritySettings=true 


oracle.install.db.config.starterdb.storageType=FILE_SYSTEM_STORAGE 
oracle.install.db.config.starterdb.fileSystemStorage.dataLocation=/opt/oradata 


oracle.install.db.config.starterdb.password.ALL=Oracle11g 
oracle.install.db.config.starterdb.password.SYS= 
oracle.install.db.config.starterdb.password.SYSTEM= 
oracle.install.db.config.starterdb.password.SYSMAN= 
oracle.install.db.config.starterdb.password.DBSNMP= 
```
### 3.2 执行安装命令
  安装过程中出现各种警告warning不要去理它，成功后会有：
Successfully Setup Software
```
./runInstaller -silent -force -responseFile /home/oracle/database/response/db_install.rsp
```
runInstaller参数说明：
    a. 选项-silent表示静默安装，免安装交互，大部分安装信息也不输出
    b. 选项-responseFile指定应答文件，要求用绝对路径
    c. 执行./runInstaller -help可以查看安装帮助
    d. 若忽略-silent选项，将会允许交互，对于应答文件中未设置的项可以再手工指定
    e. 若添加-noconfig选项，可以忽略应答文件中的安装类型，而仅安装数据库软件
### 3.3安装后需要执行脚本
root权限执行安装后的配置脚本 
```
[oracle@red5v5 /]# su - root  
[root@red5v5 /]# sh /opt/oraInventory/orainstRoot.sh  
[root@red5v5 /]# sh /opt/oracle/product/11.2.0/dbhome_1/root.sh  
```
db_install.rsp配置已经产生了数据库实例orcl,所以不需要再配置数据库啦


## 四. 启动数据库 
启动监听 

```
[oracle@red5v5 /]# lsnrctl start 
```
查看监听状态 
```
[oracle@red5v5 /]# lsnrctl status 
```
停止监听 
```
[oracle@red5v5 /]# lsnrctl stop 
```
此时安装结束，可以用scott用户测试。
## 附：
由于系统重启之后，数据库服务并不是自动启动，所以要手工启动，方法：
### 1.linux下开启oracle 
1.1启动服务
```
su - oracle 
sqlplus /nolog 
conn /as sysdba 
startup

exit
```
1.2.启动监听
```
lsnrctl start

exit 
```

### 2. linux下关闭oracle 
关闭数据库服务
```
su - oracle 

sqlplus /nolog 

conn /as sysdba 

shutdown immediate 


exit 
```
关闭监听
```

lsnrctl stop 

exit 
```
