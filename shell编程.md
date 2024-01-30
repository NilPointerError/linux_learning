### 前言

1）Linux运维在进行服务器集群管理时，需要编写shell程序进行服务器管理

2）JavaEE和python程序员，可能会编写shell脚本进行程序或者是服务器的维护，比如编写一个定时备份数据库的脚本

3）大数据程序员，编写shell程序来管理集群

shell是一个命令行解释器，它为用户提供一个向Linux内核发送请求以便运行程序的界面系统级程序，用户可以用shell启动、挂起、停止甚至是编写一些程序

#### shell脚本的执行方式

```shell
sh /path/to/your/script.sh
```

```shell
chmod u+x script.sh
./script.sh#当前目录
```

```shell
chmod u+x script.sh
/root/.../script.sh#绝对路径执行
```

### 示例代码


```shell
#!/bin/bash
echo "hello,world"
```

设置变量

```shell
#!/bin/bash

#案例1：定义变量A
A=100
#输出变量需要加上$
echo A= $A
echo "A= $A"


#案例2：撤销变量A
unset A
echo "A=$A"

#案例3：声明静态变量B=2，不能unset
readonly B=2
echo "B=$B"
#unset B

#将指令返回的结果赋给变量
:<<!
C=`date`
D=$(date)
echo "C=$C"
echo "D=$D"
!
#使用环境变量TOMCAT_HOME
echo "tomcat_home=$TOMCAT_HOME"

```

位置参数变量

```shell
#!/bin/bash
echo "0=$0 1=$1 2=$2" 
#（$0代表命令本身，$1-$9代表第一到第九个参数，十以上的参数用大括号包含，${10}）
echo "所有的参数=$*"   

echo "$@"

echo "参数的个数=$#"
```

预定义变量

```shell
#!/bin/bash


echo "当前执行的进程id=$$"

#以后台的方式运行脚本，并获取他的进程号
/root/shcode/myshell.sh & 
echo "最后一个后台方式运行的进程id=$!"
echo "执行的结果是=$?"
```

运算符

```shell
#!/bin/bash

RES1=$(((2+3)*4))
echo "res1=$RES1"


RES1=$[(2+3)*4]
echo "res1=$RES1"


TEMP=`expr 2 + 3`
echo "temp=$TEMP"
RES4=`expr $TEMP \* 4`
echo "temp=$RES4"


SUM=$[$1+$2]
echo "sum=$SUM"
```

if判断

```shell
#!/bin/bash


if [ "ok" = "ok"  ]
then 

	echo "equal"
fi

# 大于等于
if [ 23 -ge 22 ]
then 
	echo "大于"
fi

# /root/shcode/aaa.txt 目录中的文件是否存在
if [ -f /root/shcode/aaa.txt  ]
then 
	echo "存:在"
fi

if [ string ]
then 
	echo "string"
fi
```

ifelse

```shell
#!/bin/bash

if [ $1 -ge 60 ]
then
	echo "及格"
elif [ $1 -lt 60 ]
then 
	echo "不及格"
fi

```

switch case

```shell
#!/bin/bash


case $1 in 
"1")
echo "周一"
;;
"2")
echo "周二"
;;
*)
echo "other..."
;;
esac
```

while循环

```shell
#!/bin/bash

SUM=0
i=0
while [ $i -le $1 ]
do
	SUM=$[$SUM+$i]
	#i 自增
	i=$[$i+1]
done
echo "结果=$SUM"

```

for循环

```shell
#!/bin/bash
# $* 是把输入的参数，当作一个整体，所以，只会输出一句话
for i in "$*"
do 
	echo "num is $i"
done

echo "======================"
for j in "$@"
do 
	echo "num is $j"
done


#从1加到100输出

SUM=0
for(( i=1; i<=$1; i++))
do
	SUM=$[$SUM+$i]
done
echo "总和SUM = $SUM"
```

自定义函数

```shell
#!/bin/sh
#案例1：计算输入两个参数的和（动态的获取），getSum

#定义函数 getSum
function getSum() {

	SUM=$[$n1+$n2]
	echo "和是=$SUM"

}

#输入两个值
read -p "请输入一个数n1=" n1
read -p "请输入一个数n2=" n2
#调用自定义函数
getSum $n1 $n2

```

read

```shell
#!/bin/bash

#读取控制台输入一个NUM1值
read -p "请输入一个数NUM1=" NUM1
echo "你输入的NUM1=$NUM1"
# 在10s之内输入
read -t 10 -p "请输入一个数NUM2=" NUM2
echo "你输入的NUM1=$NUM2"
```

系统函数

```shell
basename /home/aaa/test.txt
basename /home/aaa/test.txt .txt
dirname /home/aaa/test.txt
```



### 编程案例

每天凌晨2：30备份数据库hspedu到/data/backup/db

备份开始和备份结束能给出相应的提示信息

备份后的文件要求以备份时间为文件名，并打包成 .tar.gz的形式，比如：2024-01-30_111211.tar.gz

在备份的同时，检查是否有10天前备份的数据库文件，有就将其删除