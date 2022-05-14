## XSS漏洞
XSS（Cross SiteScripting)跨站脚本，较合适的方式应该叫做跨站脚本攻击，诞生于1996年，人们经常将跨站脚本攻击（Cross Site Scripting）缩写为CSS，但这会与层叠样式表（Cascading Style Sheets，CSS）的缩写混淆。因此，有人将跨站脚本攻击缩写为XSS。
###反射型
反射型是非持久型，攻击者需要事先制作好工具链接，需要欺骗用户点击才能触发
###### 反射型(GET)
 通过url提交数据
##### 反射型（POST）
以表单的方式在请求体里提交
- 攻击者伪造表单自动提交页面
- 用户request伪造页面，触发表单
- 页面js自动post表单数据，触发xss。访问存在post型xss漏洞
- 执行js窃取cookie
- 攻击者利用cookie伪造用户登录，造成伤害
### DOM型
DOM型漏洞是基于文档对象模型的一种漏洞
通过document 将脚本拼接到页面标签中
### 存储型
payload 被存储到数据库内，每次访问都会触发
### xss盲打
xss盲打不是攻击类型，而是一种攻击场景，将页面数据提交，页面虽然没有回显，如果后台没有做严格的过滤，登录到后台管理页面，页面把提交的内容数据，那么后台管理员可能会遭到XSS攻击
##### XSS过滤
- 前端绕过，直接抓包重放，或者修改html页面代码
- 大小写（一个大写字母一个小写字母）
- 拼接
-  使用注释绕过进行干扰<scri<!--test-->pt>alert(111)</sc<!--test-->ript>

## CSRF
csrf 跨站请求伪造，这种漏洞是服务端对用户审核不严格导致的，就是伪造一个用户的身份做一些违法的事情，举例，假如一个用户A打开了一个网站并且登录返回了一个cookie，此时用户又访问了一个钓鱼网站，钓鱼网站会直接访问刚刚的网站，这时计算器会吧刚刚的cookie 发给服务器，这样就可以冒用他的身份。
##### 防范措施
1. 对token 增加随机码，每次都不一样
2. 不要在客户端保存敏感信息
3. 关闭或退出时会话过期机制
4. 设置会话过期机制
5. 通过http头部的referer来限制页面
6. 增加验证码
## SSRF 
SSRF服务端请求伪造 ：攻击者诱导服务器将HTTP请求发送到攻击者选择的一个目标上。一般情况下SSRF攻击的目标是从外网无法访问的内部系统
##### 容易出现的地方
- 转码服务
- 在线翻译
- 分享 http://share.xxx.com/index.php?url=http://127.0.0.1
- 图片加载一下载地址 http://image.xxx.com/image.php?image=http://127.0.0.1
- 图片文章收藏功能 http://title.xxx.com/title?title=http://title.xxx.com/as52ps63de
-  从URL关键字中寻找：`share`、`wap`、`url`、`link`、`src`、`source`、`target`、`u`、`3g`、`display`、`sourceURl`、`imageURL`、`domain`………
##### 漏洞检测
假设一个漏洞场景：某网站有一个在线加载功能可以把指定的远程图片加载到本地，功能链接如下：
http://www.xxx.com/image.php?image=http://www.xxc.com/a.jpg

那么网站请求的大概步骤应该是类似以下：
用户输入图片地址->请求发送到服务端解析->服务端请求链接地址的图片数据->获取请求的数据加载到前台显示

这个过程中可能出现问题的点就在于请求发送到服务端的时候，系统没有效验前台给定的参数是不是允许访问的地址域名，例如，如上的链接可以修改为：http://www.xxx.com/image.php?image=http://127.0.0.1:22

