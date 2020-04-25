# Shell

## 定义变量

格式：name=value

注：等号两边**不能**有空格

## 访问变量

格式：$name

## 特殊变量：$n

n表示数字[1-9]，表示执行shell脚本时传入的参数。$0为当前shell文件的文件名。十个以上的参数需要通过使用${数值}获取。

## 特殊变量：$#

返回参数的个数，不包含$0.

## 特殊变量：$*、$@

$*和$@都是获取所有的输入参数。区别如下：

$*：将所有的参数看作是一个整体。

$@：每个参数区别对待，在使用循环的时候会很有帮助。

## 特殊变量：$?

$?用于获取前一个命令是否执行成功的状态，执行成功输出0，否则输出其他值。

## 运算符

shell中默认将所有的变量类型看作是字符串，如:

```shell
sum=1+2
echo $sum
// 输出为 1+2
```

如果要实现数值之间的计算，有如下两种方式：

1. 使用`$((运算表达式))`或`$[运算表达式]`
2. 使用`expr`关键字。[ + , - , \\* , / , % ] -> [加，减，乘，除，取余]，需要注意乘法符号前需加\，expr运算符间要有空格。

```shell
### $[]
sum=$[(1+2)*3]

### expr 表达式 
$ expr 1+2
# 1+2
$ expr 1 +2
#表达式错误
$ expr 1 + 2
# 3

$ expr `expr 1 + 2` \* 3
# 9
```

## 条件判断

### 语法

[ condition ] 

注意：condition前后要有空格，条件非空即为true

### 常用的条件判断

#### 1.两个整数之间的比较

| 表达式 | 解释                      |
| ------ | ------------------------- |
| =      | 字符串比较                |
| -lt    | 小于（less than）         |
| -le    | 小于等于（less equal）    |
| -eq    | 等于（equal）             |
| -gt    | 大于（greater than）      |
| -ge    | 大于等于（greater equal） |
| -ne    | 不等于（not equal）       |

#### 2.按照文件权限进行判断

| 表达式 | 解释                    |
| ------ | ----------------------- |
| -r     | 有读的权限（read）      |
| -w     | 有写的权限（write）     |
| -x     | 有执行的权限（execute） |

#### 3.按照文件类型进行判断

| 表达式 | 解释                               |
| ------ | ---------------------------------- |
| -f     | 文件存在并且是一个常规文件（file） |
| -e     | 文件存在（existence）              |
| -d     | 文件存在并是一个目录（directory）  |

## 流程控制

### if判断

语法：

```shell
if [ 表达式 ];then
	// 语句块
elif [ 表达式 ];then
	// 语句块
else
	// 语句块
fi

// or

if [ 表达式 ]
	then
		语句块
fi
```

**注意：**

1. [ 条件表达式 ]，中括号和条件表达式之间必须有空格。
2. if后要有空格

```sh
#!/bin/bash

if [ $1 -eq 1 ];then
	echo "1"
elif [ $1 -eq 2 ];then
	echo "2"
else
	echo "3"
fi
```

### case 语句

语法：

```shell
case $变量名 in
	"值1")
		# 语句块
	;;
	"值2")
		# 语句块
	;;
	*)
	# 默认语句块
	;;
esac
```

**注意：**

* case行尾必须为`in`，每一个模式匹配必须以`)`结束
* 双分号`;;`表示命令序列结束，相当于Java中的break
* 最后的`*)`表示默认模式，相当于Java中的default

```shell
#!/bin/bash

case $1 in
1)
	echo "第一个case"
;;
2)
	echo "第二个case"
;;
*)
	echo "默认的case"
;;
esac
```

### for循环

基本语法1:

```shell
for(( 初始值;循环控制条件;变量变化 ))
do
		# 代码块
done	
```

```shell
#!/bin/bash

sum=0
for(( i=1;i<=100;i++ ))
do
	sum=$[$sum+$i]
done
```

基本语法2:

```shell
for 变量 in 值1 值2 值3...
do
	// 代码块
done
```

```shell
#!/bin/bash

for i in $*
do
	echo "$i"
done

# 将$*看着是一个整体
for i in "$*"
do
	echo "$i"
done

for i in `seq 1 10`
do
	echo "$i"
done
```

### while 循环

基础语法:

```shell
while [ 条件判断式 ]
do
	# 代码块
done
```

```shell
s=0
i=1
while [ $i -le 100 ]
do
	s=$[$s + $i]
	i=$[$i + 1]
done
```

## read 读取控制台输入

基本语法:

```shell
read (选项) (参数)
```

选项:

> -p : 指定读取值时的提示符
>
> -t : 指定读取值时等待的时间(秒)

参数:

> 变量 : 指定读取值的变量名

```shell
#!/bin/bash

read -t 7 -p "请在7秒内输入名称" INFO
echo INFO
```

## 函数

### 系统函数

#### basename

基本语法:

```shell
basename [string/pathname] [suffix]
```

basename会保留指定路径的最后一个分隔符后的内容, 如果指定了suffix, 会从生成的名称中删除该后缀.

