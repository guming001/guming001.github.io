# Shell脚本、流程控制以及计划任务
### 编写Shell脚本
#### 编写简单的Shell脚本
```
#!/bin/bash 
#For Example BY linuxprobe.com 
pwd 
ls -al
```
第一行的脚本声明（**#!**）用来告诉系统使用哪种Shell解释器来执行该脚本；

第二行的注释信息（**#**）是对脚本功能和某些命令的介绍信息，使得自己或他人在日后看到这个脚本内容时，可以快速知道该脚本的作用或一些警告信息；

第三、四行的可执行语句也就是我们平时执行的Linux命令了。

#### 接收用户的参数

| 参数 | 含义 |
| :---: | :--- |
| $0 | 当前Shell脚本程序的名称 |
| $# | 总共有几个参数 |
| $* | 所有位置的参数值 |
| $? | 上一次命令的执行返回值 |
| $1、$2、$3…… | 分别对应着第N个位置的参数值 |

```
[root@linuxprobe ~]# vim example.sh
#!/bin/bash
echo "当前脚本名称为$0"
echo "总共有$#个参数，分别是$*。"
echo "第1个参数为$1，第5个为$5。"
[root@linuxprobe ~]# sh example.sh one two three four five six
当前脚本名称为example.sh
总共有6个参数，分别是one two three four five six。
第1个参数为one，第5个为five。
```

#### 判断用户的参数
> 测试语句格式：`[ 条件表达式 ]`
> **切记，条件表达式两边均应有一个*空格*。**

Shell脚本中的条件测试语法可以判断表达式是否成立，若条件成立则返回数字**0**，否则便返回其他随机数值。

- **文件测试语句**

*文件测试所用的参数*

| 运算符 | 作用 |
| :---: | :--- |
| -d | 测试文件是否为目录类型 |
| -e | 测试文件是否存在 |
| -f | 判断是否为一般文件 |
| -r | 测试当前用户是否有权限读取 |
| -w | 测试当前用户是否有权限写入 |
| -x | 测试当前用户是否有权限执行 |
    
- **逻辑测试语句**

  - 逻辑"与"，运算符号是&&，它表示当前面的命令执行成功后才会执行它后面的命令

    ```
    [root@linuxprobe ~]# [ -e /dev/cdrom ] && echo "Exist"
    Exist
    ```

  - 逻辑"或"，运算符号为||，表示当前面的命令执行失败后才会执行它后面的命令

    ```
    [root@linuxprobe ~]# echo $USER
    root
    [root@linuxprobe ~]# [ $USER = root ] || echo "user"
    [root@linuxprobe ~]# su - linuxprobe 
    [linuxprobe@linuxprobe ~]$ [ $USER = root ] || echo "user"
    user
    ```
    
  - 逻辑"非"，运算符号是一个叹号（！），它表示把条件测试中的判断结果取相反值

    ```
    [linuxprobe@linuxprobe ~]$ exit
    logout
    [root@linuxprobe root]# [ ! $USER = root ] || echo "administrator"
    administrator
    ```
- **整数值比较语句**

*可用的整数比较运算符*

| 运算符 | 作用 |
| :---: | :--- |
| -eq | 是否等于 |
| -ne | 是否不等于 |
| -gt | 是否大于 |
| -lt | 是否小于 |
| -le | 是否等于或小于 |
| -ge | 是否大于或等于 |

- **字符串比较语句**

*常见的字符串比较运算符*

| 运算符 | 作用 |
| :---: | :--- |
| = | 比较字符串内容是否相同 |
| != | 比较字符串内容是否不同 |
| -z | 判断字符串内容是否为空 |

### 流程控制语句
#### if条件测试语句
- if单分支结构

```
if 条件测试操作
  then 命令序列
fi
```

- if双分支结构

```
if 条件测试操作
  then 命令序列1
  else 命令序列2
fi
```

- if多分支结构

```
if 条件测试操作1
  then 命令序列1
elif 条件测试操作2
  then 命令序列2
else
  命令序列3
fi
```

***例子***

```
#!/bin/bash
read -p "Enter your score（0-100）：" GRADE
if [ $GRADE -ge 85 ] && [ $GRADE -le 100 ] ; then
echo "$GRADE is Excellent"
elif [ $GRADE -ge 70 ] && [ $GRADE -le 84 ] ; then
echo "$GRADE is Pass"
elif [ $GRADE -ge 0 ] && [ $GRADE -le 69 ] ; then
echo "$GRADE is Fail" 
else
echo "Error"
fi
```

#### for条件循环语句

```
for 变量名 in 取值列表
do
  命令序列
done
```

***例子***

```
#!/bin/bash
HLIST=$(cat ~/ipadds.txt)
for IP in $HLIST
do
ping -c 3 -i 0.2 -W 3 $IP &> /dev/null
if [ $? -eq 0 ] ; then
echo "Host $IP is On-line."
else
echo "Host $IP is Off-line."
fi
done
```

#### while条件循环语句

```
while 条件测试操作
do
  命令序列
done
```

***例子***

```
#!/bin/bash
PRICE=$(expr $RANDOM % 1000)
TIMES=0
echo "商品实际价格为0-999之间，猜猜看是多少？"
while true
do
read -p "请输入您猜测的价格数目：" INT
let TIMES++
if [ $INT -eq $PRICE ] ; then
echo "恭喜您答对了，实际价格是 $PRICE"
echo "您总共猜测了 $TIMES 次"
exit 0
elif [ $INT -gt $PRICE ] ; then
echo "太高了！"
else
echo "太低了！"
fi
done
```

#### case条件测试语句

```
case 变量值 in
模式 1）
  命令序列1
  ;;
模式 2）
  命令序列2
  ;;
  ......
*）
  默认命令序列
esac
```

***例子***

```
#!/bin/bash
read -p "请输入一个字符，并按Enter键确认：" KEY
case "$KEY" in
[a-z]|[A-Z])
echo "您输入的是 字母。"
;;
[0-9])
echo "您输入的是 数字。"
;;
*)
echo "您输入的是 空格、功能键或其他控制字符。"
esac
```

### 计划任务服务程序
- 一次性计划任务

> ***at 时间*** （创建任务）
> ***at -l*** （查看未执行的任务）
> ***atrm 任务序号***（删除任务）

`[root@linuxprobe ~]# echo "systemctl restart httpd" | at 23:30`

- 长期性计划任务

> ***crontab -e*** 创建、编辑计划任务的命令
> ***crontab -l*** 查看当前计划任务的命令
> ***crontab -r*** 删除某条计划任务的命令
> ***crontab -u*** 以管理员的身份，编辑他人的计划任务

| 分 | 小时 | 日 | 月 | 星期 | 命令 |
| --- | --- | --- | --- | --- | --- |
| 50 | 3 | 2 | 1 | * | run command |

**需要注意的是，如果有些字段没有设置，则需要使用星号（\*）占位**

```
25 3 * * 1,3,5 /usr/bin/tar -czvf backup.tar.gz /home/wwwroot
0 1 * * 1-5 /usr/bin/rm -rf /tmp/*
```

**注意事项**
- 在crond服务的配置参数中，可以像Shell脚本那样以#号开头写上注释信息，这样在日后回顾这段命令代码时可以快速了解其功能、需求以及编写人员等重要信息。
- 计划任务中的“分”字段必须有数值，绝对不能为空或是\*号，而“日”和“星期”字段不能同时使用，否则就会发生冲突。
