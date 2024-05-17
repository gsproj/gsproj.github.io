---
title: day53-自动化架构-shell自动化编程（二）
date: 2024-5-16 15:23:52
categories:
- 运维
- （二）综合架构
tags: 
---

# 自动化架构-shell自动化编程（二）

今日内容：

1. Shell数学运算
2. Shell逻辑判断



# 一、Shell数学运算

## 1.1 运算符

常用运算符：

| shell-运算符   | 含义                                               |
| -------------- | -------------------------------------------------- |
| +              | 加法符号                                           |
| -              | 减法符号                                           |
| *              | 乘法符号                                           |
| /              | 除法符号                                           |
| %              | 取余                                               |
| ^或** 幂、指数 | 2^10=1024 10个2相乘.                               |
| i=i+1 i++      | 计数，计算次数                                     |
| j=j+5  j+=5    | 求和，累加                                         |
| &&             | 并且，前一个命令执行成功，再执行后面的命令(判断)   |
| \|\|           | 或者，前一个命令执行失败了，再执行后面的命令(判断) |

实验：

```shell
# 随机数字对n取余，可以得到0到n-1范围内的随机数
# 如：得到0-4的随机数
echo $RANDOM % 5 | bc
4
echo $RANDOM % 5 | bc
2
echo $RANDOM % 5 | bc
1

# 检查目录是否存在
ls /etc && echo 目录存在

# 目录不存在则创建
ls /data || mkdir -p /data
```

## 1.2 运算方法（重要）

常用运算方法

| 运算的命令/符号 | 说明                                              | 应用场景                                  |
| --------------- | ------------------------------------------------- | ----------------------------------------- |
| awk             | 可以进行计算，带小数，可以与shell脚本进行变量传递 | 一般计算都可以用awk.                      |
| bc              | 带小数                                            | 一般计算都可以用bc.需要安装.              |
| expr            | 进行计算,整数                                     | 一般用于检查输入内容是否为数字(方法之一). |
| let             | 进行计算,整数,变量直接使用即可                    | 用于计算iՎҡ                               |
| $(())           | 进行计算，整数,变量直接使用即可。                 |                                           |
| $[]             | 进行计算，整数,变量直接使用即可。                 |                                           |

### 1.2.1 使用awk计算（推荐）

>小数正常，10/20正常显示0.5

基础用法

```shell
[root@mn01[ /server/scripts/devops-shell]#awk 'BEGIN{print 1/3}'
0.333333
[root@mn01[ /server/scripts/devops-shell]#awk 'BEGIN{print 1/3 * 100}'
32.3333
```

在awk中使用shell变量

```shell
# 定义变量
[root@mn01[ /server/scripts/devops-shell]#num1=100
[root@mn01[ /server/scripts/devops-shell]#num2=200
# 使用变量
[root@mn01[ /server/scripts/devops-shell]#awk -vn1=$num1 -vn2=$num2 'BEGIN{print n1/n2}'
0.5
```

>提示：
>
>awk -v选项用于创建或修改awk中的变量。 
>
>**-v是shell脚本与awk桥梁**
>
>在awk中各种变量直接使用即可，不要加上$n1,如果加上了会被awk识别为取列  



### 1.2.2 使用bc计算

>加上-l选项，小数正常，10/20正常显示0.50000

熟悉基本使用方法

```shell
# -l 显示小数
echo 1/3 |bc -l
echo 2^10 |bc -l
```



### 1.2.3 使用expr计算

>小数异常，10 / 20 = 0

推荐用于判断变量是否是数字

用于计算坑很多，：

```shell
# 不加空格报错
[root@mn01[ /server/scripts/devops-shell]#expr 1+1
1+1
# 加空格正常
[root@mn01[ /server/scripts/devops-shell]#expr 1 + 1
2
# 到了乘法，不加转义符也报错
[root@mn01[ /server/scripts/devops-shell]#expr 2 * 2
expr: syntax error
# 加了转义符，乘法正常
[root@mn01[ /server/scripts/devops-shell]#expr 2 \* 2
4
# 除法
[root@mn01[ /server/scripts/devops-shell]#expr 2 / 2
1
```

