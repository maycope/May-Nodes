## Nginx 反向代理，负载均衡及搭建高可用集群

## 前提准备

首先是对于`linux`环境下的安装(本地机器是Windows版本，大家可以使用`Vmware`,但是需要配置网络连接等，这里就不再展示虚拟机上的演示。这里使用到个人的**阿里云云服务器**搭配上**xftp**与**xshell**来进行文件的上传与连接命令行的输入)


<div align = "center"><img src= "http://maycope.cn/images/image-20200807092205021.png"></div>

**注意：以下命令皆为CentOS7所使用。**

下面开始进行系列依赖的安装：

#### gcc 安装：

```shell
yum -y install gcc automake autoconf libtool make
yum install gcc gcc-c++
```

#### pcre 安装

```shell
cd /usr/local/src
wget    https://netix.dl.sourceforge.net/project/pcre/pcre/8.40/pcre-8.40.tar.gz
tar -zxvf pcre-8.40.tar.gz
cd pcre-8.40
./configure
make && make install
```

#### zlib 安装

```shell
cd /usr/local/src
wget http://zlib.net/zlib-1.2.11.tar.gz      wget  http://www.zlib.net/zlib-1.2.11.tar.gz
tar -zxvf zlib-1.2.11.tar.gz
cd zlib-1.2.11
./configure
make && make install
yum install -y zlib zlib-devel
```

#### openssl 安装

```shell
cd /user/local/scr
wget https://www.openssl.org/source/openssl-1.0.1t.tar.gz
tar -zxvf openssl-1.0.1t.tar.gz
```

#### nginx 安装

```shell
cd /user/local/scr
wget http://nginx.org/download/nginx-1.1.10.tar.gz
tar zxvf nginx-1.1.10.tar.gz
cd nginx-1.1.10
./configure
make && make install
启动nginx
/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
```

完成之后可以先行查看自己的自己的服务器开放了哪些的端口：

```shell
firewall-cmd --list-all
```
<div align = "center"><img src= "http://maycope.cn/images/image-20200807093326764.png"></div>

若是没有进行端口的开发可以使用如下命令进行端口的开放：

```shell
firewall-cmd --zone=public --add-port=80/tcp --permanent
# 其中80 可以进行修改为自己想要开放的端口，当然前提下是你要打开了防火墙。
```

#### 防火墙设置

```shell
systemctl status firewalld.service # 查看防火墙状态
systemctl stop firewalld.service   # 关闭防火墙
systemctl start firewalld.service  # 打开防火墙
```

### nginx基础命令

1. 在完成以上的基础准备之后，已经对nginx进行了启动，查看当前的nginx情况：

```shell
ps -ef | grep nginx
```

2. 对nginx进行启动，停止与重启。

```shell
cd /usr/local/sbin   # 注意要进入到安装的nginx对应的相关的目录下才能够执行相关联的语句。
./nginx # 表示启动nginx
./nginx -s stop    # 表示对nginx 进行停止。
./nginx - s reload # 表示重启，一般在配置文件进行修改之后使用。
```

在启动完成之后就可以进行ip地址的访问（因为对于nginx来说默认是启动在80 端口，所以可直接进行ip地址的访问）

<div align = "center"><img src= "http://maycope.cn/images/image-20200807094702610.png"></div>


## 配置文件讲解：

首先是配置文件地址：`/usr/local/nginx/conf/nginx.conf`。

对于nginx配置文件来说分为三大块：



### 全局快

从配置文件开始到 events 块之间的内容，主要会设置一些影响 nginx 服务器整体运行的配置指令，主要包括配置运行 Nginx 服务器的用户（组）、允许生成的 worker process 数，进程 PID 存放路径、日志存放路径和类型以及配置文件的引入等。

比如下面第一行配置的：

```shell
worker_processes  1;
```

这是 Nginx 服务器并发处理服务的关键配置，worker_processes 值越大，可以支持的并发处理量也越多，但是会受到硬件、软件等设备的制约

### events 块

```shell
events {
    worker_connections  1024;
}
```

events 块涉及的指令主要影响 Nginx 服务器与用户的网络连接，常用的设置包括是否开启对多 work process 下的网络连接进行序列化，是否允许同时接收多个网络连接，选取哪种事件驱动模型来处理连接请求，每个 word process 可以同时支持的最大连接数等。

上述例子就表示每个 work process 支持的最大连接数为 1024. 这部分的配置对 Nginx 的性能影响较大，在实际中应该灵活配置。

### http 块

#### 

```shell
http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
       server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }
…………………………
```

这算是 Nginx 服务器配置中最频繁的部分，代理、缓存和日志定义等绝大多数功能和第三方模块的配置都在这里。

 需要注意的是：http 块也可以包括**http 全局块**，**server块**

#### http 全局块

http 全局块配置的指令包括文件引入、MIME-TYPE 定义、日志自定义、连接超时时间、单链接请求数上限等。

