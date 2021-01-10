## 前言

前段时间学习到高并发服务器集群的搭建与实战，有了解与使用过Docker，时间再往前推是当时在自己的云服务器上安装Redis做笔记里面说使用到Docker会更快一些，后面也就真的使用到Docker来安装**Redis**，并做了集群，但是现在也都忘记的差不多了，好在有记录下来。什么时候要是再度使用到的时候拿出来看一看笔记应该是没问题的。

## 基础

### 什么是Docker

这里可以和我们的虚拟机做比较，之前也学习过系列的虚拟机，虚拟机的运行的机制就是模拟出来一台电脑，虽然说功能会好很多，就是我们基础的电脑的功能，但是相对之下也就是会占用很多我们计算机的基础的性能。因为我们本机的计算机的资源是有限的。

但是对于Docker来说，模拟的不是一个完整的操作系统，是一个又一个的容器，在同一个内核上面，每一个容器都有自己的应用和自己的库，这种情况下，就可以做到互相不干扰的状态。

可以借用《深入浅出Docker》里面的一句话，Hypervisor是对硬件的虚拟化处理,而我们的容器是操作系统虚拟化操作，容器将系统资源划分为虚拟资源。

### Docker里面的名词

注意：这部分还是建议去看具体的书籍《深入浅出Docker》（奈吉尔·波尔顿）写的这本，里面的介绍还是要比其他相关的介绍要好很多。

#### 镜像

关于镜像理解起来就是一个模板，我们可以通过这个模板来创建容器服务，例如我们的tomcat镜像，就是利用这个模板来创建的一个tomcat服务，我们可以通过这个模板来创建多个容器也就是多个服务。其实也就是可以说我们的镜像就是停止运行的容器。

#### 容器

对于容器来说，就是我们上面提到的可以来运行我们的各种服务的载体，也就是一个比较小巧的Linux操作系统，我们可以对这个容器执行，启动，停止，删除等基本的命令。

#### 仓库（repository）

就是我们所有的东西的存放的地址。

###  安装

首先是卸载原来的docker

当然在安装之前，我们还是需要配置好对应的环境，例如有一台对应的linux服务器（这里是我一直再用的阿里云centos7）和于其建立连接的连接工具xshell，