你以为就这些？还有个大坑！

```shell
# $?常用来检测前一句命令是否执行正常
# 我们来执行一条expr命令
[root@mn01[ /server/scripts/devops-shell]#expr 1 + 1
2
# 看似很正常
[root@mn01[ /server/scripts/devops-shell]#echo $?
0
# 再执行一条
[root@mn01[ /server/scripts/devops-shell]#expr 0 + 0
0
# 不是吧！明明成功了，为什么返回1
[root@mn01[ /server/scripts/devops-shell]#echo $?
1
```

#### 案例：检测输入内容是否是数字

```shell
#!/bin/bash
##############################################################
# File Name:07_check_num.sh
# Version:V1.0
# Author:Haris Gong
# Organization:gsproj.github.io
# Desc:检测输入的内容是不是数字
##############################################################

# 输入
read -p "请输入数字: " num

# 检测
expr $num \* 0 + 1 &>/dev/null && echo "$num是数字"
expr $num \* 0 + 1 &>/dev/null || echo "$num不是数字"
```

测试

```shell
[root@mn01[ /server/scripts/devops-shell]#bash 07_check_num.sh
请输入数字: dadsadsa
dadsadsa不是数字
[root@mn01[ /server/scripts/devops-shell]#bash 07_check_num.sh
请输入数字: 123123
123123是数字
```

实现原理：expr可以判断是不是整数

```shell
[root@mn01[ /]#num=rewrew
# 计算失败
[root@mn01[ /]#expr $num \* 0 + 1
expr: non-integer argument

[root@mn01[ /]#num=321321
# 计算成功
[root@mn01[ /]#expr $num \* 0 + 1
1
```

### 1.2.4 使用let进行计算

>小数异常，10/20=0

基本使用

```shell
# 定义变量
[root@mn01[ /]#n1=666
[root@mn01[ /]#n2=999

# 可以直接使用变量
[root@mn01[ /]#let c=n1+n2
[root@mn01[ /]#echo $c
1665

# 还可以进行累加
[root@mn01[ /]#let c++
[root@mn01[ /]#echo $c
1666
```

### 1.2.5 使用$(())进行计算

>小数异常，10/20=0

基本使用

```shell
# 直接计算
[root@mn01[ /]#echo $((n1+n2))
1665
# 赋值给变量
[root@mn01[ /]#d=$((n1+n2))
[root@mn01[ /]#echo $d
1665
```

### 1.2.6 使用$[]进行计算

>小数异常，10/20=0

基本用法，跟$(())类似

```shell
[root@mn01[ /]#echo $[n1+n2]
1665
[root@mn01[ /]#d=$[n1+n2]
[root@mn01[ /]#echo $d
1665
```

## 1.3 运算案例

### 案例1：计算器

参数传入脚本中2个参数，进行计算，输出结果，要求

 ```shell
sh 11.num_calc.sh 10 20
计算10-20:结果
计算10+20:结果
计算10*20:结果
计算10/20:结果
 ```

实现：

```shell
#!/bin/bash
##############################################################
# File Name:08_num_calc.sh
# Version:V1.0
# Author:Haris Gong
# Organization:gsproj.github.io
# Desc: 计算器
##############################################################


num1=$1
num2=$2

# 参数判断
expr $num1 \* 0 + $num2 \* 0 + 1 &>/dev/null || {
  echo "参数错误!"
  echo "Usage: $0 数字1 数字2"
  exit 1
}

# 计算
echo "计算$num1 - $num2 = `awk -vn1=$num1 -vn2=$num2 'BEGIN{print n1 - n2}'`"
echo "计算$num1 + $num2 = `awk -vn1=$num1 -vn2=$num2 'BEGIN{print n1 + n2}'`"
echo "计算$num1 * $num2 = `awk -vn1=$num1 -vn2=$num2 'BEGIN{print n1 * n2}'`"
echo "计算$num1 / $num2 = `awk -vn1=$num1 -vn2=$num2 'BEGIN{print n1 / n2}'`"
```

