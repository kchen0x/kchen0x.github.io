title: Linux 常用功能性命令
date: 2018/6/13 13:42:00
tag:
	- Linux
	- Tips

## 统计当前目录下特定格式文件的数目

```bash
find . -name "*.wav" -print | wc -l
```

解析：`find .` 命令可以循环查找当前目录和子目录的文件，`-name` 参数用于指定文件 pattern，`wc -l` 按行统计结果。

## 统计当前文件夹下的文件个数、目录个数

```bash
ls -l |grep "^-"|wc -l      ##统计当前文件夹下文件的个数
ls -l |grep "^d"|wc -l      ##统计当前文件夹下目录的个数
ls -lR|grep "^-"|wc -l      ##统计当前文件夹及子文件夹下文件的个数
ls -lR|grep "^d"|wc -l      ##统计文件夹及子文件夹下目录的个数
```

解析：使用 `ls -l` 命令用长列表模式查看当前目录的文件（包括目录、设备文件等）`R` 参数递归子目录，`grep` 命令过滤内容，`^d` 是文件夹（目录），`^-` 是普通文件。

## 检测本机的端口是否被占用

```bash
# 查看已开放的 tcp 端口
netstat -anp tcp

# 查看指定端口是否被占用
netstat –apn | grep 8080
```

## 检测目标机器 tcp 端口是否开放

```bash
telnet www.google.com 80
```

## 使用 firewall-cmd 管理防火墙

```bash
firewall-cmd --state                           ##查看防火墙状态，是否是running
firewall-cmd --reload                          ##重新载入配置
firewall-cmd --get-zones                       ##列出支持的zone
firewall-cmd --get-services                    ##列出支持的服务，在列表中的服务是放行的
firewall-cmd --query-service ftp               ##查看ftp服务是否支持，返回yes或者no
firewall-cmd --add-service=ftp                 ##临时开放ftp服务
firewall-cmd --add-service=ftp --permanent     ##永久开放ftp服务
firewall-cmd --remove-service=ftp --permanent  ##永久移除ftp服务
firewall-cmd --add-port=80/tcp --permanent     ##永久添加80端口 
iptables -L -n                                 ##查看规则，这个命令是和iptables的相同的
man firewall-cmd                               ##查看帮助
```

## GPU 使用情况查询

```bash
nvidia-smi      ##查看 GPU 此时的使用情况
gpustat         ##更加精简的结果（pip install gpustat）
```

## Bash 脚本中的循环控制

**1.基本格式**

```bash
for 变量 in 取值列表
do
    各种操作
done
```

还有**罕见**的写法就是都写作一行里：

```bash
for 变量 in 取值列表 ; do 各种操作 ;done
```

**2.普通枚举**

取值列表为空格或回车符分割的字符串

```bash
for i in 'apple' 'meat' 'sleep' 'woman'
do
    echo I like $i
done
```

**3.花括号迭代**

* 数字迭代，比如`{1..100}`
* 字母迭代，比如`{a..z},{A..Z},{Z..A}`
* ASCII字符迭代，比如`{a..A}`

计算1加到100的和

```bash
#!/bin/bash
ans=0
for i in {1..100}
do
    let ans+=$i
done
echo $ans
```

花括号的迭代还可以指定指定增量，格式如下：

```bash
{首..尾..增量}  
```

计算一下1到100以内的所有奇数的和：

```bash
for i in {1..100..2}
do
    echo $i
done
```

**4.C 风格 for 循环**

```bash
#!/bin/bash
ans=0
for ((i=1;i<=100;i++))
do
    let ans+=$i
done
echo $ans
```

> **注意!!!这里的for循环要有两层括号**