#### server 块

这块和虚拟主机有密切关系，虚拟主机从用户角度看，和一台独立的硬件主机是完全一样的，该技术的产生是为了节省互联网服务器硬件成本。

每个 http 块可以包括多个 server 块，而每个 server 块就相当于一个虚拟主机。

而每个 server 块也分为全局 server 块，以及可以同时包含多个 locaton 块。

1， **全局server块**

最常见的配置是本虚拟机主机的监听配置和本虚拟主机的名称或 IP 配置。 

2，**location块**

一个 server 块可以配置多个 location 块。

这块的主要作用是基于 Nginx 服务器接收到的请求字符串（例如 server_name/uri-string），对虚拟主机名称（也可以是 IP 别名）之外的字符串（例如 前面的 /uri-string）进行匹配，对特定的请求进行处理。地址定向、数据缓存和应答控制等功能，还有许多第三方模块的配置也在这里进行。

## 反向代理

### 何为反向代理：

在了解反向代理之前，先了解正向代理：

**正向代理**：如果把局域网外的 Internet 想象成一个巨大的资源库，则局域网中的客户端要访问 Internet，则需要通过代理服务器来访问，这种代理服务就称为正向代理。简而言之就是起到一个帮助到达目标网络的作用。

**反向代理：**反向代理，其实客户端对代理是无感知的，因为客户端不需要任何配置就可以访问，我们只需要将**请求发送到反向代理服务器**，由反向代理服务器去选择目标服务器获取数据后，在返回给客户端，此时反向代理服务器和目标服务器对外就是一个服务器，暴露的是代理服务器地址，隐藏了真实服务器 IP 地址。

**举个栗子**：我们在本地环境上安装一个tomcat 服务器，对于tomact来说默认访问是8080端口，但是我们不想输入80端口就可以直接进行访问，就需要我们直接访问80 端口 表示访问到nginx服务器，然后进行配置文件的配置，将请求转发到我们的tomcat服务器上。

我们可以进行如下配置：

在http模块中 进行如下配置：

```shell
 server {
        listen       80;
     #  server_name  localhost;
       server_name 121.*.*.34;# 表示监听的服务的名称是 如下地址
        location / {
            root   html;
           proxy_pass http://127.0.0.1:8080; # 表示进行转发的地址，首先需要在本机安装tomact 并启动 自行官网下载即可
            index  index.html index.htm;
        }
        }
```

完成配置之后记得重启服务：即可看到如下展示：表示从80端口跳转到8080端口

<div align = "center"><img src= "http://maycope.cn/images/image-20200807100806544.png"></div>


#### 实例二

在上面的案例中我们访问的还是默认的80端口，若是我们想要更换端口去访问呢，我们若是想要访问其他的端口，或者说是访问带路径的信息呢，该如何进行操作，这个时候就可以添加一个server，因为一个server监听一个唯一的端口，这里我们可以再创建一个server 选中我们监听的端口信息。再location中进行配置即可。

```shell
 server {
        listen      3308; # 我们监听不同的端口信息
        server_name  localhost;
        location  ~ /edu/  {
        # 在tomcat 下的webpage下创建对应的目录，下添加一个index.html 页面。
            proxy_pass http://127.0.0.1:8080;
        }
    }
```

<div align = "center"><img src= "http://maycope.cn/images/image-20200807102916770.png"></div>
这个时候我们就可以访问如下网址：可以发现如下的3308（自己测试时候要提前开启端口）端口搭配上edu路径和对应的**index.html** 就可以访问到我们`tomact `服务器下面的对应的页面信息。


<div align = "center"><img src= "http://maycope.cn/images/image-20200807103007135.png"></div>

## 负载均衡

### 概念

负载均衡即是将负载分摊到不同的服务单元，既保证服务的可用性，又保证响应足够快，给用户很好的体验。快速增长的访问量和数据流量催生了各式各样的负载均衡产品，很多专业的负载均衡硬件提供了很好的功能，但却价格不菲，这使得负载均衡软件大受欢迎， nginx 就是其中的一个，在 linux 下有 Nginx、LVS、Haproxy 等等服务可以提供负载均衡服务，而且 Nginx 提供了几种分配方式(策略)：

1. 轮询（默认）

   每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器 down 掉，能自动剔除。

2. weight

   weight 代表权,重默认为 1,权重越高被分配的客户端越多

   指定轮询几率，weight 和访问比率成正比，用于后端服务器性能不均的情况。 例如：

   ```shell
   upstream server_pool
   {
   server 121.111.2.34 weight = 10;
   server 121.111.2.35 weight = 10;
   }
   ```

3. ip_hash

   每个请求按访问 ip 的 hash 结果分配，这样每个访客固定访问一个后端服务器，可以解决 session 的问题。

   ```shell
   upstream server_pool{
   ip_hash;
   server 121.111.2.34 weight = 10;
   server 121.111.2.35 weight = 10;
   }
   ```

   

