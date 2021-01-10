### Docker镜像加载原理与操作

其实就是层级的操作，最开始是只有一层的存在就是最基础的数据，在后面操作的过程中，就是会对层级进行不断地进行添加操作，我们最开始拉取下来的层级就是最基础的镜像，在使用到run命令的候，其实就是在新的层级上对其进行的系列的操作，并不会操作到最底层的数据，然后，就可以使用到**commit**命令来提交自己的镜像。

###  Commit

>  docker commit 提交容器成为一个新的副本
>
> -m="对提交的信息进行描述"
>
> -a= "作者"
>
> 目标镜像的名字：[TAG]

我们下载下来的镜像是最基础的镜像版本，所以说我们若是想要每次都启动tomcat都需要下面的操作

```shell
docker run -d --name tomcat1 -p 8887:8080 tomcat # 表示后台启动，不会占用到当前的前台进程
docker exec -it tomcat1 /bin/bash    # 进入到docker里面进行系列的操作
cp -r ****         # 因为对于docker 安装的tomcat来说 是精简版本的，所以需要拷贝系列的东西，下面我们就可以发布一个自己的版本，然后后面运行自己的版本。
docker commit -m="add webapps" -a="Maycope" f3f1bd92b0ac tomcatF:1.0
# 进行查看
[root@yin ~]# docker images
REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
tomcatf              1.0                 d1ea0cf2c1cd        11 seconds ago      654MB
nginx                latest              bc9a0695f571        12 days ago         133MB
# 进行测试 先停掉当前的tomcat，然后启动自己新提交的docker，

```

## 容器数据卷

### 什么是容器数据卷

之前说的容器，例如我们的Mysql容器，数据都在里面，但是我们很多时候都是会对数据卷进行移除的处理，在这个时候，我们就需要==将容器里面的数据持久化保存到我们容器的外面，使得在容器里面的操作不会影响到我们的基础数据==。

> 容器的操作有一个叫做卷技术，就是目录的挂载的功能，将我们容器内的目录，挂载到我们的虚拟机上面。

### 使用数据卷

> 方法一：直接使用到命令来进行挂载 -v

```shell
docker run -it -v 外部目录：内部目录 容器名称 /bin/bash
docker run -it -v /home/test:/home  centos /bin/bash
# 可以进入到容器的home目录下面创建一个文件，然后在宿主机上进行系列的查看。
docker inspect 容器id     # 使用命令查看容器的基础信息。
```