```shell
 # 卸载原来的版本
 yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
# 安装对应的依赖
yum install -y yum-utils
# 下载对应的仓库信息
# 1国外的会比较慢：
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
# 2. 推荐国内的阿里的镜像加速：
sudo yum-config-manager \
    --add-repo \
 http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# 安装我们docker所需要的基础组件
yum install docker-ce docker-ce-cli containerd.io
# 启动docker
systemctl start docker

```
![image-20201205195714302](https://gitee.com/maycopes/MyImages/raw/master//images/image-20201205195714302.png)

```shell
# docker 进行测试
docker run hello-world

```

![image-20201205200152294](https://gitee.com/maycopes/MyImages/raw/master//images/image-20201205200152294.png)

```shell
# docker 查看我们下载的镜像信息
docker images
```

![image-20201205200256031](https://gitee.com/maycopes/MyImages/raw/master//images/image-20201205200256031.png)

```shell
#卸载docker
yum remove docker-ce docker-ce-cli containerd.io
rm -rf /var/lib/docker 
# /var/lib/docker 就是我们docker的默认资源路径。
```

### 阿里云镜像加速

![image-20201205201448228](https://gitee.com/maycopes/MyImages/raw/master//images/image-20201205201448228.png)

![image-20201205201529920](https://gitee.com/maycopes/MyImages/raw/master//images/image-20201205201529920.png)

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://ugs2j40c.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### 如何运行回顾HelloWorld？

![image-20201205202127914](https://gitee.com/maycopes/MyImages/raw/master//images/image-20201205202127914.png)

## Docker的常用命令

### 帮助命令

```shell
docker version    # docker基本的版本
docker info       # 显示docker的系统信息
docker 命令 --help # 万能的帮助命令
# 地址文档 https://docs.docker.com/reference/
```

### 镜像命令

**docker images 用来查询本机上所有的docker镜像。**

```shell
docker images --help

Options:
  -a, --all           # 表示展示所有
  -q, --quiet         # 只展示id
```

**docker search 搜索镜像**

```shell
docker search 名字

docker search mysql --filter=STARS=3000 # 表示的就是说查找收藏在3000以上的。

```

**docker pull 下载镜像** 

```shell
[root@yin ~]# docker pull mysql:5.7      # 使用到标签下载
5.7: Pulling from library/mysql
852e50cd189d: Pull complete   # 这些就是docker的高明的地方，分层概念，是比较深入的概念。
29969ddb0ffb: Pull complete 
a43f41a44c48: Pull complete 
5cdd802543a3: Pull complete 
b79b040de953: Pull complete 
938c64119969: Pull complete 
7689ec51a0d9: Pull complete 
36bd6224d58f: Pull complete 
cab9d3fa4c8c: Pull complete 
1b741e1c47de: Pull complete 
aac9d11987ac: Pull complete 
# 防伪标识。
Digest: sha256:8e2004f9fe43df06c3030090f593021a5f283d028b5ed5765cc24236c2c4d88e
Status: Downloaded newer image for mysql:5.7
docker.io/library/mysql:5.7    # 这个版本的原来的地址。

```

**docker rmi -f 删除镜像**

```shell
docker rmi -f mysql   # 可以通过 id 和名称来进行删除
docker rim -f $(docker images -aq)   # 这样就是删除所有的镜像
```





### 容器命令

说明： 想要进行容器命令的学习就必须先要有一个对应的镜像，因为对于镜像来说就是停止的容器。

```shell
docker pull centos
```

**下载完成之后就是我们容器的启动**

```shell
docker runn [可选参数] image

# 参数说明
--name = "Name"    # 容器的应用，tomcat01，用来区分不同的容器。
-d                 # 以后台的方式进行运行。 
-it                # 使用交互的方法来运行起来。进入容器查看内容
-p                 # 指定容器的端口 
   -p ip 主机端口：容器端口
   -p  主机端口：容器端口
   -p  容器端口
-P                 # 用来指定随机的端口信息。


# 测试演示
[root@yin ~]# docker run -it centos /bin/bash
[root@fc2f64d810a5 /]# ls
bin  etc   lib	  lost+found  mnt  proc  run   srv  tmp  var
dev  home  lib64  media       opt  root  sbin  sys  usr


```

**列出当前正在运行的容器：**

```shell
docker ps      # 就是查看当前在运行的容器
docker ps -a   #表示是说查看之前运行过的容器。
docker ps -a -n=1 # 表示是说查看之前运行的容器中的一个。
docker ps -aq  # 查看所有容器的id号
```

**退出容器**

```shell
eixt    # 就是 直接退出容器，然后通过docker ps 命令来查看
ctrl+P+Q # 注意要在大写的情况下来使用这种就是可以直接退出容器，但是不停止容器。
```

**docker 删除容器**

```shell
docker rm 容器id     # 注意这一r个问题 就是我们有一个正在运行的容器，注意删除容器和删除镜像的区别，一个是docker rm  一个是 docker rmi  然后正在运行的容器时不允许删除的，只有通过 docker rm -f 就是强制删除的意思。
docker rm -f $(docker ps -aq)  # 删除所有的容器
docker rm -a -q |xargs docker rm # 删除所有的容器 使用到的是管道命令。
```

**启动和停止容器的操作**

```shell
docker start 容器id     #容器的
docker restart 容器id
docker stop 容器id
docker kill 容器id

```

### 常用的其他命令

**后台启动一个容器**

```shell
docker run -d centos  # 后台启动一个docker应用。

# 通过 docker ps 来查看的时候 发现centos并不在运行，
# 为什么： 是因为常见docker 是后台运行，此时就必须要有一个对应的前台的进程，dockers发现前台没有对应的应用，就是会自动的停止。
```

**查看日志命令**

```shell
docker logs -tf --tail 条数 id # 用来查看日志信息。

# 自己编写一段shel脚本 表示写入日志。
"while true; do echo maycope;sleep 1;done"
想要停止这个日志信息
docker stop id。
```

**查看进程进行信息：**

```shell
docker ps # 用来查看当前运行的容器
docker top 容器的id信息。
```

**查看镜像的元数据**

```shell
docker inspect  # 容器的id信息
```

**进入当前正在运行的容器**

```shell
# 容器通常都是使用到后台的运行
# 第一种方式进入到正在运行的容器.同时是打开一个新的终端。
[root@yin ~]# docker exec -it a6c7646a9c96 /bin/bash
[root@a6c7646a9c96 /]# ls
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
[root@a6c7646a9c96 /]# ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 03:52 pts/0    00:00:00 /bin/bash
root        14     0  0 04:21 pts/1    00:00:00 /bin/bash

# 第二种方式也是一个进入到容器的迷
docker attach 容器id
  
```

**将容器里面的文件拷贝到外面的宿主机**

```shell
docker cp 容器id:内部路径   外面宿主机的外部路径
[root@yin ~]# docker cp a6c7646a9c96:/home/maycope /home
[root@yin ~]# cd /home
[root@yin home]# ls
git  maycope  project  soft

```

### 命令总结

```shell
attach          # 进入到运行的docker容器里面
build           # 通过DOckerfile定制镜像
commit          # 提交当前容器为新的镜像（后面可能会用到就是制作自己的镜像）
cp              # docker cp id:路径  本地路径; 就是将docker容器里面的文件拷贝到本地上
create          # 创建一个新的容器，同run类似，将镜像进行启动为docker，但是不启动容器。
diff      # 查看docker容器发生的变化
exec      # 同 attach一样 都是进入已经在运行的容器并执行相关连的命令。
images    # 列出当前docker系统有哪些镜像。
info      # 显示出当前系统的相关连的信息。
inspect   # 用来查看当前容器的详细的信息，就是这个容器时哪里来的和其镜像时哪个等等。
kill      # 杀死指定的docker容器
load      # 从一个tar包总加载一个镜像。
login     # 注册或者时登录一个docker源服务器。
logout    # 从当前Docker registry 中退出。
ps        # 用来列出当前还在运行的容器的列表
pull      # 从docker源镜像源服务拉去指定的镜像
resart    # 重新启动一个容器

rm         # docker rm -f (正在运行的只能是通过rm -f 进行删除)
rmi        # 删除的是一个镜像
run       # 创建一个新的容器并运行
save      # 保存一个镜像为tar 包。
search     # 搜索镜像
start     # 启动一个容器
stop       # 停止一个容器。

```

### 练习

> Docker 内部安装vim

```shell
docker exec -it 容器id /bin/bash

apt-get update
 
apt-get install vim
```



#### Nginx

> Docker 安装Nginx

```shell
docker search nginx
docker pull nginx
[root@yin ~]# docker run -d --name nginx01 -p 3344:80 nginx
```

> 说明

```txt
-d 表示是以后台进程进行访问。
--name 表示是重新命名
-p 3344:80 因为对于nginx来说默认启动在80端口，但是是在容器里面，我们想要通过公网的地址对其进行访问，就需要进行端口的映射处理，本宿主机是阿里云的centos 7 ，所以需要去安全组进行设置见下图。开放对应的端口。
```

![image-20201206154144743](https://gitee.com/maycopes/MyImages/raw/master//images/image-20201206154144743.png)

> 测试

就可以通过自己的ip地址进行访问：

![image-20201206154426836](https://gitee.com/maycopes/MyImages/raw/master//images/image-20201206154426836.png)

对于配置过域名映射的小伙伴来说，还可以使用到域名+端口号进行访问，前提是对应的端口号下面没有搭载对应的应用服务。

![image-20201206154602157](https://gitee.com/maycopes/MyImages/raw/master//images/image-20201206154602157.png)

> 思考： 我们现在若是现在想要对nginx进行系列的修改操作，就需要我们每次进入到容器的内部并进入到对应的目录下面才能够修改，后期就可以在容器外面提供一个外部的映射的路径，能够和容器内部的配置文件进行映射，在容器外面修改对应的信息的时候内部也能够做出对应的改变。 利用到 -v 数据卷技术。

#### tomcat

```shell
# 下载与运行
docker search tomcat
docker pull tomcat
docker run -d --name tomcat01 -p 8887:8080 tomcat
# 查看与配置
[root@yin ~]# docker exec -it tomcat01 /bin/bash
root@39772bafd2f5:/usr/local/tomcat# ls -al
# 会发现在webapps下面是空的，因为对于tomcat镜像来说 安装的是最简约版本的。
# 这个时候，就可以把webapps.dist下面的所有的信息都拷贝到webapps下面
[root@yin ~]# docker exec -it tomcat01 /bin/bash
root@39772bafd2f5:/usr/local/tomcat# ls
BUILDING.txt	 LICENSE  README.md	 RUNNING.txt  conf  logs	    temp     webapps.dist
CONTRIBUTING.md  NOTICE   RELEASE-NOTES  bin	      lib   native-jni-lib  webapps  work
root@39772bafd2f5:/usr/local/tomcat# cp -r  webapps.dist/* webapps
root@39772bafd2f5:/usr/local/tomcat# cd webapps
root@39772bafd2f5:/usr/local/tomcat/webapps# ls
ROOT  docs  examples  host-manager  index.html	manager
# 再次访问即可查看到我们目录的相关信息。
```

![image-20201206162742926](https://gitee.com/maycopes/MyImages/raw/master//images/image-20201206162742926.png)

> 思考，现在的部署项目，还是需要我们进入到容器的内部，以后就是希望能够实现在外部挂载一个**webapps**，然后映射到内部，能够进行映射才是最好的方式。
>
> 因为现在就是所有的信息都在容器里面，我们可以对容器进行移除，后期要是出现删除容器，就会出现删除我们所有数据的情况，所以应该是将重要的信息映射到外部



### Docker镜像加载原理与操作

其实就是层级的操作，最开始是只有一层的存在就是最基础的数据，在后面操作的过程中，就是会对层级进行不断地进行添加操作，我们最开始拉取下来的层级就是最基础的镜像，在使用到run命令的候，其实就是在新的层级上对其进行的系列的操作，并不会操作到最底层的数据，然后，就可以使用到**commit**命令来提交自己的镜像。

###  Commit

>  docker commit 提交容器成为一个新的副本
>
>  -m="对提交的信息进行描述"
>
>  -a= "作者"
>
>  目标镜像的名字：[TAG]

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

