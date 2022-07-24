### Ambari功能
- 提供跨任意数量的主机安装Hadoop服务分步向导
- 处理集群的Hadoop服务配置
- 提供集中管理，用于在整个集群中启动、停止和重新配置Hadoop服务
- 提供系统警报、监控Hadoop集群的运行状态
### Ambari架构
ambari使用的是Master/Slaves 架构（由一个Ambari server 和多个Agent 组成），通过一个Server主进程实现集群的管理和操作命令的发送，而具体的管理动作是由每台目标主机的Agent进行执行。
[HDP支持的ambari 版本](https://supportmatrix.cloudera.com/#Hortonworks)  
[安装参考文档1](https://blog.csdn.net/q495673918/article/details/121626972)
[参考文档2](https://blog.csdn.net/weixin_40461486/article/details/120437682))

## 准备工作
### 机器
设置ssh 免密登录 
- 上传私钥文件到/root/.ssh 
- 设置权限 chmod 700 /root/.ssh/id_rsa
- 重启ssh systemctl restart sshd
- 测试连接 ssh 用户名@ip
修改本机名 
```shell
hostnamectl set-hostname node2
```
修改后查看本机名
```shell
hostname -f
```
修改host 文件 /etc/hosts
```
::1	localhost	localhost.localdomain	localhost6	localhost6.localdomain6
127.0.0.1	localhost	localhost.localdomain	localhost4	localhost4.localdomain4
 
ip1 node1
ip2 node2
```
修改完成后用ping 测试 
```shell
ping node1
```
禁用selinux
```shell
# 临时关闭（机器重启后会再开）
setenforce 0
# 永久关闭（设置后需重启才能生效）
vi /etc/selinux/config # 然后将 SELINUX=enforcing 改为 SELINUX=disabled

```
### 安装包
**ambari 2.7.4安装包 HDP3.1.4安装包 下载后上传到node1（使用迅雷下载）**
HDP是hortonworks的软件栈，里面包含了hadoop生态系统的所有软件项目。
HDP-UTILS是工具类库。

http://public-repo-1.hortonworks.com/ambari/centos7/2.x/updates/2.7.4.0/ambari-2.7.4.0-centos7.tar.gz

http://public-repo-1.hortonworks.com/HDP/centos7/3.x/updates/3.1.4.0/HDP-3.1.4.0-centos7-rpm.tar.gz

http://public-repo-1.hortonworks.com/HDP-UTILS-1.1.0.22/repos/centos7/HDP-UTILS-1.1.0.22-centos7.tar.gz

https://archive.cloudera.com/p/HDP-GPL/3.x/3.1.4.0/centos7/HDP-GPL-3.1.4.0-centos7-gpl.tar.gz

### 环境 
#### 每台机器都安装 java 8
1. 安装jdk
```shell
rpm -ivh jdk-8u161-linux-x64.rpm
```
2. 配置环境变量
```shell
➜  /data which java
/usr/bin/java
➜  /data ls -lrt /usr/bin/java
lrwxrwxrwx 1 root root 22 Jul 24 14:21 /usr/bin/java -> /etc/alternatives/java
➜  /data ls -lrt /etc/alternatives/java
lrwxrwxrwx 1 root root 35 Jul 24 14:21 /etc/alternatives/java -> /usr/java/jdk1.8.0_161/jre/bin/java

```
后边显示的/usr/java/jdk1.8.0_161就是完整的安装路径
设置jdk 环境变量
```
vim /etc/profile
#set java 环境变量
export JAVA_HOME=/usr/java/jdk1.8.0_161
export PATH=$JAVA_HOME/bin:$PATH

```
使环境变量生效
```shell
source /etc/profile
#再检查环境变量
echo $JAVA_HOME
```
#### node1下载安装mysql 5.7
1. 下载mysql5.7的rpm包
```shell
wget https://dev.mysql.com/get/mysql57-community-release-el7-8.noarch.rpm
```
2. 安装rpm文件
```shell
rpm -ivh mysql57-community-release-el7-8.noarch.rpm
#安装成功后/etc/yum.repos.d/目录下会增加两个文件
```
3. 安装mysql 服务
```shell
yum -y install mysql-server

#安装出现问题
The GPG keys listed for the "MySQL 5.7 Community Server" repository are already installed but they are not correct for this package.
Check that the correct key URLs are configured for this repository.

# 解决方案
rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022

```
4. 启动mysql
```shell
systemctl start mysqld
```
5. 修改mysql 临时密码 获取mysql 临时密码 
```shell
grep 'temporary password' /var/log/mysqld.log
# 登录
mysql -u root -p 
# 为了可以设置简单密码  
set global validate_password_policy=LOW;

set global validate_password_length=4;

# 创建密码为root123
ALTER USER  'root'@'localhost'  IDENTIFIED BY 'root123';
```
6. 新增ambari 用户并增加权限
```shell
CREATE USER 'ambari'@'%' IDENTIFIED BY 'ambari';
GRANT ALL PRIVILEGES ON *.* TO 'ambari'@'%';
CREATE USER 'ambari'@'localhost' IDENTIFIED BY 'ambari';
GRANT ALL PRIVILEGES ON *.* TO 'ambari'@'localhost';
CREATE USER 'ambari'@'node1' IDENTIFIED BY 'ambari';
GRANT ALL PRIVILEGES ON *.* TO 'ambari'@'node1';  //本地主机名
FLUSH PRIVILEGES;
```
 删除用户
```shell
Delete FROM user Where User='your_user' and Host='your_host';
FLUSH PRIVILEGES;
```
7. 使用ambari 用户登录，并创建数据库
```
mysql -uambari -pambari
CREATE DATABASE ambari;
exit;
```
- 设置时钟同步(所有机器)
```

```
### 修改yum 源，实现离线安装
#### node1安装httpd服务
```shell
#安装
yum -y install httpd 
#启动
systemctl start httpd 
#查看启动状态
systemctl status httpd
#设置开机启动 
systemctl enable httpd
```
#### 把上传的ambari，HDP，HDP-UTILS都解压到/var/www/html (thhpd 创建的)这个目录下
```shell
tar -zxvf ambari-2.7.4.0-centos7.tar.gz -C /var/www/html/
tar -zxvf HDP-3.1.4.0-centos7-rpm.tar.gz -C /var/www/html/
tar -zxvf HDP-UTILS-1.1.0.22-centos7.tar.gz -C /var/www/html/
tar -zxvf HDP-GPL-3.1.4.0-centos7-gpl.tar.gz -C /var/www/html/
```
#### 配置基础数据源
- 在/etc/yum.repos.d/目录下创建ambari.repo
```shell
[ambari-2.7.4.0]
name=ambari
baseurl=http://node1/ambari/centos7/2.7.4.0-118
gpgkey=http://node1/ambari/centos7/2.7.4.0-118/RPM-GPG-KEY/RPM-GPG-KEY-Jenkins
gpgcheck=1
enabled=1
priority=1
 
[HDP-3.1.4.0]
name=HDP Version - HDP-3.1.4.0
baseurl=http://node1/HDP/centos7/3.1.4.0-315/
gpgcheck=1
gpgkey=http://node1/HDP/centos7/3.1.4.0-315/RPM-GPG-KEY/RPM-GPG-KEY-Jenkins
enabled=1
priority=1
 
[HDP-UTILS-1.1.0.22]
name=HDP-UTILS Version - HDP-UTILS-1.1.0.22
baseurl=http://node1/HDP-UTILS/centos7/1.1.0.22/
gpgcheck=1
gpgkey=http://node1/HDP-UTILS/centos7/1.1.0.22/RPM-GPG-KEY/RPM-GPG-KEY-Jenkins
enabled=1
priority=1

[HDP-HDP-GPL-3.1.4.0-315]
name=HDP-GPL Version - HDP-GPL-3.1.4.0-315
baseurl=http://node1/HDP-GPL/centos7/3.1.4.0-315/
gpgcheck=1
gpgkey=http://node1/DP-GPL/centos7/3.1.4.0-315/RPM-GPG-KEY/RPM-GPG-KEY-Jenkins
enabled=1
priority=1
```
- 安装createrepo
```shell
yum install yum-utils createrepo yum-plugin-priorities -y
```
- 生成本地源（软件仓库）元数据，即为存放本地特定位置的众多rpm包建立索引，描述各包所需依赖信息，并形成元数据。
```shell
createrepo  ./
```
- 将前面创建的ambari.repo复制到其他机器
```shell
scp ambari.repo node2:$PWD
```
- 清除yum 缓存（每台机器）
```shell
yum clean all
yum makecache
yum repolist
```

## 安装ambari-server
### 安装ambari-server
在node1 执行
```
yum -y install ambari-server
```
### 安装jdbc驱动
在后边会使用到
```
yum install -y mysql-connector-java
```
### ambari-server 设置
```
ambari-server setup

Using python  /usr/bin/python
Setup ambari-server
Checking SELinux...
SELinux status is 'disabled'
Customize user account for ambari-server daemon [y/n] (n)? y
Enter user account for ambari-server daemon (root):root
Adjusting ambari-server permissions and ownership...
Checking firewall status...
Checking JDK...
[1] Oracle JDK 1.8 + Java Cryptography Extension (JCE) Policy Files 8
[2] Custom JDK
==============================================================================
Enter choice (1): 2
WARNING: JDK must be installed on all hosts and JAVA_HOME must be valid on all hosts.
WARNING: JCE Policy files are required for configuring Kerberos security. If you plan to use Kerberos,please make sure JCE Unlimited Strength Jurisdiction Policy Files are valid on all hosts.
Path to JAVA_HOME: /usr/java/jdk1.8.0_161 # 设置jdk 环境变量地址
Validating JDK on Ambari Server...done.
Check JDK version for Ambari Server...
JDK version found: 8
Minimum JDK version is 8 for Ambari. Skipping to setup different JDK for Ambari Server.
Checking GPL software agreement...
GPL License for LZO: https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html
Enable Ambari Server to download and install GPL Licensed LZO packages [y/n] (n)? y
Completing setup...
Configuring database...
Enter advanced database configuration [y/n] (n)? #使用自定义的数据库
Configuring database...
==============================================================================
Choose one of the following options:
[1] - PostgreSQL (Embedded)
[2] - Oracle
[3] - MySQL / MariaDB
[4] - PostgreSQL
[5] - Microsoft SQL Server (Tech Preview)
[6] - SQL Anywhere
[7] - BDB
==============================================================================
Enter choice (1): 3 # 使用mysql
Hostname (localhost):  #直接回车使用 默认的localhost
Port (3306): y #直接回车使用默认的端口3306
Invalid port.
Port (3306): y
Invalid port.
Port (3306): y
Invalid port.
Port (3306): y
Invalid port.
Port (3306): 3306
Database name (ambari):  #直接回车使用默认的数据库名
Username (ambari):    #直接回车使用默认的用户名ambari 用户名在上边已经创建
Enter Database Password (bigdata): #输入密码 ambari
Invalid characters in password. Use only alphanumeric or _ or - characters
Enter Database Password (bigdata): #输入密码 ambari
Re-enter password: #输入密码 ambari
Configuring ambari database...
Should ambari use existing default jdbc /usr/share/java/mysql-connector-java.jar [y/n] (y)? y #使用存在默认的jdbc
Configuring remote database connection properties...
WARNING: Before starting Ambari Server, you must run the following DDL directly from the database shell to create the schema: /var/lib/ambari-server/resources/Ambari-DDL-MySQL-CREATE.sql
Proceed with configuring remote database connection properties [y/n] (y)? y
Extracting system views...
....ambari-admin-2.7.4.0.118.jar

Ambari repo file doesn't contain latest json url, skipping repoinfos modification
Adjusting ambari-server permissions and ownership...
Ambari Server 'setup' completed successfully. #安装成功

```

设置失误可以重新操作
```shell
ambari-server reset
# 然后再执行sudo ambari-server setup
```
### 初始化数据库
```
mysql -uambari -pambari
use ambari;
source /var/lib/ambari-server/resources/Ambari-DDL-MySQL-CREATE.sql;
```

### 启动ambari
服务启动成功会监听8080端口，失败的话可以查看日志 /var/log/ambari-server/ambari-server.log
```shell
➜ ambari-server start                                
Using python  /usr/bin/python
Starting ambari-server
Ambari Server running with administrator privileges.
Organizing resource files at /var/lib/ambari-server/resources...
Ambari database consistency check started...
Server PID at: /var/run/ambari-server/ambari-server.pid
Server out at: /var/log/ambari-server/ambari-server.out
Server log at: /var/log/ambari-server/ambari-server.log
Waiting for server start.....................
Server started listening on 8080

DB configs consistency check: no errors and warnings were found.
Ambari Server 'start' completed successfully.

```
## 安装hadoop集群
访问ip:8080 默认账号密码admin/admin
### 安装
点击LAUNCH INSTALL WIZARD
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1658655487349.png)

### 集群命名
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1658655540830.png)
### 选择hdp版本
修改使用本地源，除其他的选项，只保留redhat7
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1658658047197.png)

