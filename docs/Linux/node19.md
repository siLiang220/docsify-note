## 1.下载node 安装包
```sh
wget https://nodejs.org/download/release/v19.9.0/node-v19.9.0-linux-x64.tar.xz
```

## 2. 解压
```sh
xz -d node-v19.9.0.tar.xz
tar -xvf node-v19.9.0-linux-x64.tar
```

## 3.将文件夹移动到到local
```sh
mv node-v19.9.0-linux-x64 /usr/local
```

## 4.重命名文件夹
```sh
mv /usr/local/node-v19.9.0-linux-x64 /usr/local/node
```

## 5. 修改profile添加环境变量
```sh
vim /etc/profile
```
添加
```bash
export NODE_HOME=/usr/local/node
export PATH=$NODE_HOME/bin:$PATH
```

## 6. 刷新配置文件
```bash
source /etc/profile
```

## 安装过程遇到的问题
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20230617130350.png)

解决
```sh
export NODE_HOME=/usr/local/node
export PATH=$NODE_HOME/bin:$PATH
cd /root
# 编译安装
wget http://ftp.gnu.org/gnu/glibc/glibc-2.28.tar.gz
tar xf glibc-2.28.tar.gz 
cd glibc-2.28/ && mkdir build  && cd build
../configure --prefix=/usr --disable-profile --enable-add-ons --with-headers=/usr/include --with-3binutils=/usr/bin


# tips: 如果没报错可不用处理这里的步骤...
# ***********************************************************************
# 这一步提示如下错误
# configure: error: 
# *** These critical programs are missing or too old: compiler
# *** Check the INSTALL file for required versions.

# 解决：  升级gcc与make
# 1. 安装GCC-8
yum install -y devtoolset-8-gcc devtoolset-8-gcc-c++ devtoolset-8-binutils
# 如果提示以下报错说明系统没有可用的devtoolset-8软件包，需要安装devtoolset-8后重新安装GCC-8,如果没有则跳过
# No package devtoolset-8-gcc available.
# No package devtoolset-8-gcc-c++ available.
# No package devtoolset-8-binutils available.
# Error: Nothing to do
# 1.1 安装devtoolset-8
sudo yum install -y centos-release-scl
sudo yum install -y devtoolset-8
# 1.2 启用devtoolset-8
scl enable devtoolset-8 bash

# 设置环境变量
echo "source /opt/rh/devtoolset-8/enable" >> /etc/profile
source /etc/profile

# 2. 升级 make
wget https://ftp.gnu.org/gnu/make/make-4.3.tar.gz
tar -xzvf make-4.3.tar.gz && cd make-4.3/
# 安装到指定目录
./configure  --prefix=/usr/local/make
make && make install
# 创建软链接
cd /usr/bin/ && mv make make.bak
ln -sv /usr/local/make/bin/make /usr/bin/make

# 继续编译 glibc   -- 进入刚才安装`glibc-2.28/build`的目录
cd /root/glibc-2.28/build
../configure --prefix=/usr --disable-profile --enable-add-ons --with-headers=/usr/include --with-binutils=/usr/bin
# ***********************************************************************


make && make install
# 日志最后会出现如下问题
# primary library!
# make[1]: *** [Makefile:111: install] Error 1
# make[1]: Leaving directory '/root/glibc-2.28'
# make: *** [Makefile:12: install] Error 2

# 再次查看系统内安装的glibc版本
strings /lib64/libc.so.6 |grep GLIBC_

# 测试
node -v
npm -v

# 然后会报错如下：
# [root@master build]# node -v
# node: /lib64/libstdc++.so.6: version `CXXABI_1.3.9' not found (required by node)
# node: /lib64/libstdc++.so.6: version `GLIBCXX_3.4.20' not found (required by node)
# node: /lib64/libstdc++.so.6: version `GLIBCXX_3.4.21' not found (required by node)

# 解决
yum install libstdc++.so.6 -y
# 查看动态链接库 -- 发现并没有需要的1.3.9
strings /usr/lib/libstdc++.so.6 | grep 'CXXABI'
# 下载需要的版本库，之后软连接到运行系统上
wget http://ftp.de.debian.org/debian/pool/main/g/gcc-8/libstdc++6_8.3.0-6_amd64.deb
ar -x libstdc++6_8.3.0-6_amd64.deb
tar -xvf data.tar.xz
cp usr/lib/x86_64-linux-gnu/libstdc++.so.6.0.25 /usr/lib64/
find / -name "libstdc++*"
# 删除低版本库的软连接
rm -rf /usr/lib64/libstdc++.so.6
ll /usr/lib64/libstd*
ln -s /usr/lib64/libstdc++.so.6.0.25 /usr/lib64/libstdc++.so.6

# 检验
node -v
npm -v

```