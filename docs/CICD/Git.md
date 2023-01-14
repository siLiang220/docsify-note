## 安装git
### 1.下载git
```
https://github.com/git/git/releases/tag/v2.39.0
```
### 2. 解压gz
```shell
cd   /opt 			  #进入到/opt的目录下
mkdir git        			  #新建git目录
将上面获取到的gz包放入git目录中
tar -zxvf git-2.39.0.tar.gz   #解压gz包
```
### 3. 可能提前安装的依赖
```shell
yum install curl-devel expat-devel openssl-devel zlib-devel gcc-c++ 
yum install perl-ExtUtils-MakeMaker automake autoconf libtool make
```
### 4.配置git的环境变量
```shell
make prefix=/opt/git all      
make prefix=/opt/git install
export PATH=/opt/git/bin:$PATH
```

### 5.刷新profile文件
```shell
cd /opt/git/git-2.39.0

source /etc/profile
```

### 6.验证git
```shell
git --version
```


