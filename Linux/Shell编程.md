# Shell编程

## 变量

```shell
echo $USER

User=a //新建系统变量并且赋值

unset User 删除变量

export User 使变量全局有效
```

### 特殊变量

#### $n 

类似于方法传参

```shell
#!/bin/bash
echo $0 $1 $2 $3

sh hello.sh a b c 

hello.sh a b c 
```

#### $#

获得输入的所有参数个数

```
#!/bin/bash
echo $#
```

#### $*

将输入的参数当成一个整体

#### $@

也代表所有的参数，但是分开对待

#### $?

判断上一条命令是否正确执行（0代表正常）

## 运算符

### 基本语法

$((运算符))或者$[运算符]

### 运算符

```
expr + , - , \* , / , %
```

## 条件判断

### 基本语法

[ 判断条件 ]

### 判断条件

#### 比较

| 语法 | 说明               |
| ---- | ------------------ |
| =    | 判断字符串是否相等 |
| -it  | 小于               |
| -le  | 小于等于           |
| -eq  | 等于               |
| -gt  | 大于               |
| -ge  | 大于等于           |
| -ne  | 不等于             |

#### 文件权限

| 语法 | 说明 |
| ---- | ---- |
| -r   | 读   |
| -w   | 写   |
| -x   | 执行 |

#### 文件

| 语法 | 说明                       |
| ---- | -------------------------- |
| -f   | 文件存在且是一个正常的文件 |
| -e   | 文件存在                   |
| -d   | 文件存在是一个目录         |

## 流程控制

### if

```shell
#!/bin/bash
if [ $1 = 1 ]
then
	echo 10
elif [ $1 = 2 ]
then
	echo 20
elif [ 1 = 1 ]
then
	echo 30
fi
```

### case

```shell
#!/bin/bash
case $1 in
1)
	echo 10
;;  //代表break
2)
	echo 20
;;
*) //*代表默认
	echo 30
;; 
esac
```

### for

```shell
//1
#!/bin/bash
sum=0
for((i=1;i<=100;i++))
do
	sum=$[$sum + $i]
done
echo $sum

//2
#!/bin/bash
j=0
for i in $@
	do
		j=$[$j+1]
		echo "第" $j "个元素为" $i
	done
```

### while

```shell
while[条件判断]
	do
		
	done
```

### read

```shell
#!/bin/bash
read -t 10 -p "input your name" name 
echo $name
```

## 函数

### 系统函数

#### basename

获得文件名

```shell
basename /home/shell/read.sh

basename /home/shell/read.sh sh
```

#### dirname

获取文件路径（不包含文件名）

```shell
dirname /home/shell/read.sh
```

### 自定义函数

```shell
#!/bin/bash
function sum ()
{
        s=$[$1+$2]
        echo $s
}

sum $1 $2
```