```shell
basename /home/box/test.txt
# text.txt
basename /home/box/test.txt .txt
# text
```

#### dirname

基本语法:

```shell
dirname 文件绝对路径
```

从给定的文件绝对路径中, 移除最后的文件部分, 仅保留文件的上一级目录的路径

```shell
dirname /home/box/test.txt
# /home/box
```

### 自定义函数

基本语法:

```shell
[ function ] functionname[()]
{
	# do anything
	[return int;]
}
# invoke
functionname
```

**注意:**

1. 由于shell逐行执行, 必须在调用函数之前声明函数
2. 函数返回值只能通过`$?`获取, 如果省略return语句,则默认使用最后一条命令的运行结果
3. 返回值仅能为 `0-255`

```shell
#!/bin/bash

function sum()
{
	s=0
	s=$[$1 + $2]
	echo 'sum = $s'
	return 0;
}

read -p '参数1:' P1
read -p '参数2:' P2

sum $P1 $P2
```

## Shell工具

### cut

用于在文件中剪切数据, cut命令从文件的每一行剪切字节、字符和字段输出.

基本用法:

```shell
cut [选项参数] filename
```

选项:

> -f : 列号,提取第几列, 切割多列使用逗号分隔, 范围使用-
>
> -d : 分隔符, 按照指定分隔符分隔列, 默认为指标符

```shell
# a.txt
第一列:第二列:第三列:第四列:第五列

# 第二列
cut -d ':' -f 2 ./a.txt
# 第二列第三列
cut -d ':' -f 2,3 ./a.txt
# 第二列到第四列, 最后一位会被排除
cut -d ':' -f 2-5 ./a.txt
# 第二列以后的所有
cut -d : -f 2- ./a.txt
# 第二列及其之前的所有
cut -d : -f -2 ./a.txt
```

### sed

sed是一种流编辑器,它一次处理一行内容, 处理时,将当前处理的行存储在临时缓冲区中, 称为`模式空间`, 接着用sed命令处理缓冲区中的内容,处理完成后把缓冲区的内容送往屏幕. 接着处理下一行, 不断循环直到文件末尾.

基本用法:

```shell
sed [选项参数] 'command' filename
```

选项参数:

> -e : 直接在指令列模式上进行sed的动作编辑, 可以使用多个该参数串联起一个处理过程

命令功能:

> a : 新增, a的后面可以接字符串, 在下一行出现
>
> d : 删除
>
> s : 查找并替换

```shell
# a.txt
zhang san
li si
wang wu

# 在第二行后面添加 zhao liu
sed '2a zhao liu' a.txt
# 删除包含 li 的行
sed '/li/d' a.txt
# 将 si 替换为 qi  # g表示全局替换
sed 's/si/qi/g' a.txt
# 删除第二行,并将wu替换为shi
sed -e '2d' -e 's/wu/shi/g' a.txt
```

### awk

一个文本分析工具,将文件逐行读入, 以空格为默认分隔符将每行切开,切开的部分在进行分析处理.

基本用法:

```shell
awk [选项参数] 'pattern1{action1} pattern2:{action2}...' filename
patthern : 表示awk在数据中查找的内容,就是匹配模式
action : 在找到匹配内容时所执行的一系列命令
```

选项参数:

> -F : 指定输入文件的分隔符
>
> -v : 赋值一个用户定义的变量

```shell
# 搜索passwd文件所有以root开头的行,并输出第七列
awk -F : '/^root/ {print $7}' /etc/passwd
# 搜索passwd文件所有以root开头的行,并输入第一行与第七行,以逗号分隔
awk -F : '/^root/ {print $1","$7}' /etc/passwd
# 只显示passwd的第一行与第七行,以逗号分隔,且在所有行前面添加列名 user,shell 在最后一行添加 bean,beans
# BEGIN 在所有数据读取之前执行, END 在所有数据读取之后执行
awk -F : 'BEGIN{print "user,shell"} {print $1","$7} END{print "bean,beans"}' /etc/passwd
# 将passwd文件中用户id增加数值1
awk -F :  -v i=1 '{print $3+i}' 
```

**内置变量**

| 变量     | 说明                                |
| -------- | ----------------------------------- |
| FILENAME | 文件名                              |
| NR       | 已读的记录数                        |
| NF       | 浏览的记录域的个数(切割后,列的个数) |

```shell
# 统计passwd文件名, 每行的行号, 每行的列数
awk -F : '{print FILENAME NR NF}' /etc/passwd
```

### sort

将文件进行排序,并将排序结果标准输出

基本用法:

```shell
sort (选项) (参数)
```

选项:

> -n : 依照数值大小排序
>
> -r : 以相反的顺序拍戏
>
> -t : 设置排序时所用的分隔字符
>
> -k : 指定需要排序的列

参数:

> 指定需要排序的文件列表

```shell
# a.txt
aa:1:a1
bb:2:b2

sort -t : -nrk 2 a.txt
```

