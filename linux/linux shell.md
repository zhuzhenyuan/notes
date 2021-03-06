
教程

# shell编程

> bash程序由以下三部分组成：  
> 以“#！”开头指定的解释器，以“#”开头的注释行，其他的都是可执行语句，即程序体。
```bash
#!/bin/bash
```

# 一、shell编程之变量

## bash变量命名规则
* 只能包含字母、数字和下划线，并且不能以数字开头；
* 不应该跟系统中已有的环境变量重名；
* 最好能见名知意

## 1.用户自定义变量

> shell中的变量值都是字符型的，要进行算数运算要通过另外的方式，在后面介绍  

### 定义变量

>格式：变量名=变量值 
 
举例：
```shell
x=5  
name="zhu"
```
### 变量调用：

>echo ${变量名}(推荐)或者echo $变量名  


举例：

	echo ${x}或者$x
	echo ${name}或者$name

## 2.bash环境变量

> 与用户自定义变量区别：  
> 用户自定义变量：  
> 是局部变量，只在当前的Shell中生效  
> 环境变量：  
> 是全局变量，在在当前Shell和这个Shell的所有子Shell中生效  
> 对系统生效的环境变量名和变量作用是固定的

### 设置环境变量

> export 变量名=变量值  
> 或  
> 变量名=变量值  
> export 变量名

### 查看环境变量

* set :查看所有变量
* env :查看环境变量

### 删除环境变量

* unset 变量名

### 常用环境变量

* HOSTNAME :主机名
* SHELL :当前的shell
* TERM :终端环境
* HISTSIZE :历史命令条数
* SSH_CLIENT :当前操作环境是用ssh连接的，这里记录客户端ip
* SSH_TTY :ssh连接的终端时pts/1
* USER :当前登录的用户

### PATH环境变量

> PATH变量：系统查找命令的路径
```shell
echo $PATH #查看PATH环境变量
PATH="$PATH":/root/sh #增加PATH变量的值
```

## 3.bash语系变量  

> locale : 查询当前系统语系  
> - LANG :定义系统主语系的变量
> - LC_ALL :定义整体语系的变量

```shell
	echo $LANG #查看系统当前语系
	locale -a | more #查看Linux支持的所有语系
```

### 查询系统默认语系
> 使用以下命令查看配置文件

```shell
cat /etc/sysconfig/i18n
```

## 4.位置参数变量

### 位置参数变量

| 位置参数变量 | 作用                                   |
|:-------|:-------------------------------------|
| $n     | n为数字，$0代表命令本身，一到九参数：$1-$9，十以上： ${10} |
| $*     | 代表命令行中所有的参数，$*吧所有的参数看成一个整体           |
| $@     | 这个变量也代表命令行中所有的参数，不过$@吧每个参数区分对待       |
| $#     | 这个变量代表命令行中所有参数的个数                    |


举例:新建一个文件名为 a.sh,将以下程序写入文件  
```shell
#!/bin/bash
num1=$1
num2=$2
sum=$(( $num1 + $num2 )) #变量sum的和是num1加num2
echo $sum #打印变量sum的值
```
> 编辑完成，退出后对文件赋予执行权限  

```shell
	chmod 755 a.sh  
	./a.sh 1 2  #执行
```

## 5.预定义变量

| 预定义变量 | 作用                                     |
|:------|:---------------------------------------|
| $?    | 最后一次执行的命令的返回状态，若为0，上一个命令正确执行，非0，则执行不正确 |
| $$    | 当前进程的进程号                               |
| $!    | 后台运行的最后一个进程的进程号                        |


### 接受键盘输入

```
read [选项] [变量名]

*  选项:
*  -p "提示信息": 在等待read输入时， 输出提示信息
* -t 秒数:read命令会一直等待用户输入，使用此选项可指定等待时间
* -n 字符数:read命令值接受指定的字符数，就会执行
* -s :隐藏输入的数据，适用于机密信息的输入

```
举例:
```shell
#!/bin/bash
read -p "please input your name: " name
echo -e "\n"  #使用 -e 选项就能使echo识别"\n"
echo $name
```

# 二、shell编程之数值运算

## declare命令

declare 声明变量类型

```
declare [+/-] [选项] 变量名
* 选项
* -: 给变量设定类型属性
* +: 取消变量的类型属性
* -a: 将变量声明为数组型
* -i: 将变量声明为整数型（integer）
* -x: 将变量声明为环境变量
* -r: 将变量声明为只读变量
* -p: 显示指定变量的被声明的类型
```

### 把变量声明为数值型

举例:

```shell
aa=11
bb=22
echo $aa+$bb
declare -i cc=$aa+$bb
echo $cc
```

### 声明数组变量

举例:
```shell
#定义数组
movie[0]=aa
movie[1]=bb
declare -a movie[2]=live
#查看数组
echo ${movie}
echo ${movie[2]}
echo ${movie[*]}
```

### 声明环境变量