如上请求时则可能返回请求的端口banner。如果协议允许，甚至可以使用其他协议来读取和执行相关命令。例如
```
http://www.xxx.com/image.php?image=file:///etc/passwd
http://www.xxx.com/image.php?image=dict://127.0.0.1:22/data:data2 (dict可以向服务端口请求data data2)
http://www.xxx.com/image.php?image=gopher://127.0.0.1:2233/_test (向2233端口发送数据test,同样可以发送POST请求)
```
##### 绕过方式
- 限制为http://www.xxx.com 域名时（利用@）
如：`http://www.aaa.com@www.bbb.com@www.ccc.com
- 采用短网址绕过，比如百度短地址[https://dwz.cn/](https://dwz.cn/)  
- 利用特殊域名  原理是DNS解析。xip.io可以指向任意域名
```
127.0.0.1.xip.io，可解析为127.0.0.1
```
- 采用进制转换，127.0.0.1八进制：0177.0.0.1。十六进制：0x7f.0.0.1。十进制：2130706433
- 可以利用[::]来绕过localhost
```
http://[::]:80/ >>> http://127.0.0.1
```
- 利用句号
```
127。0。0。1 >>> 127.0.0.1
```
## 文件包含漏洞
## XXE漏洞
xxe漏洞全称XML外部实体注入漏洞，XXE漏洞发生在应用程序解析XM输入时，没有禁止外部实体的加载，导致可加载恶意的外部文件，造成文件读取、命令执行、内网端口扫描，攻击内网网站，发起DOS攻击
#### XML实体
根据实体**来源**可定义为内部实体和外部实体。XML定义了两种类型的ENTITY，一种在XML文档中使用，另一种作为参数在DTD文件中使用，定义好的ENTITY在文档中通过“&实体名；”来使用

#### DTD
DTD（文档类型定义）用来是定义 XML 文档的合法构建模块。
DTD 可以在 XML 文档内声明，也可以外部引用。
- 内部声明：
```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE xdsec [
<!ELEMENT methodname ANY >
<!ENTITY xxe(实体引用名) SYSTEM "file:///etc/passwd"(实体内容) >]>
<methodcall>
<methodname>&xxe;</methodname>
</methodcall>
```
- 外部声明：
有SYSTEM和PUBLIC两个关键字，表示实体来自本地计算机还是公共计算机，外部实体的引用可以借助各种协议
```md
file:///path/to/file.ext
http://url
php://filter/read=convert.base64-encode/resource=conf.php
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE xdsec [
<!ELEMENT methodname ANY >
<!ENTITY xxe(实体引用名) SYSTEM "file:///etc/passwd"(实体内容) >]>
<methodcall>
<methodname>&xxe;</methodname>
</methodcall>
```
[XXE漏洞](https://www.cnblogs.com/zhaijiahui/p/9147595.html)
## 反序列化漏洞
## NMAP
```

1. nmap -sT 192.168.96.4  //TCP连接扫描，不安全，慢

2. nmap -sS 192.168.96.4  //SYN扫描,使用最频繁，安全，快

3. nmap -Pn 192.168.96.4  //目标机禁用ping，绕过ping扫描

4. nmap -sU 192.168.96.4  //[UDP](https://so.csdn.net/so/search?q=UDP&spm=1001.2101.3001.7020)扫描,慢,可得到有价值的服务器程序

5. nmap -sI 僵尸ip 目标ip  //使用僵尸机对目标机发送数据包

6. nmap -sA 192.168.96.4  //检测哪些端口被屏蔽

7. nmap 192.168.96.4 -p <portnumber>  //对指定端口扫描

8. nmap 192.168.96.1/24 //对整个网段的主机进行扫描

9. nmap 192.168.96.4 -oX myscan.xml //对扫描结果另存在myscan.xml

10. nmap -T1~6 192.168.96.4  //设置扫描速度，一般T4足够。

11. nmap -sV 192.168.96.4  //对端口上的服务程序版本进行扫描

12. nmap -O 192.168.96.4  //对目标主机的操作系统进行扫描

13. nmap -sC <scirptfile> 192.168.96.4  //使用脚本进行扫描，耗时长

14. nmap -A 192.168.96.4  //强力扫描，耗时长

15. nmap -6 ipv6地址   //对ipv6地址的主机进行扫描

16. nmap -f 192.168.96.4  //使用小数据包发送，避免被识别出

17. nmap –mtu <size> 192.168.96.4 //发送的包大小,最大传输单元必须是8的整数

18. nmap -D <假ip> 192.168.96.4 //发送参杂着假ip的数据包检测

19. nmap --source-port <portnumber> //针对防火墙只允许的源端口

20. nmap –data-length: <length> 192.168.96.4 //改变发生数据包的默认的长度，避免被识别出来是nmap发送的。

21. nmap -v 192.168.96.4  //显示冗余信息(扫描细节)

22. nmap -sn 192.168.96.4  //对目标进行ping检测，不进行端口扫描（会发送四种报文确定目标是否存活,）

23. nmap -sP 192.168.96.4  //仅仅对目标进行ping检测。

24. nmap -n/-p 192.168.96.4  //-n表示不进行dns解析，-p表示要

25. nmap --system-dns 192.168.96.4  //扫描指定系统的dns服务器

26. nmap –traceroute 192.168.96.4  //追踪每个路由节点。

27. nmap -PE/PP/PM: 使用ICMP echo, timestamp, and netmask 请求包发现主机。

28. nmap -sP 192.168.96.4       //主机存活性扫描，arp直连方式。

29. nmap -p "*" 192.168.96.4 //扫描所有端口 Nmap默认扫描的端口是1000个

30. nmap -iR [number]       //对随机生成number个地址进行扫描。
```
[CSDN原文地址](https://blog.csdn.net/qq_37964989/article/details/84330693)
[CSDN namp](https://blog.csdn.net/weixin_45551083/article/details/106967465)


