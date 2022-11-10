
## Docker 容器运行模式
docker 容器进程有两种运行模式
1.  Foreground 前台模式（默认）
```
docker run 
或
docker run -d=false
```
对于foreground容器，由于其只是在开发调试过程中短期运行，其用户数据并无保留的必要，因而可以在容器启动时设置--rm选项，这样在容器退出时就能够自动清理容器内部的文件系统，--rm 不能清理Detached模式

2.  Detached 后台模式
```
docker run -d
或
docker run -d=true
```

## Docker run
[Docker 命令详解（run篇） - 海兵的正义 - 博客园 (cnblogs.com)](https://www.cnblogs.com/shijunjie/p/10488603.html)

```shell
## 从远程服务器拉取(下载)取镜像
docker pull nginx # 不指定版本相当于 docker pull nginx:latest 通常约定latest为最新版本

## 查看本地拥有的镜像
docker images

## 将镜像运行成一个真正在运行的容器(虚拟机)
docker run -d -p 81:80 nginx: #(镜像名称:版本号或用nginx镜像的IMAGE ID)，-d后台运行而不阻塞shell指令窗口,-p指定内外端口映射，外部81端口映射内部80端口;如果直接run某个镜像，而本地没又该镜像，则会自动尝试从远程服务器pull下载然后继续run

## 查看正在运行的容器
docker ps #-a 查看所有容器，包括没运行的

## 查看容器日志
docker logs -f 容器id

## 修改nginx容器
docker exec -it nginx容器id bash #(只要开头几位就好，能区分其他即可),bash是通过bash方式进入容器，能看到输入指令的开头变成root@容器ID:/# 光标位置

## 退出docker容器
exit

## 强制删除容器
docker rm -f 容器ID #(同样只要开头几位能识别就好了)

## 将本地的容器提提提交到本地形成一个指定名称的镜像，该镜像保留容器中所作的修改
docker commit 容器ID m1 #容器ID同样只要开头几位；m1指定镜像的名字，例如这里取名m1

## 编写Dockerfile文件
vim Dockerfile

## 编写一个示例的文件
vim index.html #随便写写就好了

## 通过docker build使用Dockerfile去构建一个定制的镜像
docker build -t m2 . # .点点表示指定当前目录下的Dockerfile文件去构建

## 将镜像保存到tar文件
docker save m2 > 1.tar # docker save [镜像名称] > tar文件名.tar

## 删除镜像
docker rmi m2 # 可以通过名称也可以通过IMAGE ID删除，如果镜像被运行成容器就删除不了

## 读取tar文件恢复一份镜像
docker load < 1.tar #这里运行后，发现images又重新出现了之前Dockerfile创建的m2镜像

## push操作需要自己去dockerhub或者其他官方仓库注册
## docker run -d -p 88:80 --name ，其中--name 指定容器运行后的名字NAMES，默认随机单词拼写的名字
## docker run -d -p 88:80 --name -v ，-v映射文件，比如可以把大哥前目录映射到内部的/usr/share/nginx/html
docker run -d -p 88:80 --name mynginx -v 'pwd':/usr/share/nginx/html nginx:latest# 这样就可以将一些静态文件放在外面，外面修改文件(因为是映射的)，里面的文件就会跟着变化
## 文件的映射也可用于一些数据的保存，比如MySQL的data目录可以映射到外面，防止数据丢失
## 最后接的参数就是镜像的名字，后面的版本如果知道具体的版本号就最好不要省略，省略就是latest最新版，而镜像如果一直在全新的构建的话，latest会不断的更新，如果有一个已知正在使用的稳定版本，最好指定这个版本
```

## Dockerfile

### Dockerfile常用指令

[Dockerfile reference | Docker Documentation](https://docs.docker.com/engine/reference/builder/)

Dockerfile其内部**包含了一条条的指令**，**每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建**。

- **FROM** :指明构建的镜像来自于那个基础镜像
```shell
FROM centos:6
```

- **MAINTAINER**：指明镜像维护者
```
MAINTAINER Edison zhou <xxxx.gmail.com>
```

- **RUN**：构建镜像时运行的shell命令
```shell
## cmd 格式
RUN yum install httpd
## exec 格式
RUN ["yum", "install", "httpd"]
```

- **CMD**：启动容器时执行的shell命令，用于运行程序，指定整个镜像运行起来时执行的脚本(容器真正运行时执行)，并且这个脚本运行完后，整个容器的生命周期也就结束了，所以通常可以指定一些阻塞式的脚本
```shell
CMD /usr/sbin/sshd -D
```

- **EXPOSE**：声明容器运行的服务端口
```shell
EXPOSE 80 443
```
 
 - **ENV**：设置环境内的环境变量
```shell
ENV MYSQL_ROOT_PASSWORD 123456
ENV JAVA_HOME /usr/local/jdk1.8.0_45
```

- **ADD：** 将本地文件添加到容器中，tar类型的文件会自动解压（网络压缩资源不会被解压），可以访问网络资源，类似wget
```Shell
ADD html.tar.gz /var/www/html
ADD https://xxx.com/html.tar.gz /var/www/html
```

- **COPY**：同ADD，但不支持解压
```shell
COPY ./start.sh /start.sh
```

- **VOLUME** ：定义匿名数据卷。在启动容器时忘记挂载数据卷，会自动挂载到匿名卷。
 volume只是指定了一个目录，用以在用户忘记启动时指定-v参数也可以保证容器的正常运行。比如mysql，你不能说用户启动时没有指定-v，然后删了容器，就把mysql的数据文件都删了，那样生产上是会出大事故的，所以mysql的dockerfile里面就需要配置volume，这样即使用户没有指定-v，容器被删后也不会导致数据文件都不在了。还是可以恢复的。
```bash
VOLUME ["<路径1>", "<路径2>"...] 
VOLUME ["/var/lib/mysql"]
```
	PS:一般不会在Dockerfile中用到，docker run -v指定


- **USER**：为RUN CMD和ENTRYPOINT执行Shell命令指定运行用户
```bash
USER <user>[:<usergroup>]
USER <UID>[:<UID>]
USER xxx
```

- **WORKDIR**： WORKDIR 命令为后续的 RUN、CMD、COPY、ADD 等命令配置工作目录。在设置了 WORKDIR 命令后，接下来的 COPY 和 ADD 命令中的相对路径就是相对于 WORKDIR 指定的路径。比如我们在 Dockerfile 中添加下面的命令：
```bash
WORKDIR /app
COPY checkredis.py .
```

- **ARG**： 在构建镜像时，指定一些参数，与ENV不同的是，ENV在构建和运行时都生效，是系统环境变量；而ARG是构建参数，只有在构建时生效，前面说过CMD是运行时才生效,如果ARG B=10那么CMD echo $B是无法打印出10的，找不到参数，打印出空行。可以通过ARG B=10而ENV A $B而CMD echo $A把ARG指定的参数B打印出来。**构建指的是docker build**。因为ARG要被用到终究还是得靠CMD，而ARG指定的值可以当作默认值，那么使用 docker build -t test（这个是镜像名） --build-arg B=12则会将里面的B设置为12，而不指定B时，内部的参数则为默认的10.
```bash
    ARG <name>[=<default value>]
```

-  **HEALTHCHECK**：检查容器健康状态的配置
- **SHELL**：指定基于那种shell，一般Linux 默认是/bin/sh

```bash
FROM alpine
WORKDIR /app
COPY src/ /app # 复制宿主机src/目录下所有内容到容器的/app目录下
RUN echo 321 >> 1.txt # 此时目录即WORKDIR指定的 /app目录
CMD tail -f 1.txt
```

### Dockerfile 示例
```bash
docker pull node:16.15.0

## 编写Dockerfile
vim Dockerfile

FROM NODE:16.15.0
RUN git clone https://gitee.com/mirschao/webserver-vue.git
WORKDIR webserver-vue
RUN npm install
EXPOSE 8080
CMD ["npm","run","serve"]

## 构建镜像
docker build -t webswever-vue:v1.0.0 .

## 运行镜像
docker run -itd --name webserver-vue-project -p 8090:8080 webserver-vue:v1.0.0
```

多阶段构建镜像
```bash
git clone https://gitee.com/mirschao/webserver-vue.git

## 编写Dockerfile
vim Dockerfile

FROM NODE:16.15.0
COPY ./ /app
WORKDIR /app
RUN npm install && npm run build

FROM nginx:1.21
RUN mkdir /app
## --from=0将构建的第一个镜像产物服务 到/app，
## 也可以使用as别名 FROM NODE:16.15.0 as basic
## COPY basic:/app/dist /app
COPY --from=0 /app/dist /app
COPY nginx.conf /ect/nginx/nginx.conf

```