title: 日常Linux操作总结
date: 2022-01-27 15:58:57
tags:
- Linux
- Shell
categories:
- Linux
cover: https://tianmy.oss-cn-shanghai.aliyuncs.com/img/Linux.jpg
---

# 常用场景

包括：

- 1、查看Linux的版本、cpu、内存、磁盘等信息
- 2、查看磁盘占用，查找文件和目录，处理大文件
- 3、处理日志
- 4、端口、进程相关操作
- 5、crontab定时任务
- 6、自定义快捷命令
- 7、常见环境配置：防火墙、免登陆、yum、时间、JDK等
  

## 1.查看基础信息
```shell
# 查看CentOS系统版本号
cat /etc/centos-release
# 查看系统内核
uname -r
# 查看系统位数
getconf LONG_BIT
```
```shell
# 查看cpu和内存，磁盘等信息

```
## 2.处理磁盘、大文件
### 2-1.使用的命令
#### find命令查找目录和文件
```shell
# 常用参数：
-type: 文件类型，d 目录 f 文件
-name: 名称 通配符："*.*" 文件名称:文件后缀
-iname: 名称不区分大小写
-size: 文件/目录大小
# 执行参数：
-exec： find的结果作为exec的入参
-ok：功能同-exec，区别是在请求之前询问是否执行，输入y执行
# 查询所有目录下所有大于50M的文件
find / -size +50M -exec ls -lh {} \;
# 查找所有目录下jmx类型的文件
find / -type f -name "*.jmx" -exec ls -lh {} \;
# 查找指定目录下文件名为test的文件，不区分大小写
find /{dir} -type f -iname "test" -exec ls -lh {} \;
# 查找指定目录下不为test的文件
find /{dir} -type f !-name "test" -exec ls -lh {} \;
```
##### 与正则配合使用
```
#

```

#### du命令查看目录和文件的磁盘使用空间
```shell
# 常用参数：
-k: 以KB单位显示
-m: 以MB单位显示
-h: 以K M G为单位显示，提高可读性
--max-depth: 显示层级
# 查看/opt目录下的目录和文件大小并按文件从大到小排序
du -h --max-depth=1 /opt/ | sort
```
#### df命令查看磁盘中的可用空间
```shell
# 常用参数
-a: 查看所有文件系统的磁盘占用，单位默认KB
-h: 以K M G为单位显示，注意：这里的1k=1000byte,1m=1000k
```
#### sort排序
```shell
#常用参数：
-u: 排序后去除重复行
-n: 按照数值排序
-r: 按降序排序，sort默认升序排序
-t: 指定分隔符
-k: 指定列数，与-t配合使用
#当前目录下的文件按照文件大小排序
ls -lh | sort -n -k 5
```
#### uniq去重
```shell
#常用参数
-c: 去重并在第一列显示每一行重复的次数
```

### 2-2.场景
#### 1、当前磁盘满了，需要定位大文件并删除
```shell
#1、查看自盘占用
df -h
#2、进入磁盘占用大的目录找到大文件
du -h --max-depth=1 /opt/ | sort
```
#### 2、找到/test目录下所有大于1G的.log文件删除
```shell
find /test -name "*.log" -size +1024M -exec rm -rf {} \;
```

## 3.处理日志

包括：

- 1、日志内容筛选与编辑;正则匹配，命令行高亮显示
- 2、日志文件切割与批处理
  

**使用到的命令：split cat grep awk sed**

### 3-1.使用的命令
#### split命令切割文件
```shell
#用法
split [-b ][-C ][-][-l ][要切割的文件][输出文件名前缀][-a ]
#常用参数：
-b: 指定拆分文件大小，也可以指定 K、M、G、T 等单位
-l: 指定每多少行要拆分成一个文件
-d: 使用数字作为拆分文件的后缀
-a: 指定后缀长度
# 将test.log拆分成每个文件大小不超过1M，且命名采用test_01,test_02...的文件
split -b 1M -d test.log test_
# 将test.log拆分成每个文件不超过100行，且命名采用test_01,test_02...的文件
split -l 100 -d test.log test_
```
#### tail命令分析日志文件
```shell
#常用参数：
-f: 当文件增长时，持续输出
-n: 输出文件最后n行，而不是默认的最后10行
--pid: 与-f结合使用，在PID终止后结束 --
-q: 不输出文件名
-s: 与-f结合使用，-s=n：每次输出间隔n秒
-v: 输出文件名
# 输出test.log
tail -f test.log
# 输出test.log和error.log
tail -f test.log error.log
# 输出test.log中最后200行的内容
tail -n 200 test.log
```
#### 与head、grep、正则的配合使用

