---
title: 使用frp渗透公司内网中的本机虚拟机服务器
date: 2019-09-29 10:05:11
tags: 工具使用
---

# 序言

由于公司配了一条新的8G内存条，之前开虚拟机都会导致本机内存满掉，这次终于可以愉快的开虚拟机了:smirk:，虽然手上有三台服务器，搬瓦工和Google cloud 还有阿里云各一台，但是都是最低配置的，没法做集群，而且没法开多个服务，所以决定将电脑的centos虚拟机当成一台服务器，但是公司做了很多网络限制，例如网盘，网易云，bilibili等很多网站都被做了限制，以及外网根本连不进去公司的电脑，所以只能通过代理的形式，目前使用shadowsocks 全局代理形式访问被公司封禁的网站，使用frp / ngrok和 teamview 连进公司的电脑，teamview 安装Linux的会出现很多问题，而且只能进行ssh 连接，不能进行端口转发，所以这次我们使用frp来进行端口转发，外网访问内网的虚拟机的rabbitMQ服务

# FRP简介

> frp 是一个可用于内网穿透的高性能的反向代理应用，支持 tcp, udp 协议，为 http 和 https 应用协议提供了额外的能力，且尝试性支持了点对点穿透。
>
> 架构图
>
> ![1569723904265](/blog_image/text2/1569723904265.png)

其实就是外网去访问一个带有公网ip的反向代理服务器，然后代理服务端被内网的服务所连接绑定，然后配置一些端口，告诉代理服务器，这些端口进来的请求，转发到这台内网服务的某个端口，然后将响应内容在转回给代理服务器，再由代理服务器转给外网

FRP 服务可以分配给你一个域名让你本地的 web 项目提供给外网访问， 特别适合向别人展示你本机的 web demo 以及调试一些远程的 API (比如微信公众号，企业号的开发)，还有计算资源很有限的情况，例如只有一台很低配置的不能开启多个服务的外网服务器，但是内网的计算资源却很充足，也可以frp来充分利用资源。

# 搭建服务

首先到frp的GitHub拿到安装包 <https://github.com/fatedier/frp/releases>

![1569724589166](/blog_image/text2/1569724589166.png)

然后复制你要安装的安装包地址，Linux 在命令行输入

``` shell
wget https://github.com/fatedier/frp/releases/download/v0.29.0/frp_0.29.0_linux_amd64.tar.gz
```

然后解压安装包，进入frp目录

``` shell
 tar -zxvf frp_0.29.0_linux_amd64.tar.gz
```

需要说明的一点是frp分为服务端和客户端分别以frp开头，服务端s结尾，客户端c结尾，frps / frpc，对应的配置文件为frps.ini / frpc.ini

![1569724910226](/blog_image/text2/1569724910226.png)

按照上面的架构图，需要一台服务做反向代理，而且这台服务器必须是公网的，来部署frp服务端，这里我之前想拿到公司内网电脑的外网ip，但是试了很多办法都没用，所以我用了一台阿里云来当服务端，Windows的centos虚拟机来当客户端

# 分辨自己路由器 IP 是真实的公网 IP

tracert 追踪路由也是依赖 ICMP 回显请求和 ICMP 回显应答数据包，因此可以看作 ping 的一个扩展，它用来查看网络所经过的每一跳路由并显示路由的延迟与 ip 地址，可以用来发现出故障的网络路径。

Windows :

