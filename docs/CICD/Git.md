## 一、安装git
### 1.下载git
```
https://github.com/git/git/releases/tag/v2.39.0
```
### 2. 解压gz
```shell
cd /opt 			  #进入到/opt的目录下
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



## 二、git 分支误删除找回记录

### 1.git 的垃圾回收机制
在git 中删除一个分支时，实际上他并没有真正的删除，他只是被git标记为了删除，这是因为git有一种回收机制，会定期清理不需要的对象，包括标记为“已删除”的分支。

### 2.使用git reflog命令
Git reflog 命令可以列出所有的 Git 引用（如分支、标签等）的历史记录，包括已经被删除的引用。因此，使用 Git reflog 命令可以找到之前删除的分支，并恢复它。

具体步骤
1. 进入你的 Git 仓库目录，并打开终端（MacOS 或 Linux）或 Git Shell（Windows）。
2. 在终端或 Git Shell 中，输入以下命令，查看引用历史记录：
```shell
git reflog
```
3. 找到删除的分支最后一个commit id
```shell
0c4dad71 HEAD@{6}: commit: 修改代码命名
ed185b30 HEAD@{7}: Branch: renamed refs/heads/0426_workorder to refs/heads/设备安装记录
ed185b30 HEAD@{9}: commit: 设备安装记录功能
dab5b07a HEAD@{10}: commit: 设备安装记录
030b2ce7 HEAD@{11}: checkout: moving from 0517_产品二维码 to 0426_workorder
3499b628 (0523_设备安装记录) HEAD@{12}: checkout: moving from 0426_workorder to 0517_产品二维码
030b2ce7 HEAD@{13}: commit (merge): Merge branch 'master' of http://xxxxx.com
695189c6 HEAD@{14}: checkout: moving from 0523_设备安装记录 to 0426_workorder
3499b628 (0523_设备安装记录) HEAD@{15}: checkout: moving from 0517_产品二维码 to 0523_设备安装记录
3499b628 (0523_设备安装记录) HEAD@{16}: pull --no-stat -v --progress master master: Fast-forward
d19484dd (origin/0517_产品二维码) HEAD@{17}: commit: 修改二维码类型为枚举
566feaea HEAD@{18}: commit: 修改产品和生成二维码接口
ec745662 HEAD@{19}: commit: 修改产品和生成二维码接口
46680c02 HEAD@{20}: commit: 增加生成二维码
a33110a6 HEAD@{21}: pull --no-stat -v --progress master master: Fast-forward
aa37e451 HEAD@{22}: commit: 微信验证文件
918816a8 HEAD@{23}: commit: 对接微信公众号授权
67157733 HEAD@{24}: commit: 对接微信公众号授权
a7fc2d86 HEAD@{25}: commit: 增加二维码，销售商接口
79c1f33a HEAD@{26}: pull --no-stat -v --progress master master: Merge made by the 'ort' strategy.
e57fd1c8 HEAD@{27}: commit: 创建销售实体表
b822500e HEAD@{28}: checkout: moving from 0426_workorder to 0517_产品二维码
```

4. 然后使用以下命令来恢复分支
```shell
git branch <brach-name> <commit-id>
```


## 三、git Commit 规范

### 1.commit message格式

```text
<type>(<scope>): <subject>
```

**type(必须)**

用于说明git commit 的类别

- feat：新功能（feature）

- fix/to：修复bug，可以是QA发现的BUG，也可以是研发自己发现的BUG。
	-  fix：产生diff并自动修复此问题。适合于一次提交直接修复问题
	- to：只产生diff不自动修复此问题。适合于多次提交。最终修复问题提交时使用fix

- docs：添加/更新文档（documentation）。

- style：格式更新UI样式（不影响代码运行的变动）。

- format：格式化代码

- init：初始提交或初始化代码 

- refactor：重构（即不是新增功能，也不是修改bug的代码变动）。

- perf：优化相关，比如提升性能、体验。

- test：增加测试。

- publish：发布新版本

- tag：发布版本/添加标签 

- patch：添加重要补丁

- file：添加文件 

- git：添加或修改gitignore文件

- chore：构建过程或辅助工具的变动。

- revert：回滚到上一个版本。

- ci：对CI配置文件和脚本的更改。

- config：修改配置文件（配置） 

- merge：代码合并。

- sync：同步主线或分支的Bug。

**scope(可选)**

scope用于说明 commit 影响的范围，比如数据层、控制层、视图层等等，视项目不同而不同。

例如在Angular，可以是location，browser，compile，compile，rootScope， ngHref，ngClick，ngView等。如果你的修改影响了不止一个scope，你可以使用*代替。

**subject(必须)**

subject是commit目的的简短描述，不超过50个字符。

建议使用中文（感觉中国人用中文描述问题能更清楚一些）。

- 结尾不加句号或其他标点符号。
- 根据以上规范git commit message将是如下的格式：**:是英文的的冒号， 后边需要带有一个空格，这个是不能省略的**

```text
fix(DAO):用户查询缺少username属性 
feat(Controller):用户查询接口开发
```

以上就是我们梳理的git commit规范，那么我们这样规范git commit到底有哪些好处呢？

- 便于程序员对提交历史进行追溯，了解发生了什么情况。
- 一旦约束了commit message，意味着我们将慎重的进行每一次提交，不能再一股脑的把各种各样的改动都放在一个git commit里面，这样一来整个代码改动的历史也将更加清晰。
- 格式化的commit message才可以用于自动化输出Change log。

## 四、git stash 使用