测试

```shell
[root@mn01[ /server/scripts/devops-shell]#bash 08_num_calc.sh
参数错误!
Usage: 08_num_calc.sh 数字1 数字2
[root@mn01[ /server/scripts/devops-shell]#bash 08_num_calc.sh 10 20
计算10 - 20 = -10
计算10 + 20 = 30
计算10 * 20 = 200
计算10 / 20 = 0.5
```

### 案例2：计算器改写

改成read读取参数

```shell
cat 08_num_calc.sh
#!/bin/bash
##############################################################
# File Name:08_num_calc.sh
# Version:V1.0
# Author:Haris Gong
# Organization:gsproj.github.io
# Desc: 计算器
##############################################################


read -p "请输入数字num1 num2: " num1 num2

# 参数判断
expr $num1 \* 0 + $num2 \* 0 + 1 &>/dev/null || {
  echo "参数错误!"
  echo "Usage: $0 数字1 数字2"
  exit 1
}

# 计算
echo "计算$num1 - $num2 = `awk -vn1=$num1 -vn2=$num2 'BEGIN{print n1 - n2}'`"
echo "计算$num1 + $num2 = `awk -vn1=$num1 -vn2=$num2 'BEGIN{print n1 + n2}'`"
echo "计算$num1 * $num2 = `awk -vn1=$num1 -vn2=$num2 'BEGIN{print n1 * n2}'`"
echo "计算$num1 / $num2 = `awk -vn1=$num1 -vn2=$num2 'BEGIN{print n1 / n2}'`"
```



# 二、Shell逻辑判断

逻辑判断分为：

- 条件表达式：最基本的判断（核心）
- if判断：更加灵活
- case语句：适用于做选择

## 2.1 条件表达式

条件表达式，也叫条件测试语句，属于判断中的核心，if后面都在使用它

目标:

- 熟练掌握条件表达式的格式.
- 熟练使用条件表达式进行判断（文件，大小，与或非）  

### 2.1.1 格式

格式如下

```shell
# 判断指定路径是否存在且为文件
[ -f /etc/hosts ]
# 或者
test -f /etc/hosts
```

#### 面试题：[]和[[]]的区别

| 含义与特点 | test或[]                                  | [[]]或(())                    |
| ---------- | ----------------------------------------- | ----------------------------- |
| 共同点     | 都可以用于判断                            | 都可以用于判断                |
| 区别1      | 不支持正则                                | [[]]支持正则                  |
| 区别2      | 表示逻辑关系(与或非)符号不同 -a -o !  -gt | [[ ]] && \|\| ! > < <= >=     |
| 应用场景   | 常见判断                                  | 使用正则[[]]<br/>进行运算(()) |



### 2.1.2 判断文件

常用如下

| 条件表达式 | 说明                                               |
| ---------- | -------------------------------------------------- |
| **-f**     | 判断指定路径是否存在且为文件，是文件，返回true     |
| **-d**     | 判断指定路径是否存在且为文件夹，是文件夹，返回true |
| -x         | 判断指定路径是否存在且具有可执行权限               |
| -s         | 判断指定文件是否存在且有内容(大小>0)，非空为真     |
| -r         | 是否具有读权限                                     |
| -w         | 是否具有写权限                                     |
| -nt        | newer than 对比两个文件修改时间，是否更新          |
| -ot        | older than 对比两个文件修改时间，是否更老          |
| -L         | 是否是软连接                                       |
| -e         | 是否存在（任何文件类型）                           |

#### 检测：文件是否存在

```shell
[ -f /etc/hosts ] && echo "成立 " || echo "失败"
[ -f /etchosts ] && echo "成立 " || echo "失败"
test -f /etchosts && echo "成立 " || echo "失败"
```

#### 检测：目录是否存在

