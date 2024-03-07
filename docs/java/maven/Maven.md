## 梳理maven工程属性

**GAV遵循一下规则：**

1.  **GroupID 格式**：com.{公司/BU }.业务线.[子业务线]，最多 4 级。

- 说明：{公司/BU} 例如：`alibaba/taobao/tmall/aliexpress` 等 BU 一级；子业务线可选。
- 正例：`com.taobao.tddl` 或 `com.alibaba.sourcing.multilang`  `com.atguigu.java`

2.  **ArtifactID 格式**：产品线名-模块名。语义不重复不遗漏，先到仓库中心去查证一下。

- 正例：`tc-client / uic-api / tair-tool / bookstore`

3. **Version版本号格式推荐**：主版本号.次版本号.修订号 1.0.0
  例如： 初始→1.0.0  修改bug → 1.0.1  功能调整 → 1.1.1等
- 主版本号：当做了不兼容的 API 修改，或者增加了能改变产品方向的新功能。
- 次版本号：当做了向下兼容的功能性新增（新增类、接口等）。
- 修订号：修复 bug，没有修改方法签名的功能加强，保持 API 兼容性。

4. **Packaging定义规则：**
  指示将项目打包为什么类型的文件，idea根据packaging值，识别mave n项目类型！

  - packaging 属性为 jar（默认值），代表普通的Java工程，打包以后是.jar结尾的文件。
  - packaging 属性为 war，代表Java的web工程，打包以后.war结尾的文件。
  - packaging 属性为 pom，代表不会打包，用来做继承的父工程。

## idea创建Maven javaEE项目
### 使用idea插件JBLJavaToWeb创建

创建maven项目后右`JBLJavaToWeb`插件快速补全web项目

### Maven 项目结构说明
```xml
|-- pom.xml                               # Maven 项目管理文件 
|-- src
    |-- main                              # 项目主要代码
    |   |-- java                          # Java 源代码目录
    |   |   `-- com/example/myapp         # 开发者代码主目录
    |   |       |-- controller            # 存放 Controller 层代码的目录
    |   |       |-- service               # 存放 Service 层代码的目录
    |   |       |-- dao                   # 存放 DAO 层代码的目录
    |   |       `-- model                 # 存放数据模型的目录
    |   |-- resources                     # 资源目录，存放配置文件、静态资源等
    |   |   |-- log4j.properties          # 日志配置文件
    |   |   |-- spring-mybatis.xml        # Spring Mybatis 配置文件
    |   |   `-- static                    # 存放静态资源的目录
    |   |       |-- css                   # 存放 CSS 文件的目录
    |   |       |-- js                    # 存放 JavaScript 文件的目录
    |   |       `-- images                # 存放图片资源的目录
    |   `-- webapp                        # 存放 WEB 相关配置和资源
    |       |-- WEB-INF                   # 存放 WEB 应用配置文件
    |       |   |-- web.xml               # Web 应用的部署描述文件
    |       |   `-- classes               # 存放编译后的 class 文件
    |       `-- index.html                # Web 应用入口页面
    `-- test                              # 项目测试代码
        |-- java                          # 单元测试目录
        `-- resources                     # 测试资源目录
```

## Maven 核心功能
### 依赖管理和配置

**依赖传递**：指的是当一个模块或库 A 依赖于另一个模块或库 B，而 B 又依赖于模块或库 C，那么 A 会间接依赖于 C。这种依赖传递结构可以形成一个依赖树。当我们引入一个库或框架时，构建工具（如 Maven、Gradle）会自动解析和加载其所有的直接和间接依赖，确保这些依赖都可用。仅`complie dependenices`作用域。

依赖传递的作用是：

1. 减少重复依赖：当多个项目依赖同一个库时，Maven 可以自动下载并且只下载一次该库。这样可以减少项目的构建时间和磁盘空间。
2. 自动管理依赖: Maven 可以自动管理依赖项，使用依赖传递，简化了依赖项的管理，使项目构建更加可靠和一致。
3. 确保依赖版本正确性：通过依赖传递的依赖，之间都不会存在版本兼容性问题，确实依赖的版本正确性！

## 依赖传递和冲突

发现依赖已经存在（重复依赖）会终止依赖传递，避免循环依赖和重复依赖的问题

maven自动解决依赖冲突问题能力，会按照自己的原则，进行重复依赖选择。同时也提供了手动解决的冲突的方式，不过不推荐！

解决依赖冲突（如何选择重复依赖）方式：
  1. 自动选择原则
      - 短路优先原则（第一原则）

          A—>B—>C—>D—>E—>X(version 0.0.1)

          A—>F—>X(version 0.0.2)

          则A依赖于X(version 0.0.2)。
      - 依赖路径长度相同情况下，则“先声明优先”（第二原则）

          A—>E—>X(version 0.0.1)

          A—>F—>X(version 0.0.2)

          在`<depencies></depencies>`中，先声明的，路径相同，会优先选择！
## maven导入失败解决方案

清除本地 Maven 仓库缓存（`lastUpdated` 文件），因为只要存在`lastupdated`缓存文件，刷新也不会重新下载。本地仓库中，根据依赖的`gav`属性依次向下查找文件夹，最终删除内部的文件，刷新重新下载即可！
```sh
@echo off
rem 这里写你的仓库路径
set REPOSITORY_PATH=D:\repository
rem 正在搜索...
for /f "delims=" %%i in ('dir /b /s "%REPOSITORY_PATH%\*lastUpdated*"') do (
    del /s /q %%i
)
rem 搜索完毕
pause
```

## maven 核心功能依赖和构建管理

项目构建是指将源代码、依赖库和资源文件等转换为可执行或可部署的应该程序过程

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/202403072042433.png)

**maven 命令**

|命令|描述|
|---|---|
|mvn clean|清理编译或打包后的项目结构,删除target文件夹|
|mvn compile|编译项目，生成target文件|
|mvn test|执行测试源码 (测试)|
|mvn site|生成一个项目依赖信息的展示页面|
|mvn package|打包项目，生成war / jar 文件|
|mvn install|打包后上传到maven本地仓库(本地部署)|
|mvn deploy|只打包，上传到maven私服仓库(私服部署)|

**maven 构建生命周期**
构建生命周期可以理解成是一组固定构建命令的有序集合，触发周期后的命令，会自动触发周期前的命令！也是一种简化构建的思路!

- 清理周期：主要是对项目编译生成文件进行清理

    - 包含命令：clean
- 默认周期：定义了真正构件时所需要执行的所有步骤，它是生命周期中最核心的部分

    - 包含命令：compile - test - package - install / deploy
- 报告周期

    - 包含命令：site

    - 打包: mvn clean package 本地仓库: mvn clean install

## maven 继承和聚合特性

### maven 工程继承关系
Maven 继承是指在 Maven 的项目中，让一个项目从另一个项目中**继承配置信息**的机制。继承可以让我们在多个项目中共享同一配置信息，简化项目的管理和维护工作。

**继承作用：** 在父工程中统一管理项目中的依赖信息,进行统一版本管理!
### maven 聚合工程关系

Maven 聚合是指将多个项目组织到一个父级项目中，通过触发父工程的构建,统一按顺序触发子工程构建的过程!!

**聚合作用：** 
- 统一管理子项目构建：通过聚合，可以将多个子项目组织在一起，方便管理和维护。
- 优化构建顺序：通过聚合，可以对多个项目进行顺序控制，避免出现构建依赖混乱导致构建失败的情况。