![image-20201208220613582](https://gitee.com/maycopes/MyImages/raw/master//images/image-20201208220613582.png)

#### Mysql测试

```shell
docker run -d -p 3310:3306 -v /home/mysql/conf:/etc/mysql/conf -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql:5.7
-d 后台运行
-v 数据卷挂载
-e 配置数据库密码
--name 重新命名
```

**远程连接测试：**可以看到的是在本地连接到了远程服务器上的Mysql服务，此处可以对比自己的远程主机上面安装Mysql的复杂过程（注意对于Mysql与Redis来说，远程主机的服务的密码要经常进行更改，不然很有可能被挖矿软件攻击），这样我们在容器删除的时候，也就仍然能够将数据保存下来，而不会丢失。

![image-20201208223605788](https://gitee.com/maycopes/MyImages/raw/master//images/image-20201208223605788.png)

### Docker的匿名挂载与具名挂载
```shell
#  匿名挂载
# -d 表示匿名 -P 表示端口号 -v 表示挂载的路径的信息。
docker run -d -P --name nginx1 -v /etc/nginx  nginx
# docker volume ls 查看我们所有卷的情况。会发现所有的vloume 都是没有命名的，就是所谓的匿名挂载
docker run -d -P --name nginx2 -v ju-nginx:/etc/nginx nginx 
# 此刻就是 具名挂载 通过 -v 名称:容器内的路径。  然后再使用到 docker volume ls 查看。
docker volume inspect ju-nginx   
# 来查看 对于这个容器的具体挂载到主机上的具体的路径信息。
```

![image-20201211102341390](https://gitee.com/maycopes/MyImages/raw/master//images/image-20201211102341390.png)

```shell
此刻进入到对应的目录下面，就可以看到挂载出来的目录信息
# 如何确定我们是具名，匿名 还是 指定路径挂载。
—v 容器内路径   # 匿名挂载
-v 卷名: 容器内路径   # 具名挂载
-v /宿主机路径: 容器内路径     # 
```

**拓展：**

```shell
# 通过 -v 容器内路径： ro,rw 改变我们的读写权限。
ro   readonly   # 只能读
rw   readwrite  # 可读可写。

docker run -d -P --name nginx02 -v ju-nginx:/etc/nginx:ro nginx
docker run -d -P --name nginx03 -v ju-nginx:/etc/nginx:rw nginx
# ro 表示是只读信息，这个时候就只能够再宿主机上对信息进行修改，在容器里面不能够进行系列的修改，是只读的
```

## Dockerfile

###  初始DockerFIle

Dockerfile 就是用来构建文件。

通过这个脚本可以生成镜像，镜像是一层一层的，脚本也是一个一个的命令，每一个命令都是一层。

```shell
# 创建一个dockerfile 文件名字可以随机 
FROM centos
# 指令名字都是大写的。
VOLUME [“volume01","volume02"]

CMD echo "-----end----"
CMD /bin/bash
# 现在开始启动我们的镜像的创建。
# 执行这条语句之前，进入到对应的目录的下面。
docker build -f dockerfile -t maycope/centos .
```

![image-20201211113219071](https://gitee.com/maycopes/MyImages/raw/master//images/image-20201211113219071.png)

经过查看之后，发现我们自定义的镜像是存在的。此时我们就可以使用run命令来进行启动。

```shell
docker run -d -it maycope/centos  /bin/bash #运行
docker exec -it 7bee8331c3f3 /bin/bash     # 进行交互进入到容器里面。
# 退出来之后，使用到 inspect 之后 可以查看到挂载的地址信息。因为我们是么有指定挂载的其他具体信息，所以说是匿名挂载。可以查看到挂载的地址。
docker inspect 容器id。
```

![image-20201211133411350](C:\Users\ASUS\Desktop\upload\image-20201211133411350.png)

> 在后面的使用的过程中，这种使用非常多，因为我们通常会构建属于自己的镜像。

### 构建

我们所谓的docker 核心上就是说可以用来构建docker镜像的核心的文件。我们可以对其进行内容的编写，然后 进行系列的生成，就可以完成我们制作精选的目的。

1. 编写一个dockerFile 文件。
2. docker  build 构建成为一个镜像。
3. docker run 来运行一个镜像。
4. docker push   发布镜像（DockersHub，阿里云镜像仓库）

>我们也可以去对应的centos（docker) [docker 官网](https://hub.docker.com/)，然后点击对应的镜像信息，可以看到最基础的centos，其实是最基础的什么都没有，我们想要自己编写一个centos，带上我们想要的jdk,tomcat,mysql,redis,时候，就可以自己发布一个对应的centos。

#### 指令

**基础知识**

1. 每一个保留的关键字（指令）都必须是完全的大写字母。
2. 执行从上到下顺序执行。
3. “#” 表示的是注释信息。
4. 每一个指令都会提交一个新的镜像层，并执行提交。
5. 我们的**dockerfile** 是面向开发，在后面我们开发项目，都是直接编写**dockerFile**，然后添加上我们进行的更改，就可以将这个dockerFile修改之后，我们就可以这个dockerFile 进行交付就好了。‘

>DockerFile： 构建文件，定义了一切的步骤，源代码。
>
>DockerImages： 通过 DockerFile  构建成为的镜像，最终发布和运行的产品。
>
>Docker容器： 容器就是镜像运行起来，然后提供服务的服务器系统。



```shell
FROM # 基础的镜像，一切都是从基础的镜像开始构建的。
maintainer  # 告诉别人，维护这个镜像的人是谁。
run   # 镜像构建的时候 会执行的命令。
add   # 步骤： tomcat 镜像  就是添加自己想要的内容
workid # 镜像的工作的目录。
volume # 挂载的目录。
expose # 暴露的端口。
cmd    # 指定这个容器启动的时候要运行的命令，只有最有一个会生效，可以被替代。
entrypoint # 指定这个容器要运行的命令，可以追加命令。
copy     # 类似于add 将我们文件拷贝到镜像中。
env      # 构建的时候设置环境变量。
```





![image-20201211171157555](https://gitee.com/maycopes/MyImages/raw/master//images/image-20201211171157555.png)

####  开始操作

> 创建一个属于自己的centos

```shell
# 1. 编写自己的dockerfile 配置文件。
FROM centos
MAINTAINER maycope<19129183@qq.com>

ENV MYPATH /usr/local
WORKDIR $MYPATH

RUN  yum -y install vim
RUM yum -y install net-tools

EXPOSE 80

CMD echo $MYPATH
CMD echo "-----end----"
CMD /bin/bash

# 2.通过这个文件来构建一个镜像 -f 表示构建的文件，-t 版本号。
docker build -f mydoc-centos -t mycentos:0.1 .
 # 最后出现，表示构建成功。
Successfully built a70bf824d356
Successfully tagged mycentos:0.1
# 3. 测试运行。可以看到的是当前的目录就是我们设置的跟目录，并且，我们安装的一些其他的基础的信息也都存在。而我们之前的centos，默认的根目录，并且其他我们所谓的命令也都是不存在的。
[root@yin dockerfile]# docker run -it mycentos:0.1
[root@101f2a753ed8 local]# pwd
/usr/local
# 4. 使用到docker history 来查看历史是如何做出来的。平时也可以使用到history 来查看别人的镜像是如何做的。
```

![image-20201211203007305](https://gitee.com/maycopes/MyImages/raw/master//images/image-20201211203007305.png)



#### 比较重要的命令

##### CMD与ENTRYPOINT

```SHELL
[root@yin dockerfile]# vim mydoc-cmd

FROM centos 
CMD ["ls","-a"]

[root@yin dockerfile]# docker build -f mydoc-cmd -t cmdtest .
[root@yin dockerfile]# docker run cmdtest
.
..
.dockerenv
bin
dev
etc

# 以上就是自己构建容器的三部过程。通过执行我的cmd命令，来获取到我们执行的命令。

# 思考 上面我们简单的镜像实现的功能是ls -a 我们想要进行命令的追加功能
# 使用到 docker run cmdtest -l 就是实现命令的追加发现并不成功，这个时候就引出来了 entrypoint。


# 我们重复上面的操作过程，只不过将命令行里面的
FROM centos
ENTRYPOINT ["ls","-a"]
docker build -f mydoc-entry -t entrytest.
# 没有使用 -l
[root@yin dockerfile]# docker run entrytest
.
..
.dockerenv
bin
dev
etc

# 追加 -l 的时候。

[root@yin dockerfile]# docker run entrytest -l
total 56
drwxr-xr-x   1 root root 4096 Dec 11 13:01 .
drwxr-xr-x   1 root root 4096 Dec 11 13:01 ..
-rwxr-xr-x   1 root root    0 Dec 11 13:01 .dockerenv
```

###  实战通过 DockerFile 制作Tomcat 镜像

> 这一节也就主要是利用我们现有的基础的信息来构建一个自己的**tomcat**镜像，利用基础的命令，挂载。

1. 准备我们的镜像文件，需要tomcat压缩包，jdk 压缩包。

   把东西都传递到**tomcat**里面，然后vim 

2.  编写==Dockerfile== 文件，这个文件（名字是通用的）

关于这个如何制作的问题，前提准备工作：新建立一个**tomcat**的文件，里面存放上我们必须要的**tar.gz**包，就是两个**jdk**和**tomcat**，然后在这个tomcat下面使用到*vim Dockerfile*来建立==dockerfile==文件。

```shell   
FROM centos
MAINTAINER    maycope<99876@qq.com>

COPY readme.txt /usr/local/readme.txt

ADD  资源（）   /usr/local
ADD  资源（）   /usr/local

RUN yum -y install vim
ENV MYPATH  /usr/local
WORKDIR  $MYPATH

ENV JAVA_HOME /usr/local/jdk.1.8.0_11
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/toos.jar

ENV CATALINA_HOME /usr/local/apache
ENV CATALINA_BATH /usr/local/apache

ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin

EXPOSE 8080

# 日志输出。
CMD /usr/local/apache/bin/startup.sh && tail -F /usr/local/apache/bin/logs/catalina.out


# 构建   -f 表示配置文件，这里，我们配置文件的名称就是Dockerfile，所以不需要指定-f。
docker build -t divtomcat 
# 启动
docker run -d -p 8090:8080 --name maycopetomcat -v /home/maycope/build/tomcat/test:/url/apache/webapps/test -v /home/maycope/build/tomcat/logs:/url/apache/logs divtomcat
```



## 数据卷容器

> 理解起来就是利用这个容器去给其他的容器共享数据。

![image-20201211140354298](C:\Users\ASUS\Desktop\upload\image-20201211140354298.png)

```shell
# 启动三个容器，通过我们自己写的镜像启动。
docker run -d -it --name centos01 /bin/bash maycope/centos
# 进行挂载
docker run -it --name centos02 --volumes-from docker01 maycope/centos
# 挂载完成值后，我们想要进入到容器里面进行查看
docker exec -it 容器id /bin/bash
docker attach 容器id
# 进入到数据卷里面 在卷一里面修改一些配置，对应查看在另外一个数据卷里面能否进行展示。
```

![image-20201211150642670](upload\image-20201211150642670.png)

同步的数据信息，是我们两个==volume==下面的文件信息。因为我们在创建镜像的时候，就是这两个文件目录下面的具体信息。

#### 思考

若是说我们的==docker01==删除之后，我们的**docker02**和**docker03**里面的内容并不会删除，还是会存在。

###  实现多个Mysql数据的共享

```shell
docker run -d -p 3310:3306 -v /etc/mysql/conf.d /var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql:5.7

docker run -d -p 3310:3306 -e MYSQL_ROOT_PASSWORD=123456 --name mysql02 --volumes-from mysql01 mysql:5.7 

# 在这个时候，我们两个数据库之间的数据就能够进行互相的通信处理。
```

> 结论：
>
> 容器之间信息的互相传递，就像是之前我们多个服务器之间的配置要兼顾的时候，可以把所有服务器的配置都放置在一起。并且对于数据卷来说，数据卷容器的生命周期，会一直持续倒没有容器使用为止。但是当我们在使用到-v保存到本地时候，就永远不会删除。

## 发布自己的镜像

1. 首先是需要登录到dockerbub上面，注册一个属于自己的账号。

```shell
docker login -u maycope
password:password。
docker tag 916af5c4122a entrytest:1.0
docker push 镜像
# 提交的时候也是一层一层进行提交。
```

## Docker 基础总结

![image-20201214171651931](https://gitee.com/maycopes/MyImages/raw/master//images/image-20201214171651931.png)

## Docker 网络

首先是来查看网络的大体都有哪些：

![image-20201214174041882](https://gitee.com/maycopes/MyImages/raw/master//images/image-20201214174041882.png)

```shell
现在我有三个网络，但是docker又是如何处理容器之间的网络请求于访问的。
# 有一个容器是tomcat，有一个是mysql容器，但是对于两个容器之间的网络是如何进行请求与转发的？

# 表示后台启动，下载最新的tomcat。
docker run -d -P --name tomcat01 tomcat

# 表示交互获取到当前
docker exec -it tomcat01 ip addr
```

> 原理

1. 我们每次启动一个容器的时候，docker 就会给docker容器分配一个ip地址，我们只要安装了docker，就会有一个网卡docker0，使用到的是 **evth-pair** 技术，此时我们可以通过在docker宿主机来ping容器的ip地址，发现是可以进行互通的。

![image-20201215095755828](upload\image-20201215095755828.png)

```shell
我们可以发现的是，对于这个容器所带来的网卡都是一对一对存在的。
evth-pair 就是一对虚拟设备接口，他们都是成对出现的，一端是连着协议，一端是彼此相连接，就是因为有这个特性，evth-pair 充当着一个桥梁，连接着各种虚拟网络设备。
# 如下图所示，本来我们的宿主机到容器里面是不能够进行互通的，但是通过evth-pair，两个里面都会有一个对接的地方，在两边对接完成之后，就可以进行互相的通信了。
```

![](upload\image-20201215100540497.png)

**多个容器进行测试时候**：

```shell
可以发现的是对于ip都是成对存在的，我们的tomcat01，要和02进行通信的时候，并不是直接进行通信的，而是先通过我们的docker0 网络路由，来进行网络请求的转发，对于路由器里面的可以通过直接注册将所有的路由信息都存放在里面，或者通过广播的方式来进行路由的请求转发接收。
```



![image-20201215101605862](https://gitee.com/maycopes/MyImages/raw/master//images/image-20201215101605862.png)

所有的容器不指定网络的情况下，都是docker0路由的，docker会给我们的容器分配一个默认的可用IP。

> 思考一个场景，我们在编写了一个微服务的时候，我们指定了对应的数据库地址，但是在项目进行重启时候，数据库的地址在变更之后，我们就需要重新变更数据库的连接地址，所以想要解决这个问题，通过服务名称来访问对应的容器。

### 容器互联

####  --link

我们想要实现上面的需求，就需要使用到 ==--link==,来实现

```shell
docker run -d -P --name tomcat003 --link tomcat002 tomcat
docker exec -it tomcat003 ping tomcat002
# 就可以实现具体的网络的联通。不需要通过具体的ip，但是弊端在于容器创建时候，不是双向的，是单向的，就是启动哪个容器和对应的容器进行连接时候，是一对一的
```

#### 自定义网络

>查看所有的docker网络

![image-20201216102446804](upload\image-20201216102446804.png)

**基本的网络的模式**

bridge :  桥接模式（默认）

none   ： 不配置网络

host     ： 主机模式 和宿主机一起来共享网络

**测试**

```shell
# 第一条 我们不带网络执行的命令，执行的就是docker0 
docker run -d -P --name tomcat001 tomcat 
= docker run -d -P --name tomcat001 --net bridge tomcat 

# bridge0 默认 域名不能访问 --link 能够通 但是不方便


# 自定义网络 
# --driver bridge
# --subnet 192.168.0.0/16
# --gateway 192.168.0.1

docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet
8bbe2c13b2180ad08fd8281a51dcb5f0e5cdb5adeccb074e03ea340f85ef51bb
[root@yin ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
2e8a058516bd        bridge              bridge              local
8af4b107b451        host                host                local
8bbe2c13b218        mynet               bridge              local
96610e843152        net1                bridge              local
# 使用 inspect 查看。
[root@yin ~]# docker network inspect mynet
[
    {
        "Name": "mynet",
        "Id": "8bbe2c13b2180ad08fd8281a51dcb5f0e5cdb5adeccb074e03ea340f85ef51bb",
        "Created": "2020-12-16T10:55:43.466196107+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.0.0/16",
                    "Gateway": "192.168.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,

```

**将自己的服务放在自己的网络上**

```shell
# 首先是部署
docker run -d -P --name tomcat01 --net mynet tomcat
docker run -d -P --name tomcat02 --net mynet tomcat 
docker network inspect mynet

```

![image-20201216110606897](upload\image-20201216110606897.png)

```shell
# 好处： 还记得之前我们提出的问题是，对于容器与容器之间绑定ip地址时候，出现ip地址变化问题，想要通过 --link来建立连接，但是通过 --link进行连接时候，是单向的，而且不方便操作。所以我们通过上面的 create 来创建自己的网络，然后将容器发布到自己的网络里面，下面就是可见的好处
# 现在不但可以通过ip地址进行连接，还能够通过容器的名称进行连接的建立，这里就可以看到我们自定义网络的好处，可以自定义很多的网络，然后将那些需要建立连接的容器再启动的时候，启动在我们的自定义网络里面。
# 这个时候 我们若是想要在运行redis和mysql两个集群的时候，并且不想要两个网络混杂在一起的时候，就会需要建立两个自己的网络
```

![image-20201216111530845](upload\image-20201216111530845.png)

####  网络连通

```shell
[root@yin ~]# docker run -d -P --name tomcat-bri-01 tomcat
5549f684f066186804cadb86ebf1975ed1396ea860da64ef223a3114654b325d
[root@yin ~]# docker run -d -P --name tomcat-bri-02 tomcat
2a9459a040dfa3c89ced5d291b94c2f47a54d2f5117f5172eadee2ea88bb4fc1
[root@yin ~]# docker exec -it ping tomcat01
Error: No such container: ping
[root@yin ~]# docker exec -it tomcat-bri-01  ping tomcat01
ping: tomcat01: Name or service not known
# 网络连通，利用到 docker network connnet 将一个容器联通到一个网络，我们这里就是将上面创建的tomcat-bri-01 联通到我们的 192.168.0.1 网络，就是所谓的网络连通。													
```

![image-20201216134315116](upload\image-20201216134315116.png)

```shell
# 具体使用（见下图），我们的网络192.168.0.1网络名称是mynet，连通到我们新创建的容器（其网络就是我们基本的网络），然后连通之后，使用的ping信息时候，就可以正常的展示。
[root@yin ~]# docker network connect mynet tomcat-bri-01
[root@yin ~]# docker exec -it tomcat-bri-01 ping tomcat01
PING tomcat01 (192.168.0.2) 56(84) bytes of data.
64 bytes from tomcat01.mynet (192.168.0.2): icmp_seq=1 ttl=64 time=0.060 ms
64 bytes from tomcat01.mynet (192.168.0.2): icmp_seq=2 ttl=64 time=0.062 ms
```

![image-20201216135350766](upload\image-20201216135350766.png)

**具体的流程图**

![image-20201216140627669](upload\image-20201216140627669.png)

###  部署redis集群

1. 对于redis集群来说，是偶数存在的。是三个从，三个主，所以最低需要启动六个redis单点容器。

2. 既然我们是自己的redis集群，就需要有自己的网络。建立自己的网络信息。

   ```shell
   docker network create --driver bridge --subnet 192.169.0.0/16 --gateway 192.169.0.1 redisnet
   
   # 具体的配置信息。这里就不具体配置出来。
   
   ```

##  SpringBoot微服务打包Docker镜像

1、构建一个SpringBoot项目

就是我们最基础的SpringBoot项目，定义一个接口，使用到对应的注解。

2、打包应用

启动能够正常进行访问之后，打包起来。

3、 编写对应的dockerfile

![image-20201216154725754](upload\image-20201216154725754.png)

4、构建镜像

将对应的jar包和对应的`Dockerfile`文件拷贝到服务器上对应的一个文件夹下面。

```SHELL
# 按理来说应该还会有对应的 文件信息，但是文件名称是标准的文件名称，所以可以不用指定对应的文件名。
docker build -t maycopeweb .
```

5、发布运行

```shell
# 不指定端口，后台启动服务。
docker run -d -P --name maycopes maycopeweb 
# 查看启动的容器,为了能够获取到对应的映射出来的端口。
docker ps
#  curl localhost:端口号/hello
conds       0.0.0.0:32779->8080/tcp   maycopeweb
[root@yin idea]# curl localhost:32779/hello
Hello,Maycope[root@yin idea]# 
# 可以查看到自己发布的接口信息。
```

# Docker 高级部分

## Docker Compose 

[网址](https://docs.docker.com/compose/)

> 之前我们讲到我们发布的一个微服务的镜像，就是先创建一个SpringBoot项目，然后，打包起来成为一个jar包，编写Dockerfile文件，上传上去之后先进行一个build，创建完成之后，进行系列的run，才算是一个完整的微服务，但是有很多容器时候，我们不可能启动100个对应的微服务。所以需要进行系列的判断。
>
> 所以对于Docker Compose来说 可以来轻松高效的管理容器，定义运行多个容器。

**使用：**

1. 还是需要我们定义对应的Dockerfile来保证我们的环境再任何地方运行。
2. 定义对应的**servies**通过`docker-compose-yml`,然后运行环境。
3. 运行`docker-compose up`命令就可以运行我们的对应App。

**作用：**批量对容器进行操作。

> 自己的理解：
>
> 1. 只是一个Docker的开源的项目，需要安装。
> 2. Dockerfile 让程序再任何地方运行。web服务，可能还会有redis，mysql，nginx等等。下面就是配置文件如何控制处理。
> 3. 重要概念
>    1. 服务 service就是我们的应用。
>    2. 最终打包起来就是 项目 project。就是我们很多容器的集合体。

**步骤**

[快速开始](https://docs.docker.com/compose/gettingstarted/)

1、应用**app.py**。

2、Dockerfile 应用打包为镜像

3、Docker-compose yaml 文件（定义整个服务，需要的环境，web，redis）。

4、启动compose。

5、启动起来之后我们可以发现是启动起来一个叫做web_1,和一个redis_2。

关于这个文件名字的问题默认的服务名字： 文件名 _ 服务名 _ num，在多个服务器时候，也就是集群的时候，若是我们的集群服务有两个服务器，现在有一个应用，既要在A服务器上运行，也要在B服务器上运行，对应的num就是副本的数量。现在我们有一个集群里面有四个服务器，需要部署四台redis，此时对应的就是 四个副本。在集群的状态下面，也不可能只会有一个运行的实例，就会有很多的实例，可以实现高可用。

![image-20201221112319681](https://gitee.com/maycopes/MyImages/raw/master//images/image-20201221112319681.png)

6、 网络规则，记得之前网络相关的具体信息，有关于网络访问的问题，现在情况就是若是我们将所有服务都部署在同一个网络下面，这样就可以不通过ip地址来访问到对应的服务，可以通过对应的服务的名称来对应访问。

![image-20201221113737315](https://gitee.com/maycopes/MyImages/raw/master//images/image-20201221113737315.png)

7、 停止docker 服务 docker-compose down ctrl+c。

**总结：**所谓的docker-compose 就是 能够通过配置文件的编写，通过`compose`

来一键启动所有的服务。

1. 首先是我们通过了解Docker 镜像，run=> 容器。
2. Dockerfile 构建镜像（服务打包）
3. docker-compose up 启动项目（编排，多个微服务/环境）
4. docker 网络，通过 inspect 进行查看网络之间的关系。

## yaml配置