```shell
[ -d /etc/ ] && echo "成立 " || echo "失败"
成立
[ -d /etc/hosts ] && echo "成立 " || echo "失败"
失败
```

#### 检测：是否有执行权限

```shell
# 检查/etc/rc.d/rc.local是否有执行权限.
[ -x /etc/rc.d/rc.local ] && echo "成立 " || echo "失败"
失败

# 在脚本中的使用
[ -x /sbin/ip ] || exit 1
# ip命令是否有执行权限，如果没有则退出
```

#### 检测：文件是否有内容

```shell
# 创建空文件
[root@mn01[ /tmp]#>test.sh
# 有内容为真（>0），没有内容为假(0)
[root@mn01[ /tmp]#[ -s test.sh ] && echo "有内容" || echo "没内容"
没内容
```

### 2.1.3 对比字符串

用于对比两个字符串的内容

| 对比选项         | 说明                                                  |
| ---------------- | ----------------------------------------------------- |
| "str1" = "str2"  | str1是否等于str2，相等为真                            |
| "str1" != "str2" | str1是否不等于str2，不相等为真                        |
| -z "str"         | zero 检查str字符串是否是空的，空为真                  |
| -n "str"         | no zero 检查str字符串是否不是空的，非空（有内容）为真 |

#### 对比两字符串是否相等

```shell
input=start
[ "$input" = "start" ] && echo "成立 " || echo "失败"
成立
```

应用：判断程序是否是Root执行

```shell
[ "$UID" != "0" ] && exit 4
```

企业级小技巧：在进行字符串比较的时候，变量尾巴加个x，防止变量为空，造成匹配/执行失败

```shell
# 定义变量
str1=qwer
str2=asdf
# 对比字符串，加x
[ "${str1}x" = "${str2}x" ] && echo "成立" || echo "失败"
失败
```

#### 检查字符串是否为空

```shell
# 字符串有内容
[root@mn01[ /tmp]#echo $str2
test

# 判断是否为空
[root@mn01[ /tmp]#[ -z "$str2" ] && echo "变量为空" || echo "变量非空"
变量非空

# 判断是否为非空
[root@mn01[ /tmp]#[ -n "$str2" ] && echo "变量非空" || echo "变量为空"
变量非空
```

### 2.1.4 比大小

常用：

- -eq：equal 等于
- -gt：greater than 大于
- -lt：less than 小于

| 比大小（整数） | []   | [[]] |
| -------------- | ---- | ---- |
| 大于           | -gt  | >    |
| 大于等于       | -ge  | >=   |
| 小于           | -lt  | <    |
| 小于等于       | -le  | <=   |
| 等于           | -eq  | ==   |
| 不等于         | -ne  | !=   |

> 不支持小数对比，仅支持整数

#### []比较

```shell
[ 666 -gt 1 ] && echo "成立 " || echo "失败"
成立
[ 0 -gt -1 ] && echo "成立 " || echo "失败"
成立
[ 0 -gt -1000 ] && echo "成立 " || echo "失败"
成立
[ 0 -gt 0.5 ] && echo "成立 " || echo "失败"
-bash: [: 0.5: 期待整数表达式
失败
```

#### [[]]比较

```shell
[[ 60 > 6 ]] && echo "成立 " || echo "失败"
成立
```

