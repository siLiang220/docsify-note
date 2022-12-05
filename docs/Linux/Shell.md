## 入门

### source .  sh bash  ./ 的区别

- **source** ：是在当前的shell 内执行,通常用于重新执行刚修改的初始化文档
- **.** ：是source 命令简写
- **sh/bash** ：打开一个subShell 执行，通常在subShell 里运行脚本设置变量，不会影响到父shell
- **./** ：打开一个subShell执行，但需要权限

## 变量

### 系统预定义变量

$HOME $PWD $SHELL $USER

1. 查看当前系统变量的值
```shell
echo $HOME
```
2. 查看shell中所有的变量
```shell
set
```

### 自定义变量
1. 定义变量 变量名=变量值
2. 撤销变量: unset var_name
3. 提升变量为全局变量: export var_name
4. 数学运算 
```shell
a=$((运算式)) 可使用数学运算符
a=$[ 运算式 ] 不可使用数学运算
a=$(expr 运算式)
a=`expr 运算式`
```
5.只读变量
```shell
┌──(kali㉿kali)-[~/scripts]
└─$ readonly b='hello'
                                                                             
┌──(kali㉿kali)-[~/scripts]
└─$ b=1            
zsh: read-only variable: b
```
6. 特殊变量
| 指令/参数 | 使用/作用                              |
| --------- | -------------------------------------- |
| $n        | 获取某个位置参数，十以上 ${10}         |
| $#        | 获取输入参数的个数                     |
| $?        | 返回前一个命令的返回码,0没有错误       | 
| $*        | 以一个单字符串显示所有向脚本传递的参数 |
| $@        | 获取所有的参数，数组形式               |

## 条件判断

Shell test 命令的用法为：

```
test expression
```

expression 之间符号使用空格。当 test 判断 expression 成立时，退出状态为 0，否则为非 0 值。  
  
test 命令也可以简写为`[]`，它的用法为：
```
[ expression ]
```
注意`[]`和`expression`之间的空格，这两个空格是必须的，否则会导致语法错误。

条件判断式中使用and ，or 符号 -a，-o
 
三目运算（command2，command3 可为操作命令）
```shell
command1 && command2 || command3
```

## 流程控制

### if 判断
1. 单分支
```shell
if [ 条件判断式 ];then
	程序
fi	
```
或者
```shell
if [ 条件判断式 ]
then
	程序
fi
```
2. 多分支
```shell
if [ 条件判断式 ]
then
	程序
elif [ 条件判断式 ]
then
	程序
else
	程序
fi	
```

### case语句

```shell
case $变量名 in 
"值1")
	程序
;;
"值2")
	程序
;;
*)
	默认程序
;;
esac
```
;; 相当于break

### for 循环
```shell
for((初始值;循环控制条件;变量变化))
do
	程序
done	
```

```shell
#!/bin/bash
for((i=0;i<$1;i++))
do
        sum=$[ $sum + $i ]
done
echo $sum

```

```shell
for 变量 in 值1 值2 值3
do
	程序
done

for os in linux windows mac;do echo $os;done
```

### while
```shell
while [ 条件判断式 ]
do
	程序
done	
```

## 函数

### 系统函数

## 文本处理工具