```shell
#head常用参数
-n: 显示起始的n行，非默认的10行
-v: 显示文件名
-q: 不显示文件名
#grep常用参数
-v: 不包含匹配的所有行
-c: 计算符合匹配的列数
-w: 只显示完整匹配的结果，不包括部分匹配，比如匹配要求是apple，则只有准确地为apple的内容会返回
-e: 后面接正则条件，一个参数只能加一个条件
-E: 后面接正则条件，一个参数可以加多个条件
-i: 忽略大小写
-o: 只显示匹配pattern的部分
-n: 输出内容时添加结果所在的行号
-r: 递归匹配
-color: -color=auto / always / never;
#a,b的意义分别表示加颜色的方式和颜色值
export GREP_COLOR='a;b'
#a取值范围:【0,1,4,5,7,8】 0关闭所有属性; 1设置高亮度; 4下划线; 5闪烁; 7反显; 8消隐; 
#b取值范围:【30-37或40-47】30 black 31 red 32 green 33 yellow 34 blue 35 purple 36 cyan 37 white 30—37 设置前景色 40—47 设置背景色
-A: 打印出紧随匹配的行之后的下文 NUM 行
-B: 打印出匹配的行之前的上文 NUM 行
-C: 打印出匹配的行的上下文前后各 NUM 行

#pattern的常用参数

# 输出test.log的第100行到第200行
cat test.log | head -n 200 | tail -n 100
# 持续输出test.log中包含”Listener“的行
tail -f jmeter.log | grep "Listener"
```

#### awk处理列数据
```shell
#用法 pattern：匹配内容；action：找到匹配后对每一行执行的操作
awk '{pattern + action}' {filenames}
#内置变量：
$0: 整行
$1~n: 第n列
NF: 字段数量
NR: 行数
FS: -F 分隔字段符号，可以传一个或多个
BEGIN和END模块
#输出test.log的第100行到第200行使用awk实现
awk '{if(NR>100 && NR<200)print $0 }' test.log
#输出test.log中带有https的行中第二列是200的内容
cat test.log | grep -w 'https' | awk '{if ($2 == 200)print}'
#统计test.log中带有https的行中第二列是200的数量
## 注意在加入BEGIN和END语句之后，不要忘记分号
cat test.log | grep -w 'https' | awk '{count++;if ($2 == 200);} END{print "total 200 is",count}'
# 

```
#### sed处理行数据、替换数据
```shell
#常用参数：

```

### 3-2.场景

#### 1、匹配输出test.log中带有https的行并高亮显示
```shell
# 这里grep使用-w参数为了过滤带有http的行，只匹配完全是https的行
tail -f test.log | grep -w "https" --color=auto
```
#### 2、统计Nginx日志中访问量前10的IP
```shell
# 注意参数-n，不使用-n参数会得到错误的结果

awk '{a[$1]++} END{for(i in a)print i,a[i]}' access.log | sort -n -r -k 2 | head -10 
```
#### 3、统计Nginx日志中访问超过10次的IP
```shell
awk '{a[$1]++} END{for(i in a){if(a[i]>10)print i,a[i]}}' access.log | sort -nr -k 2 
```
#### 4、统计每个IP访问结果状态码的数量
```shell
#方法一，使用awk统计
awk '{a[$1" "$13]++}END{for(i in a)print i,a[i]}' access.log | sort -r
#方法二，使用uniq做去重和统计
awk '{print $1 " "$13}' access.log | sort -r | uniq -c
```
## 4.端口、进程操作
## 5.crontab定时任务
## 6.自定义快捷命令

编写shell函数的sh文件，将路径放在当前shell的环境变量下，即可在命令行中通过命令的形式调用函数
## 7.其他常规操作