>注意：
>
>不推荐使用>=这种格式，对比的时候会有语法问题
>
>```shell
>[[ 6 >= 6 ]] && echo "成立 " || echo "失败"
>-bash: 条件表达式中有语法错误
>-bash: `6' 附近有语法错误
>```
>
>这里面也可以用 -gt -lt .......
>
>```shell
>[root@mn01[ /tmp]#[[ 6 -ge 6 ]] && echo "成立 " || echo "失败"
>成立
>```



### 2.1.5 逻辑判断

与、或、非

| 逻辑判断 | []        | [[]] |
| -------- | --------- | ---- |
| 与       | -a  (and) | &&   |
| 或       | -o (or)   | \|\| |
| 非       | !         | !    |

基本使用：

```shell
sex=nv
state=china
[ "$sex" = "nv" -a "$guo" = "china" ] && echo 肯定能成 || echo 不一定，还需努力
肯定能成
```



### 2.1.6 正则表达式

初步使用正则

```shell
# 变量中只要有0-9的数字就行
num=666
[[ $num =~ [0-9] ]] && echo 成立 || echo 失败
成立

num=lidao996
[[ $num =~ [0-9] ]] && echo 成立 || echo 失败
成立 

# 开头结尾中间全是数字, 连续数字
[[ $num =~ ^[0-9]+$ ]] && echo 成立 || echo 失败
失败 

num=666
[[ $num =~ ^[0-9]+$ ]] && echo 成立 || echo 失败
成立
```

#### 案例：优化计算器脚本

加入正则判断参数是否为数字

```shell
# 支持负数的正则
"\$num1" =~ ^-?[0-9]+$
# 判断小数正则 
^-?[0-9]+\.?[0-9]+$
```

![image-20240517141734924](../../../img/image-20240517141734924.png)

## 2.2 if判断

应用:if一般与条件表达式一起使用，也可以直接加上命令.

目标:

- if判断适用于更加复杂的判断与检查
- if判断语句的格式  

### 2.2.1 单分支判断，if...then

语法：

```shell
if 条件;then
  满足条件后执行的内容。
fi

# 或者
if 条件
then
  满足条件后执行的内容。
fi
```

基本使用：

```shell
[ $# -eq 2 ] Վҗ {
echo "必须要2个数字"
exit 1
}

# 或者
if [ $# -ne 2 ];then
echo "脚本必须要2个参数"
exit 1
fi
```

### 2.2.2 双分支判断，if..then..else

语法：

```shell
if 条件;then  
  满足条件后执行的内容。
else
  不满足条件执行的内容。
fi
```

案例：检查根分区磁盘空间使用率

```shell
[root@mn01[ /server/scripts/devops-shell]#cat 09_disk_check.sh
#!/bin/bash
##############################################################
# File Name:08_disk_check.sh
# Version:V1.0
# Author:Haris Gong
# Organization:gsproj.github.io
# Desc: 检查根分区磁盘使用率
##############################################################

# 使用率>=97%报警
error=97
# 查询现在的使用率， -w 强匹配
root_usage=`df -h | grep -w '/' | awk -F '[ %]+' '{print $(NF - 1)}'`


# 判断
if [ $root_usage -ge $error ];then
        echo "磁盘空间不足，请尽快删除无用数据"
else
        echo "磁盘空间正常"
fi
```

测试

```shell
[root@mn01[ /server/scripts/devops-shell]#bash 09_disk_check.sh
磁盘空间正常
```

![image-20240517143023303](../../../img/image-20240517143023302.png)

### 2.2.3 多分支判断

语法

```shell
if 条件;then
	满足条件后执行的内容。
elif 条件;then #else if
	满足elif条件，执行的内容。
elif 条件;then
	满足elif条件，执行的内容。
else
	不满足条件执行的内容。
fi
```

案例：

```shell
if [ $num1 -gt $num2 ] ;then
	echo "$num1 大于 $num2"
elif [ $num1 -lt $num2 ] ;then
	echo "$num1 小于 $num2"
else
	echo "$num1 等于 $num2"
fi
```

### 2.2.4 案例-输出指定用户的信息

>温馨提示：这个脚本未来可以用于做安全检查  

步骤:

- 执行脚本输入用户名(参数/read)
- 判断用户是否存在,如果不存在则提示用户不存在,退出脚本.
- 如果用户存在输出用户的信息  
  - 是否可以登录(命令解释器)
  - uid,gid(过滤)
  - 用户家目录
  - 最近1次登录情况  

输出样式：

```shell
用户名: $user
是否可以登录: $if_login
用户UID,GID: $user_ids
用户家目录: $user_homedir
最近的登录情况: $user_login_info
```

实现：

```shell
#!/bin/bash
##############################################################
# File Name:10_check_user.sh
# Version:V1.0
# Author:Haris Gong
# Organization:gsproj.github.io
# Desc: 检查用户是否存在
##############################################################


# 1、输入用户名
read -p "请输入用户名" user

# 2、查寻用户
## 变量不能为空
[ "{user}x" = "x" ] && {
        echo "输入为空，错误！脚本退出"
        exit 1
}

## 查询是否存在
id $user &> /dev/null
if [ $? -ne 0 ];then
        echo "用户${user}不存在"
        exit 1
fi

## 如存在, 查询信息
user_shell=`awk -F: -vname=$user '$1==name{print $NF}' /etc/passwd`
if [ "$user_shell" = "/bin/bash" ];then
        if_login="可以登录"
else
        if_login="无法登录"
fi

## uid、gid、家目录
user_ids=`awk -F: -vname=$user '$1==name{print$3,$4}' /etc/passwd`
user_homedir=`awk -F: -vname=$user '$1==name{print $6}' /etc/passwd`

## 登录
user_login_info=`lastlog |awk -vname=$user '$1==name'`


#3、输出
cat <<EOF
  用户名: $user
  是否可以登录: $if_login
  用户UID,GID: $user_ids
  用户家目录: $user_homedir
  最近的登录情况: $user_login_info
EOF
```

测试

![image-20240517144531642](../../../img/image-20240517144531642.png)



## 2.3 case语句

条件分支语句，一般用于实现有多种选择的脚本。

- 这个功能用if也能实现，不过使用case语句会更加清晰直观
- 如服务的状态：start|stop|restart|status  

```shell
case "变量" in 
	start)
		echo "start"
		;;
	stop)
		echo "start"
		;;
	restart)
		echo "start"
		;;
	*)	# 默认
		echo "Error"