4. fair(第三方)

按后端服务器的响应时间来分配请求，响应时间短的优先分配。

```shell
upstream server_pool{
fair;
server 121.111.2.34 weight = 10;
server 121.111.2.35 weight = 10;
}
```

### 实例

在http中添加`upstream`关键字：

```shell
http{
……
upstream myserver{
# 拥有两台服务器
fair # 表示 按照相应的时间进行分配哪个服务来相应当前的请求 。
server 121.111.2.34:8080;
server 121.111.2.35:8080;
# 注意 这里是模拟两台服务器，对于没有两台真实的服务器的，可以开两个tomcat 服务 配置不同的端口
server 121.111.2.34:8080;
server 121.111.2.34:8081;
}

server{
		listen 80;
		server_name 121.111.2.34;
		……
		location /{
		proxy_pass http://myserver
		# myserver 和上面自己所起的upstream 相对应
		…… 
		}
}
```

这样就可以在访问到本机的80端口时候进行数据的转发处理，选择一个特定的主机进行服务的相应接收

##  高可用

### 简介

关于这部分知识，之前在学习[基于Mysql与Redis集群上项目负载均衡](https://blog.csdn.net/weixin_44015043/article/details/105869962)时候有使用的到`docker` 来搭建后端的`nginx`负载均衡,同时，这部分的具体完成的信息也在我的个人`github`[github项目地址](https://github.com/maycope/Docker-front-and-back-project-deployment-)账户上有所体现。

无论是nginx集群的创建，还是负载均衡的完成上，对于`docker `来说都为我们提供了很大的遍历，当时的docker配置上夹杂了一些其他的配置，可能把这部分分割出来还是需要点时间去理解，这里就单独进行分析与学习。

### 实例

关于具体的逻辑信息，在我如上的**基于Mysql与Redis集群上项目负载均衡**，主要就是说：为了防止只有一个`nginx`导致问题的出现的问题，然后添加两个`nginx`进行轮询，但是这个轮询的工作不需要我们自行完成，而是借助第三方的工具`Keepalived`，帮助我们完成系列的工作,见下图：利用心跳检测，来具体检测出故障的机器，并且进行ip的虚拟化，只需要记录一个ip信息，然后后面的操作



<img src="https://img-blog.csdnimg.cn/20200501081514734.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDAxNTA0Mw==,size_16,color_FFFFFF,t_70">

1. 首先是有两台服务器，然后在其上都进行`keepalived`的安装

   ```shell
   yum install keepalived -y # 安装
   rpm -q -a keepalived # 查看安装的情况
   cd /etc/keepalived
   vi keepalived.conf
   ```

2. 然后修改两台keepalived中的文件的配置信息：

```shell
global_defs {

    notification_email {

        acassen@firewall.loc

        failover@firewall.loc

        sysadmin@firewall.loc

    }

    notification_email_from Alexandre.Cassen@firewall.loc

    smtp_server 121.111.2.34

    smtp_connect_timeout 30

    router_id LVS_DEVEL #唯一

}

vrrp_script chk_http_port {
    script "/usr/local/src/nginx_check.sh"
    interval 2	#（检测脚本执行的间隔）2 s执行一次
    weight 2   # 脚本成立 权重增加2。
}

vrrp_instance VI_1 {

    state BACKUP	# 备份服务器上将 MASTER（主） 改为 BACKUP 

    interface ens33	# 网卡

virtual_router_id 51	# 主、备机的 virtual_router_id 必须相同

    priority 100	# 主、备机取不同的优先级，主机值较大，备份机值较小

    advert_int 1 # 进行心跳检测 每隔一秒 发送检测信息 查看是否存活

    authentication {

        auth_type PASS
        auth_pass 1111
    }

    virtual_ipaddress {

        121.111.2.20 # 虚拟地址
    }
}
```

3. 脚本文件的编写,不同于使用docker 的方式，如下是使用到`dcoker `的解决方案,配置文件的不同。

   <div align = "center"><img src= "http://maycope.cn/images/image-20200807135628511.png"></div>

如下我们编写具体的信息查看nginx是否存活。存放的目录是`/usr/local/src/`。

```shell
#!/bin/bash

A = `ps -C nginx –no-header |wc -l`

if [$A - eq 0]; then

    / usr / local / nginx / sbin / nginx
    # 查看nginx 是否还存活

sleep 2

if [`ps -C nginx --no-header |wc -l` - eq 0]; then killall keepalived

fi

fi
```

4. 完成之后对`nginx`和`keepalived` 进行启动即可`keepalived` 启动命令：`systemctl start keepalived`

这个时候我们访问虚拟地址：**121.111.2.20**，然后两个`keepalived`会抢占这个虚拟ip，同时将服务请求发送到不同对应的`nginx`服务器上,这样就保证了在一个服务器出现故障时候，另外一个还能够正常的工作。