> declare -x test=123 #和export作用相似，但其实是declare命令的作用  
> set #查看环境变量

## 数值运算方法

> 数值运算方法1:使用declare定义


> 数值运算方法2:使用 expr或let数值运算工具

举例:

```shell
aa=1
bb=22
dd=$(expr $aa + $bb) #"+"号左右两侧必须有空格
echo $dd
```

> 数值运算方法3:"$((运算式))"（推荐）或者"$[运算式]"

举例：

```shell
aa=1
bb=3
cc=$(( $aa+$bb ))
dd=$[ $aa+$bb ]
echo $cc
echo $dd
```

## 变量测试(了解)

> 变量测试在脚本优化时使用，一般使用者，只要了解即可

| 变量置换方式     | 变量y没有设置     | 变量y为空       | 变量y设置值    |
|:-----------|:------------|:------------|:----------|
| x=${y-新值}  | x=新值        | x为空         | x=$y      |
| x=${y:-新值} | x=新值        | x=新值        | x=$y      |
| x=${y+新值}  | x为空         | x=新值        | x=新值      |
| x=${y:+新值} | x为空         | x为空         | x=新值      |
| x=${y=新值}  | x=新值 y=新值   | x为空 y值不变    | x=$y y值不变 |
| x=${y:=新值} | x=新值 y=新值   | x=新值 y=新值   | x=$y y值不变 |
| x=${y?新值}  | 新值输出到标准错误输出 | x=为空        | x=$y      |
| x=${y:?新值} | 新值输出到标准错误输出 | 新值输出到标准错误输出 | x=$y      |


# 三、shell编程之条件判断和流程控制
## 1.条件判断式语句

> 两种判断格式  
> test -e /root/install.log  
> [ -e /root/install.log ]

举例：按照

	[ -e /root/install.log ] && echo yes || echo no
	#第一个判断命令如果正确执行，则打印“yes”，否则打印“no”

### 按照文件类型进行判断

| 测试选项  | 作用                        |
|:------|:--------------------------|
| -b 文件 | 判断给文件是否存在，并且是否为块设备文件，是为真  |
| -c 文件 | 判断给文件是否存在，并且是否为字符设备文件，是为真 |
| -d 文件 | 判断给文件是否存在，并且是否为目录文件，是为真   |
| -e 文件 | 判断给文件是否存在，是为真             |
| -f 文件 | 判断给文件是否存在，并且是否为普通文件，是为真   |
| -L 文件 | 判断给文件是否存在，并且是否为符号链接文件，是为真 |
| -p 文件 | 判断给文件是否存在，并且是否为管道文件，是为真   |
| -s 文件 | 判断给文件是否存在，并且是否为非空，非空为真    |
| -S 文件 | 判断给文件是否存在，并且是否为套接字文件，是为真  |


### 按照文件权限进行判断

| 测试选项  | 作用  |
|---|---|
| -r 文件  | 判断给文件是否存在，并且是否拥有读权限,有为真  |
| -w 文件  | 判断给文件是否存在，并且是否拥有写权限,有为真  |
| -x 文件  | 判断给文件是否存在，并且是否拥有执行权限,有为真  |
| -u 文件  | 判断给文件是否存在，并且是否拥有SUID权限,有为真  |
| -g 文件  | 判断给文件是否存在，并且是否拥有SGID权限,有为真  |
| -k 文件  | 判断给文件是否存在，并且是否拥有SBit权限,有为真  |


### 两个文件之间进行比较

| 测试选项  | 作用  |
|---|---|
| 文件1 -nt 文件2  | 判断文件1的修改时间是否比文件2的新(如果新则为真)  |
| 文件1 -ot 文件2  | 判断文件1的修改时间是否比文件2的旧(如果旧则为真)  |
| 文件1 -ef 文件2  | 判断文件1是否和文件2的lnode号一致，可以理解为是否为同一文件，可用于判断硬链接  |


### 两个整数之间比较

| 测试选项  | 作用  |
|---|---|
| 整数1 -eq 整数2  | 判断整数1是否和整数2相等(相等为真)  |
| 整数1 -ne 整数2  | 判断整数1是否和整数2不相等(不相等为真)  |
| 整数1 -gt 整数2  | 判断整数1是否大于整数2(大于为真)  |
| 整数1 -lt 整数2  | 判断整数1是否小于整数2(小于为真)  |
| 整数1 -ge 整数2  | 判断整数1是否大于等于数2(大于等于为真)  |
| 整数1 -le 整数2  | 判断整数1是否小于等于整数2(小于等于为真)  |


### 字符串的判断

| 测试选项  | 作用  |
|---|---|
| -z 字符串  | 判断字符串是否为空(为空返回真)  |
| -n 字符串  | 判断字符串是否为非空(非空返回真)  |
| 字串1 == 字串2  | 判断字符串1是否和字符串2相等(相等返回真)  |
| 字串1 != 字串2  |  判断字符串1是否和字符串2不相等(不相等返回真) |