esac
```

>补充：
>
>case的一个选项可以放多种组合，比如
>
>```shell
>case ""$var" in
>	yes|y|Y|Yes|YES  echo "yes!!";;
>```

### 2.2.1 案例1-某会所菜单展示

实现：

```shell
#!/bin/bash
##############################################################
# File Name:11_huisuo.sh
# Version:V1.0
# Author:Haris Gong
# Organization:gsproj.github.io
# Desc:
##############################################################

#1.输出提示
cat<<EOF
##############olding私人高级会所#####################
####################套餐############################
输入1,选择 138套餐) 吃饱套餐
输入2,选择 443套餐) 吃饱喝足套餐
输入3,选择 888套餐) 吃喝拉撒套餐
输入4,选择 1688套餐) 你想干啥就干啥套餐
输入其他内容,退出
EOF

# 2.vars
read -p "请输入你的选择：" num

# 3、判断
case "$num" in
        1)      echo "138套餐) 吃饱套餐" ;;
        2)      echo "443套餐) 吃饱喝足套餐";;
        3)      echo "888套餐) 吃喝拉撒套餐";;
        4)      echo "1688套餐) 你想干啥就干啥套餐";;
        tomcat) echo "8080套餐) 恭喜你获得隐藏套餐 不做人套餐";;
        oldgirl) echo "8443套餐) 恭喜你获得隐藏套餐 老板娘套餐";;
        *)
                echo "退出"
                exit 1
esac
```

测试

![image-20240517150044293](../../../img/image-20240517150044292.png)

### 2.2.2 案例2-判断用户输入yes还是no

实现

```shell
[root@mn01[ /server/scripts/devops-shell]#cat 12_yesorno.sh
#!/bin/bash
##############################################################
# File Name:12_yesorno.sh
# Version:V1.0
# Author:Haris Gong
# Organization:gsproj.github.io
# Desc:
##############################################################

read -p "请输入yes或者no：" var

case "$var" in
        yes|y|Y|YES|Yes)  echo "Yes!!";;
        no|n|N|NO|No) echo "No!!!";;
        *)
                echo "Error!"
                exit 1
esac
```