1. 点击该 [**链接**](http://www.net.cn/static/customercare/yourIP.asp) 得到自己当前的 IP 地址。
2. 环境：Windows10— 打开一个命令提示窗口
   `tracert 108.216.***.***(输入刚才得到的IP地址)`
3. 如果只有一跳那就说明你的路由器上面是有公网 IP 地址的，如果超过一个跳跃点那就肯定是内网 IP 地址了。

Linux/ Max OS :

1. curl 请求 ifconfig.me 拿到ip 
2. 输入traceroute 来代替 windows的tracert

# 配置服务端

连上外网服务器的ssh 命令行，wget获取安装压缩包，解压进去目录，vi 编辑frps.ini，注意是frps，后面是s不是c，下面的配置是最简单的情况，还有很多特殊的参数，有需要的自行去看官方文档

```shell
# frps.ini
[common]
bind_port = 7000
```

这个端口是后期客户端要连接的端口（如果设置了防火墙，注意开放端口，把端口暴露出来，阿里云要配置安全组），然后保存，输入启动服务命令，注意这里是frps后面是s不c

```she&#39;l&#39;l
./frps -c ./frps.ini # -c 后面指定服务端配置文件
```

启动如果没用报错之类的，可以考虑按ctrl + C停掉服务用后台启动的方式启动服务

``` shell
nohub ./frps -c ./frps.ini & 
```

目录下会生成nohub.out 日志文件

# 配置客户端

现在我们在内网虚拟机配置客户端，和上面一样，解压frp，配置frpc.ini 

```shell
# frpc.ini
[common]
# 服务端服务器的公网ip地址
server_addr = 47.107.x.x
# 端口
server_port = 7000
```

配置好要连接的代理服务器IP就可以启动了，当然这里没进行任何的端口转发配置

``` shell
./frpc -c ./frpc.ini # -c 后面指定客户端端配置文件
```

如果如果出错了，会报错，一般都是报连接失败，可能你的服务端没开启，导致的。

同样可以改成后台启动

```shell
nohub frpc -c frpc.ini &
```



# 外网ssh连接内网服务器

下面我们配一下ssh转发，这样我们就可以使用外网的ip加上固定的端口，ssh连接上我们内网的服务器了

```
# frpc.ini
[common]
server_addr = x.x.x.x
server_port = 7000

#[这个里面的内容可以随意命名]
[secret_ssh]
type = stcp
# 只有 sk 一致的用户才能访问到此服务 可以删除
sk = abcdefg
# 127.0.0.1 或者内网ip
local_ip = 127.0.0.1 
local_port = 22
# 外网ssh连接内网时用的端口 这里我用6000 意思就是 ssh阿里云服务22端口# 就是连接阿里云，6000就是ssh连接内网服务器
remote_port = 6000
```

然后启动客户端 

```
./frpc -c ./frpc.ini # -c 后面指定客户端端配置文件
```

服务端开放6000端口，使用Xshell工具或者Linux/mac 输入命令行 ssh -p 6000 root@47.107.x.x进行ssh连接，连接成功

![1569728370074](/blog_image/text2/1569728370074.png)

使用22端口链接阿里云服务器

![1569728464608](/blog_image/text2/1569728464608.png)

使用6000端口连接公司内网的服务器

# 外网访问rabbitMQ等服务

首先先去docker 拉个rabbitMQ镜像下拉，然后启动/或用其他方式安装个rabbitMQ，启动服务

![1569728761608](/blog_image/text2/1569728761608.png)

然后去浏览器用内网ip访问一下rabbitMQ是否能正常访问

![1569728886520](/blog_image/text2/1569728886520.png)

这时用外网IP访问一下，发现是失败的，现在我们就要配一下15672的端口转发,

将下面的配置复制到客户端的frpc.ini 文件

``` shell
# frpc.ini
[tcp1]
type = tcp
# 本地监听地址，内网地址均可，运行在路由器时有大用处。
local_ip = 127.0.0.1
# 本地监听端口
local_port = 15672
# 入口端口，这个端口会绑定本地的15672
remote_port = 15672
```

然后启动客户端，外网浏览器访问服务端ip加remote_port端口

![1569729593691](/blog_image/text2/1569729593691.png)

java连接mq发现报错了，拒绝连接，需要暴露amqp的5672端口

``` shell
[tcp2]
type = tcp
local_ip = 127.0.0.1
local_port = 5672
remote_port = 5672
```

阿里云记得配置安全组暴露5672和15672端口

不知道有哪些端口被使用可以使用sudo nmap -p 1-65535 localhost查看![1569732446828](/blog_image/text2/1569732446828.png)、



# 客户端具体配置参数

> 客户端可选参数 如下
>
> ```shell
> # [common] 主配置标识
> [common]
> # frps服务端ip/域名
> server_addr = 155.254.32.55
> # frps服务端通讯端口，需要与服务端保持一致
> server_port = 7000
> # 认证密钥 需要与服务端保持一致
> token = 12345678
> 
> ### 以上为必须配置，错误将无法连接服务器 ###
> ### 以下为可选配置 ###
> 
> # 日志记录路径
> log_file = ./frpc.log
> # 日志记录级别: trace, debug, info, warn, error
> log_level = info
> # 日志保留天数
> log_max_days = 3
> 
> 
> # 通过代理链接服务器 ，可选http代理与sock5代理
> # 仅对tcp穿透有效
> # http_proxy = http://user:passwd@192.168.1.128:8080
> # http_proxy = socks5://user:passwd@192.168.1.128:1080
> 
> 
> # 通过http api设置控制frpc动作的管理地址 例如热加载配置文件。
> admin_addr = 127.0.0.1
> admin_port = 7400
> admin_user = admin
> admin_passwd = admin
> 
> # 为客户端启用连接池，指定预创建连接的数量，默认为0及不启用
> pool_count = 5
> 
> # TCP 多路复用，该配置项在服务端和客户端必须一致。默认启用
> tcp_mux = true
> 
> # 自定义穿透名称。可自定义
> user = your_name
> 
> # 决定第一次登录失败时是否退出程序，否则将继续尝试登陆。
> login_fail_exit = true
> 
> # 选择用于连接服务器的通讯协议。
> # 可选tcp/kcp 默认tcp,kcp 使用kcp需要服务器支持。
> protocol = tcp
> 
> # 指定DNS服务器。否则默认
> dns_server = 8.8.8.8
> 
> # 您想要启动的代理名称，以','分割
> # 默认为空，表示所有代理，默认即可，此项我也没搞懂有啥用。
> # start = ssh,dns
> 
> # 心跳包配置，默认即可，无需配置。与服务器保持一致
> # heartbeat_interval = 30
> # heartbeat_timeout = 90
> 
> 
> 
> 
> ###         以上为frpc连接frps的总体配置 接下来配置具体的穿透服务        ###
> 
> 
> 
> # 每个穿透服务的名称，可以自定义。不重复即可
> # 本例是一个http 穿透，用于将搭建与本地http协议的站点穿透至服务器，提供公网访问。
> [http_demo] 
> 
> # 选择穿透协议类型 可选：tcp，udp，http，https，stcp，xtcp
> type = http
> 
> # 本地监听地址，内网地址均可，运行在路由器时有大用处。
> local_ip = 127.0.0.1
> # 本地监听端口
> local_port = 8080
> # 是否启用加密，默认关闭
> use_encryption = true
> # 是否启用压缩，默认关闭
> use_compression = true
> 
> # 通过密码保护你的 web 服务
> # 由于所有客户端共用一个 frps 的 http 服务端口，任何知道你的域名和 url 的人都能访问到你部署在内网的 web 服务，但是在某些场景下需要确保只有限定的用户才能访问。
> # frp 支持通过 HTTP Basic Auth 来保护你的 web 服务，使用户需要通过用户名和密码才能访问到你的服务。
> # 该功能目前仅限于 http 类型的代理，需要在 frpc 的代理配置中添加用户名和密码的设置。
> http_user = admin
> http_pwd = admin
> 
> ## 自定义二级域名
> # 假如你的frps服务端已经配置了subdomain_host参数，并且已经将 *.{subdomain_host}解析到服务器，则可以直接使用subdomain参数，只需要填写子域名，无需填写完整域名
> # 该参数可以自定义，但是不能重复，即同一个服务端同时只能绑定一个唯一的二级域名。
> subdomain = demo_http 
> 
> # 假如服务器端未配置subdomain_host参数，则使用该参数设置绑定域名，需提前将域名解析至服务器。
> custom_domains = demo_http.frp02.wefinger.club
> 
> # frp 支持根据请求的 URL 路径路由转发到不同的后端服务。
> # 通过配置文件中的 locations 字段指定一个或多个 proxy 能够匹配的 URL 前缀(目前仅支持最大前缀匹配，之后会考虑正则匹配)。例如指定 locations = /news，则所有 URL 以 /news 开头的请求都会被转发到这个服务。
> # 仅支持http类型。
> locations = /,/pic
> 
> # 原来 http 请求中的 host 字段 test.yourdomain.com 转发到后端服务时会被替换为 dev.yourdomain.com
> # 可选配置，默认不改变数据。
> host_header_rewrite = example.com
> 
> 
> ## tcp 穿透示例，以穿透ssh服务为例
> [demo_ssh]
> type = tcp
> local_ip = 127.0.0.1
> local_port = 22
> use_encryption = true
> use_compression = true
> # 绑定远程端口。例如本例就是将本地22端口绑定至服务器2222端口，启用穿透后ssh 服务器ip的2222端口即可访问本地主机
> # 如果此处填写0 则服务器会随机分配一个可用端口用于此穿透服务
> remote_port = 2222
> 
> ## udp穿透示例，以转发dns请求为例
> [demo_dns]
> type = udp
> local_ip = 8.8.8.8
> local_port = 53
> remote_port = 6002
> use_encryption = false
> use_compression = false
> ```
>
>

# 小结

其实使用起来就是配置一下服务端和客户端，通过代理转发端口来达到内网渗透

# 引用资料

[frp官方文档](<https://github.com/fatedier/frp/blob/master/README_zh.md>)

[frpc 配置文件全解](<https://www.fingertc.com/archives/171/>)

