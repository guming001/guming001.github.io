# Linux 命令学习收录笔记

#### 常用系统工作命令
- **echo命令**：在终端输出字符串或变量提取后的值
- **date命令**：显示及设置系统的时间或日期
- **reboot命令**：重启系统
- **poweroff命令**：关闭系统
- **wget命令**：在终端中下载网络文件
- **ps命令**：查看系统中的进程状态
  - **R（运行）**：进程正在运行或在运行队列中等待。
  - **S（中断）**：进程处于休眠中，当某个条件形成后或者接收到信号时，则脱离该状态。
  - **D（不可中断）**：进程不响应系统异步信号，即便用kill命令也不能将其中断。
  - **Z（僵死）**：进程已经终止，但进程描述符依然存在, 直到父进程调用wait4()系统函数后将进程释放。
  - **T（停止）**：进程收到停止信号后停止运行。
- **top命令**：动态地监视进程活动与系统负载等信息
- **pidof命令**：查询某个指定服务进程的PID值
- **kill命令**：终止某个指定PID的服务进程
- **killall命令**：终止某个指定名称的服务所对应的全部进程

#### 系统状态检测命令
- **ifconfig命令**：获取网卡配置与网络状态等信息
- **uname命令**：查看系统内核与系统版本等信息
- **uptime命令**：查看系统的负载信息（同top命令第一行）
- **free命令**：显示当前系统中内存的使用量信息（同top命令末尾两行）
- **who命令**：查看当前登入主机的用户终端信息
- **last命令**：查看所有系统的登录记录
- **history命令**：显示历史执行过的命令
- **sosreport命令**：用于收集系统配置及架构信息并输出诊断文档

#### 工作目录切换命令
- **pwd命令**：用于显示用户当前所处的工作目录
- **cd命令**：用于切换工作路径
- **ls命令**：用于显示目录中的文件信息

#### 文本文件编辑命令
- **cat命令**：用于查看纯文本文件（内容较少的）
- **more命令**：用于查看纯文本文件（内容较多的）
- **head命令**：用于查看纯文本文档的前N行
- **tail命令**：用于查看纯文本文档的后N行或持续刷新内容
- **tr命令**：用于替换文本文件中的字符
- **wc命令**：用于统计指定文本的行数、字数、字节数
- **stat命令**：用于查看文件的具体存储信息和时间等信息
- **cut命令**：用于按“列”提取文本字符 

| 参数 | 作用 |
| :---: | :--- |
| -f | 设置需要看的列数 |
| -d | 设置间隔符号 |

`cut -d: -f1 /etc/passwd`
  
- **diff命令**：比较多个文本文件的差异

#### 文件目录管理命令
- **touch命令**：用于创建空白文件或设置文件的时间

| 参数 | 作用 |
| :---: | :--- |
| -a | 仅修改“读取时间”（atime）|
| -m | 仅修改“修改时间”（mtime）|
| -d | 同时修改atime与mtime |

`touch -d "2017-05-04 15:44" anaconda-ks.cfg`

- **mkdir命令**：用于创建空白的目录 

> ***-p***:递归创建

`mkdir -p a/b/c/d/e`

- **cp命令**：用于复制文件或目录

| 参数 | 作用 |
| :---: | :--- |
| -p | 保留原始文件的属性 |
| -d | 若对象为“链接文件”，则保留该“链接文件”的属性 |
| -r | 递归持续复制（用于目录） |
| -i | 若目标文件存在则询问是否覆盖 |
| -a | 相当于-pdr（p、d、r为上述参数）|

- **mv命令**：用于剪切文件或将文件重命名
- **rm命令**：用于删除文件或目录
- **dd命令**：用于按照指定大小和个数的数据块来复制文件或转换文件 

| 参数 | 作用 |
| :---: | :--- |
| if | 输入的文件名称 |
| of | 输出的文件名称 |
| bs | 设置每个“块”的大小 |
| count | 设置要复制“块”的个数 |

`dd if=/dev/zero of=560_file count=1 bs=560M`

- **file命令**：用于查看文件的类型

#### 打包压缩与搜索命令
- **tar命令**：用于对文件进行打包压缩或解压

| 参数 | 作用 |
| :---: | :--- |
| -c | 创建压缩文件 |
| -x | 解开压缩文件 |
| -t | 查看压缩包内有哪些文件 |
| -z | 用Gzip压缩或解压 |
| -j | 用bzip2压缩或解压 |
| -v | 显示压缩或解压的过程 |
| -f | 目标文件名 |
| -p | 保留原始的权限与属性 |
| -P | 使用绝对路径来压缩 |
| -C | 指定解压到的目录 |

- **grep命令**：用于在文本中执行关键词搜索，并显示匹配的结果

| 参数 | 作用 |
| :---: | :--- |
| -b | 将可执行文件（binary）当作文本文件（text）来搜索 |
| -c | 仅显示找到的行数 |
| -i | 忽略大小写 |
| -n | 显示行号 |
| -v | 反向选择——仅列出没有“关键词”的行 |

`grep /sbin/nologin /etc/passwd`

- **find命令**：用于按照指定条件来查找文件

`find / -user linuxprobe -exec cp -a {} /root/findresults/ \;`