### 多重条件判断

| 测试选项  | 作用  |
|---|---|
| 判断1 -a 判断2  | 逻辑与，都成立为真 |
| 判断1 -o 判断2  |  逻辑或，有一个成立就为真 |
|  !判断 | 逻辑非，取反  |


## 2.条件判断if语句

### 单分支if语句

> 格式  
> if [ 条件判断式 ];then  
> 　程序  
> fi  
> 或者  
> if [条件判断式 ]  
> 　then  
> fi  
> [ 条件判断式 ]就是使用test命令判断，所以中括号和太监判断式之间必须有空格

### 双分支if语句

> 格式  
> if [ 条件判断式 ]  
> 　then  
> 　　条件成立时，执行程序  
> 　else  
> 　　条件不成立时，执行的另一个程序  
> fi  

### 多分支if语句

> 格式  
> if [ 条件判断式 ]  
> 　then  
> 　　当条件判断式1成立时，执行程序1  
> elif  
> 　then  
> 　　当条件判断式2成立时，执行程序2  
> ...省略更多条件...  
> else  
> 　　当所有条件都不成立时，最后执行此程序  
> fi  

##  3. 多分支case语句

> 格式  
> case $变量名 in  
> 　"")  
> 　　如果变量的值等于1，则执行程序1  
> 　　;;  
> 　"")  
> 　　如果变量的值等于1，则执行程序1  
> 　　;;  
> 　...省略其他分支...  
> 　*)  
> 　　如果变量的值都不是以上的值，则执行此程序  
> 　　;;  
> esac  


##  4. for循环

### 语法一

> 格式  
> for 变量 in 值1 值2 值3...  
> 　do  
> 　　程序  
> 　done  

举例1:
```shell
#!/bin/bash
for i in 1 2 3 4 5
    do
        echo $i
    done
```
举例2:
```shell
#!/bin/bash
#列出当前目录下的文件
for i in ls
    do
        echo $i
    done
```

### 语法二

> 格式  
> for (( 初始值;循环控制条件;变量变化 ))  
> 　do  
> 　　程序  
> 　done  

举例:
```shell
#!/bin/bash
#从1加到100
s=0
for(( i=1;i<=100;i=i+1 ))
    do
        s=$(( $s+$i ))
    done
echo "The sum of 1+2+...+100 is : $s"
```

##  5. while循环和until循环

> while格式  
> while [ 条件判断式 ]  
> 　do  
> 　　程序  
> 　done  

举例：
```shell
#!/bin/bash
#从1加到100
i=1
s=0
while [ %i -le 100 ]
#如果变量i的值小于等于100，则执行循环
    do
        s=$(( $s+$i ))
        i=$(( $i+1 ))
    done
echo "The sum is: $s"
```

> until格式  
> until [ 条件判断式 ]  
> 　do  
> 　　程序  
> 　done  

举例:
```shell
#!/bin/bash
#从1加到100
i=1
s=0
until [ $i -gt 100 ]
#循环知道变量i的值大于100，就停止循环
    do
        s=$(( $s+$i ))
        i=$(( $i+1 ))
    done
echo "Teh sum is: $s"
```

## 6.break命令
> break命令允许跳出所有循环（终止执行后面的所有循环）

举例:
```shell
#!/bin/bash
while :
do
    echo -n "输入 1 到 5 之间的数字:"
    read aNum
    case $aNum in
        1|2|3|4|5) echo "你输入的数字为 $aNum!"
        ;;
        *) echo "你输入的数字不是 1 到 5 之间的! 游戏结束"
            break
        ;;
    esac
done
```

执行以上代码，输出结果为：

> 输入 1 到 5 之间的数字:3  
> 你输入的数字为 3!  
> 输入 1 到 5 之间的数字:7  
> 你输入的数字不是 1 到 5 之间的! 游戏结束  

## 7.continue命令

> continue命令与break命令类似，只有一点差别，它不会跳出所有循环，仅仅跳出当前循环

举例:
```shell
#!/bin/bash
while :
do
    echo -n "输入 1 到 5 之间的数字:"
    read aNum
    case $aNum in
        1|2|3|4|5) echo "你输入的数字为 $aNum!"
        ;;
        *) echo "你输入的数字不是 1 到 5 之间的!"
            continue
            echo "游戏结束"
        ;;
    esac
done
```
> 运行代码发现，当输入大于5的数字时，该例中的循环不会结束，语句 echo "Game is over!" 永远不会被执行。


# 四、shell程序调试

> Bash常用的调试方法是带-x执行程序，这样会把执行到的语句全部显示出来。  

格式：

> bash -x file.sh

举例:

	...
	set -v
	#需要调试的语句
	set +v
	...


> 如果bash程序很长，可在需要调试的程序块前后标记调试标记—块前插入语句set –v，块后插入语句set+v即可，这样调试时只打印调试块中的执行路径。建议bash程序的语句最好不要超过1000行。


