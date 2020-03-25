# shell
```
文件比较运算符
-e filename 	如果 filename存在，则为真 	[ -e /var/log/syslog ]
-d filename 	如果 filename为目录，则为真 	[ -d /tmp/mydir ]
-f filename 	如果 filename为常规文件，则为真 	[ -f /usr/bin/grep ]
-L filename 	如果 filename为符号链接，则为真 	[ -L /usr/bin/grep ]
-r filename 	如果 filename可读，则为真 	[ -r /var/log/syslog ]
-w filename 	如果 filename可写，则为真 	[ -w /var/mytmp.txt ]
-x filename 	如果 filename可执行，则为真 	[ -L /usr/bin/grep ]
filename1-nt filename2 	如果 filename1比 filename2新，则为真 	[ /tmp/install/etc/services -nt /etc/services ]
filename1-ot filename2 	如果 filename1比 filename2旧，则为真 	[ /boot/bzImage -ot arch/i386/boot/bzImage ]
字符串比较运算符 （请注意引号的使用，这是防止空格扰乱代码的好方法）
-z string 	如果 string长度为零，则为真 	[ -z "$myvar" ]
-n string 	如果 string长度非零，则为真 	[ -n "$myvar" ]
string1= string2 	如果 string1与 string2相同，则为真 	[ "$myvar" = "one two three" ]
string1!= string2 	如果 string1与 string2不同，则为真 	[ "$myvar" != "one two three" ]
算术比较运算符
num1-eq num2 	等于	[ 3 -eq $mynum ]
num1-ne num2 	不等于	[ 3 -ne $mynum ]
num1-lt num2 	小于	[ 3 -lt $mynum ]
num1-le num2 	小于或等于	[ 3 -le $mynum ]
num1-gt num2 	大于	[ 3 -gt $mynum ]
num1-ge num2 	大于或等于	[ 3 -ge $mynum ]
```
+ rm -rf * 这个命令的意思是:删除当前目录下的所有文件
+ if-then-fi,fi表示if结束
+ tar zxcf解压
  + 分别是四个参数
  + x : 从 tar 包中把文件提取出来
  + z : 表示 tar 包是被 gzip 压缩过的，所以解压时需要用 gunzip 解压
  + v : 显示详细信息
  + f xxx.tar.gz : 指定被处理的文件是 xxx.tar.gz
+ cp source_file ... target_directory

#### mv 移动
+ 把文件 source_file 移动到 target_file，实际的意思就是重命名，其他不变，例如 inode 信息，最近修改时间点等等不变。
```
    mv source_file target_file
```
+ 如果文件 target_file 已经存在呢，可以加上 -i 参数，系统会提示是否覆盖， 也可以加上 -n 参数，不让覆盖的行为发生。
+ 文件到目录的移动
```
     mv source_file target_directory
```
+ 目录到目录的移动
```
     mv source_directory target_directory
```
+ 这就要分为两类 
  + 如果target_directory不存在，这就相当于目录重全名。 
  + 如果target_directory存在，就会把整个 source_directory 目录移动到 target_directory 目录中，相当于剪切整个目录，然后粘贴。
#### bash命令 [ $? -eq 0 ]
+ $? 是指上一条命令的执行状态， 0就是正常
+ -z 判断 变量的值，是否为空； zero = 0
  
  + \- 变量的值，为空，返回0，为true
  + \- 变量的值，非空，返回1，为false
  + -n 判断变量的值，是否为空   name=名字
  + \- 变量的值，为空，返回1，为false
  + \- 变量的值，非空，返回0，为true
+ 现在git上的代码已经要求迁回svn了，后续code review没git方便，现在看svn的code review是结合了Review Board，主要流程是在33或36上

# jps
+ 查看java进程PID


# nginx
+ Linux环境下，怎么确定Nginx是以那个config文件启动的？
```
    输入命令行： ps  -ef | grep nginx
```
+ su - root 切换到root用户 36 12345678
+ 访问地址为对应小机的ip，可以检测nginx是否启动
+ 重新刷新config cd 到 nginx sbin目录 
```
./nginx -s reload
```


# linux
+ ps -ef | grep control ps命令将某个进程显示出来/grep命令是查找/中间的|是管道命令 是指ps命令与grep同时执行
+ lsof -i :3306/列出谁在使用某个端口 获取pid
+ grep 、tail 加-v 排除某些字符 使用正则
+ 根据pid获取程序位置ps -aux |grep 28990  执行cd /proc/$pid   执行ls -ail 
```
【命令格式】

grep [option] "string_to_find" filename

常见选项：

（1）-i：忽略搜索字符串的大小写

（2）-v：取反，即输出不匹配的那些文本行

（3）-n：输出行号

（4）-l：输出能够匹配模式的文件名，相反的选项为-L

（5）-q：静默输出

 (6) -a: 输出所在行
 
 -c: 统计次数

选项是可选的，根据实际需求进行选择即可

string_to_find为需要匹配的模式，可以填写字符串或者正则表达式

filename为需要查找的文件的名称
```
+ 装了Glibc的rpm软件包，把linux系统搞崩了
+ glibc是linux系统中最底层的api，几乎其他任何运行库都会依赖glibc，不要在运行中的系统安装Glibc，否则导致系统崩溃，至少应当将新Glibc安装到其他单独目录，以保障不覆盖当前正在使用的Glibc
  
+ 执行时代码里有bash xx.sh 报文件找不到，实际上是编码格式不对
+ linux下，通过vim进入，输入命令：set ff查看编码格式，如果为dos，改成unix
+ set ff=unix修改编码格式
+ notepad++ 可以在右下角编码那里点击，改为UNIX编码格式
+ rm -f 无需确认删除文件 rm -rf 删除文件夹
+ su root 管理员
+ /etc/profile 调整环境变量
+ 执行shell脚本权限不足 chmod u+x *.sh
