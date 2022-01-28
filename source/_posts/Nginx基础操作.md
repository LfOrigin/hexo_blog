title: Nginx基础操作
date: 2022-01-28 16:38:57
tags:
- Nginx
- 工具
categories:
- Nginx
cover: https://tianmy.oss-cn-shanghai.aliyuncs.com/img/nginx.jpg
---
```shell
#快速启动一个Nginx
docker run --name nginx-learn -p 80:80 -d nginx
```
### 1.常用命令
```shell
#常用命令
-T: 查看当前nginx的配置
-s: 向master进程发送指令: 
    stop: 关闭
    quit: 当前工作处理完成后关闭
    reopen: 重启
    reload: 重载配置文件，热重启
```
### 2.语法
- 每条指令以 ; 结尾，指令与参数之间以空格区隔
- 指令块放在 {} 中
- include 允许引用外部的配置文件
- 使用 # 符号添加注释
- 使用 $ 符号使用变量
- 部分指令的参数支持正则表达式

### 3.全局变量
```shell
#请求信息中的 Host，如果请求中没有 Host 行，则等于设置的服务器名，不包含端口
$host
#客户端请求类型，如 GET、POST
$request_method
#客户端的 IP 地址
$remote_addr
#请求中的参数
$args
#请求中变量名 PARAMETER 参数的值
$arg_PARAMETERGET
#请求头中的 Content-length 字段
$content_length
#客户端agent信息
$http_user_agent
#客户端cookie信息
$http_cookie
#客户端的IP地址
$remote_addr
#客户端的端口
$remote_port
#客户端agent信息
$http_user_agent
#请求使用的协议，如 HTTP/1.0、HTTP/1.1
$server_protocol
#服务器地址
$server_addr
#服务器名称
$server_name
#服务器的端口号
$server_port
#方法（如http，https）
$schemeHTTP 
```
### 4.配置文件
nginx.conf示例：
```shell
#全局块，此处配置全局生效
daemon on;
user nobody;
work_process 2;
pid logs/nginx.pid;
error_log logs/error.log debug;
#events块
events {
    #此处是处理连接的配置
    #此处配置影响性能
    worker_connections 1024;
}
#http块
http {
    # 最常用部分，代理、缓存、日志等配置  
    include test.conf;
    keepalive_timeout = 10;
    #server块
    server {
        # 监听端口
        listen 80
        # 域名配置
        server_name localhost
        #location块
        location /one {
            # 配置请求路径是'/one'的代理
        }
        #同一个server块下可以有多个location块
        location /two {
            # 配置请求路径是'/two'的代理
        }
    } 
    #同一个http块下可以有多个server块
    server {
        # 配置另一个服务的代理
    }
}
```
#### 4-1.全局块
一般位于events块之前，通常配置内容包括：
##### daemon
```shell
#以守护模式运行Nginx
daemon on;
```
##### user
```shell
#Nginx的用户
#用法
user [用户名] [用户组]
#示例: 允许root组的admin用户访问
user admin root
## user在Windows中不生效
user nobody 或者 # user xx 代表所有用户都可以运行
```
##### work_process
```shell
#配置工作线程数，一般与服务器的CPU核数保持一致
#用法
worker_process [线程数] | auto
#示例: 指定2个线程
worker_process 2;
```
##### pid
```shell
#Nginx 主进程Pid的记录位置
#用法
pid [路径]
#示例
pid logs/nginx.pid
```
##### error_log
```shell
#日志保存路径
#用法
error_log [路径] [日志级别,常用的级别：debug|info|error|warn]
#示例
error_log logs/error.log error
error_log logs/info.log info
```
#引入的配置文件

#### 4-2.events块
##### worker_connections
```shell
#配置单个worker的最大连接数
worker_connections 2000;
```
##### 工作模式
```shell
use epoll|select|poll|kqueue...
```

#### 4-3.http块
##### client_max_body_size
```shell

```
##### keepalive_timeout
```shell

```
#### 负载均衡
#### 动静分离
#### 高可用
### 最佳实践参考
### 使用注意
#### 1.路径配置
```text
# 例子1
server {
    listen 8080;
    server_name 192.168.0.222;
    
    location /api {
                proxy_pass   http://192.168.1.123:9000;  
            }
        
# 例子2
server {
    listen 8080;
    server_name 192.168.2.222;
    
    location /api {
                proxy_pass   http://192.168.1.123:9000/;  
            }
```
**注意location后面路径的 / 和proxy_pass路径的 /**

```text
#例子1：
#请求：
http://192.168.0.222:8080/api/user/login
#nginx处理后的请求：
http://192.168.1.123:9000/api/user/login
#例子2：
#请求：
http://192.168.0.222:8080/api/user/login
#nginx处理后的请求：
http://192.168.1.123:9000//user/login
```
例子1与例子2的区别是例子2中的proxy_pass除了IP和Port外，还配置了上下文，这里的规则是：
1、如果proxy_pass中的路径存在上下文(即端口后存在/)：
- 替换IP和Port；
- 将location中的路径替换为上下文；
- 拼接余下的路径；

2、如果proxy_pass中的路径不存在上下文(即端口后没有/及后续路径)：
- 替换IP和Port，其余不动；