### 添加节点信息，使用主节点形式，将主节点私钥填入，用于集群免密登录
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1658658418290.png)

### 等待host安装验证
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1658658524615.png)
- **node2 注册失败**
解决方法 在/etc/amabri-agent/conf/ambari-agent.in  文件的[security]加上force_https_protocol=PROTOCOL_TLSv1_2
[参考地址]([ambari agents cannot reach ambari-server after cha... - Cloudera Community - 193251](https://community.cloudera.com/t5/Support-Questions/ambari-agents-cannot-reach-ambari-server-after-changing/m-p/193251))
```shell
[security]
keysdir=/var/lib/ambari-agent/keys
server_crt=ca.crt
passphrase_env_var_name=AMBARI_PASSPHRASE
ssl_verify_cert=0
credential_lib_dir=/var/lib/ambari-agent/cred/lib
force_https_protocol=PROTOCOL_TLSv1_2
credential_conf_dir=/var/lib/ambari-agent/cred/conf
credential_shell_cmd=org.apache.hadoop.security.alias.CredentialShell

```
- 安装失败
ambari-server错误内容
```
Unable to lookup the cluster by ID; assuming that there is no cluster and therefore no configs for this execution command: Cluster not found, clusterName=clusterID=-1
```
ambari-agnet 错误内容
```
agent:hostname_script configuration not defined thus read hostname 'node2' using socket.getfqdn().

Failed to connect to https://node1:8440/ca due to [Errno 110] Connection timed out
```
 开放 node1 8440 8441端口
 ### 选择安装的框架和分配资源