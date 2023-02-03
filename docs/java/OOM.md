## 生成dump文件
1.程序挂掉

出现OOM时自动生成dump 在JVM的启动命令增加两个参数
```
-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=./==heap.hprof
```

 2. 程序运行时
- 使用jmap -dump:format=b,live,file=文件名  进程PID,文件名可以使txt,bin,hprof等
```shell
jmap -dump:format=b,live,file=hea.hprof 10127
```
## visualvm 排查OOM

1.选择内存溢出的类，或者选择内存占用最大的类
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20230203230515.png)

2. 找到溢出对象所在的线程
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1675437140785.png)


3. 溢出对应的执行方法

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20230203231143.png)
