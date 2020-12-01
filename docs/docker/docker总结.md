### Docker总结

![image-20201129163631293](.\image-20201129163631293.png)

#### Docker命令帮助文档(重要)

```shell
#机翻
attach 	Attach local standard input, output, and error streams to a running container#将本地标准输入、输出和错误流附加到运行的容器
build 	Build an image from a Dockerfile#从Dockerfile构建一个映像
commit 	Create a new image from a container's changes#根据容器的更改建一个新映像
cp 		Copy files/folders between a container and the local filesystem#在容器和本地文件系统之间复制文件/文件夹
create 	Create a new container#创建一个新的容器
diff 	Inspect changes to files or directories on a container's filesystem#diff检查容器文件系统上文件或目录的更改
events 	Get real time events from the server#事件从服务器获取实时事件
exec 	Run a command in a running container#在正在运行的容器中运行命令
export 	Export a container's filesystem as a tar archive#将容器的文件系统导出为tar归档文件
history Show the history of an image#历史是一个形象的历史
images 	List images#图像列表图片
import 	Import the contents from a tarball to create a filesystem image#从压缩文件中导入内容以创建文件系统映像
info 	Display system-wide information#显示系统范围信息
inspect Return low-level information on Docker objects#检查Docker对象上返回的低级信息
kill 	Kill one or more running containers#杀死一个或多个正在运行的容器
load 	Load an image from a tar archive or STDIN#从tar归档文件或标准输入文件加载映像
login 	Log in to a Docker registry#登录到Docker注册表
logout 	Log out from a Docker registry#从Docker注册表注销
logs 	Fetch the logs of a container#获取容器的日志
pause 	Pause all processes within one or more containers#暂停一个或多个容器中的所有进程
port 	List port mappings or a specific mapping for the container#端口列表端口映射或容器的特定映射
ps 		List containers#列表容器
pull 	Pull an image or a repository from a registry#从注册表中获取图像或存储库
push 	Push an image or a repository to a registry#将图像或存储库推送到注册表
rename 	Rename a container#重命名容器
restart Restart one or more containers#重新启动一个或多个容器
rm 		Remove one or more containers#删除一个或多个容器
rmi 	Remove one or more images#rmi删除一个或多个图像
run 	Run a command in a new container#在新容器中运行命令
save 	Save one or more images to a tar archive (streamed to STDOUT by default)#将一个或多个图像保存到tar归档文件(默认情况下流化为STDOUT)
search 	Search the Docker Hub for images#搜索Docker Hub中的图像
start 	Start one or more stopped containers#启动一个或多个已停止的容器
stats 	Display a live stream of container(s) resource usage statistics#stats显示容器资源使用统计信息的实时流
stop 	Stop one or more running containers#停止一个或多个正在运行的容器
tag 	Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE#创建一个引用SOURCE_IMAGE的标记TARGET_IMAGE
top 	Display the running processes of a container#顶部显示容器正在运行的进程
unpause Unpause all processes within one or more containers#将一个或多个容器中的所有进程暂停
update 	Update configuration of one or more containers#更新一个或多个容器的更新配置
version Show the Docker version information#显示Docker的版本信息
wait 	Block until one or more containers stop, then print their exit codes#等待块，直到一个或多个容器停止，然后打印它们的退出代码
```

##### docker可视化

```shell
#docker run -d -p 8088:9000 --restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer
```



##### 容器数据卷

```shell
#docker run -it -v 主机目录:容器内目录
#docker run -it -v /home/ceshi:/home centos /bin/bash
# docker inspect  
可以查看绑定的信息
"Mounts": [
            {
                "Type": "bind",
                "Source": "/home/ceshi",
                "Destination": "/home",
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            }
        ],

```

好处:以后只需要修改本机文件，不需要进入容器



##### 安装mysql

```shell
官方命令
# docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag

-d 后台启动
-p 端口映射
-v 卷挂载
-e 环境配置
-name 容器名称

#docker pull mysql:5.7
#docker run -d -p 3310:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name=mysql01 mysql:5.7

删除容器后，挂载到本地的数据不会被删除，这就实现了容器内部数据持久化
```

##### 具名和匿名挂载

```shell
#匿名挂载
-v 容器内路径
#docker run -d -P --name nginx -v /etc/nginx nginx

查看所有volume的情况
[root@docker_host data]# docker volume ls
DRIVER              VOLUME NAME
local               c260e9da60bf621b2c3a684f1f8f0f09e63fb6acf83edd1d8771191e580ca903
local               e4055a2013c4c46b1a4295013f50f4046e5b839af403ca208b8744a3ddd62b85

-v 容器内部路径

#具名挂载
#docker run -d -P --name nginx02 -v juming-nginx:/etc/nginx nginx
[root@docker_host data]# docker volume ls
DRIVER              VOLUME NAME
local               c260e9da60bf621b2c3a684f1f8f0f09e63fb6acf83edd1d8771191e580ca903
local               e4055a2013c4c46b1a4295013f50f4046e5b839af403ca208b8744a3ddd62b85
local               juming-nginx
#-v 卷名:容器内部路径
[root@docker_host data]# docker volume  inspect juming-nginx
[
    {
        "CreatedAt": "2020-11-29T05:57:40-05:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/juming-nginx/_data",
        "Name": "juming-nginx",
        "Options": null,
        "Scope": "local"
    }
]
所有的docker容器内的卷，没有指定目录的情况下都是在/var/lib/docker/volumes下的

#如何区分是具名挂载、匿名挂载、还是指定路径挂载
-v 容器内路径 			#匿名挂载
-v 卷名:容器内路径 	   #具名挂载
-v /宿主机路径:容器内路径	 #指定路径挂载


扩展:
#通过-v 容器内路径:ro rw 改变读写权限
ro readonly #只读 文件只能通过宿主机操作，容器内部无法操作
rw readwrite #可读可写

一旦这个了设置了容器权限，客器对我们挂载出来的内容就有限定了!
#docker run -d -P --name nginx02 -v juming-nginx:/etc/nginx:ro nginx
#docker run -d -P --name nginx02 -v juming-nginx:/etc/nginx:rw nginx
```

##### Dockerfile

Dockerfile就是用来构建docker镜像的构建文件

通过这个脚本可以生成镜像，镜像是一层一层的，脚本一个个的命令，每个命令都是一层!

```shell
# cd /
# mkdir docker-test-volume
# cd docker-test-volume/

#创建一个dockerfile文件，名字可以随机建议Dockerfile
#文件中的内容指令(大写)参数

# vim dockerfile1
FROM centos

VOLUME ["volume01","volume02"]

CMD echo "---end---"
CMD /bin/bash
#VOLUME ["volume01","volume02"]  这里匿名挂载了两个目录
#docker build -f /docker-test-volume/dockerfile1 -t charles/centos:1.0 .

```

### 数据卷容器

多个容器间实现数据共享

```shell
创建docker01
[root@docker_host docker-test-volume]# docker run -it --name docker01 charles/centos:1.0
[root@fa95ee773f47 /]# ls -l
ctrl+p+q 退出当前容器
创建docker02
[root@docker_host /]# docker run -it --name docker02 --volumes-from docker01 charles/centos:1.0
[root@65d8e7256d88 /]# ls -l

进入docker01创建文件
[root@docker_host ~]#docker attach fa95ee773f47
[root@fa95ee773f47 /]# cd volume01/
[root@fa95ee773f47 volume01]# touch test.java

查看docker02 volume01目录中是否存在文件
[root@65d8e7256d88 /]# cd volume01/
[root@65d8e7256d88 volume01]# ls
test.java


创建docker03
[root@docker_host /]# docker run -it --name docker03 --volumes-from docker01 charles/centos:1.0
[root@9f230c2ea1f0 volume01]# cd volume01
[root@9f230c2ea1f0 volume01]# touch docker01

#测试可以删除docker01 查看docker02 docker03中共享的文件是否还存在
#存在，共享卷文件是拷贝的
```



##### 多个mysql实现数据共享

```shell
#docker run -d -p 3310:3306 -v /etc/mysql/conf.d -v /var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name=mysql01 mysql:5.7

#docker run -d -p 3311:3306 --volume-from mysql01  -e MYSQL_ROOT_PASSWORD=123456 --name=mysql02 mysql:5.7
```

###### 结论:

容器之间配置信息的传递，数据卷容器的生命周期一直持续到没有容器使用为止.

但是一旦你持久化到了本地，这个时候，本地的数据是不会删除的



### DockerFile

#### DockerFile介绍

dockerfile是用来构建dokcer镜像的文件。命令参数脚本
	构建步骤︰
		1、编写一个dockerfile 文件
		2、docker build构建成为一个镜像
		3、docker run运行镜像
		4、docker push 发布镜像（DockerHub、阿里云镜像仓库!)

#### DockerFile构建过程

​	基础知识∶
​	1、每个保留关键字(指令）都是必须是大写字母
​	2、执行从上到下顺序执行
​	3、#表示注释
​	4、每一个指令都会创建提交一个新的镜像层，并提交。

![image-20201129204309129](.\image-20201129204309129.png)



dockerfile是面向开发的，我们以后要发布项目，做镜像，就需要编写dockerfile文件，这个文件十分简单!

Docker镜像逐渐成为企业交付的标准，必须要掌握

步骤:开发，部署，运维 缺一不可

DockerFile: 		构建文件，定义了一切的步骤，源代码
Dockerlmages:  通过 DockerFile构建生成的镜像，最终发布和运行的产品
Docker容器:	    容器就是镜像运行起来提供服务的



#### DockerFile的指令

```shell
FROM		#基础镜镜像，一切从这里开始构建
MAINTAINER	#镜像是谁写的,姓名+邮箱
RUN			#镜像构建的时候需要运行的命令
ADD			#步骤:tomcat镜像，这个tomcat压缩包!添加内容
WORKDIR		#镜像的工作日录
VOLUME		#挂载的目录
EXPOSE		#保留端口配置
CMD			#指定这个容器启动的时候要运行的命令,只有最后一个会生效，可被替代
ENTRYPOINT	#指定这个容器启动的时候要运行的命令，可以追加命令
ONBUILD 	#当构建一个被继承 DockerFile这个时候就会运行ONBUILD的指令。触发指令。
COPY		#类似ADD ，将我们文件持贝到镜像中
ENV			#构建的时候设置环境变量!
```

![image-20201129204832891](.\image-20201129204832891.png)

##### 实战测试

Docker Hub 中99%镜像都是从这个基础镜像过来的FROM scratch，然后配置需要的软件和配置来进行的构建

![image-20201129205557678](.\image-20201129205557678.png)

> 建一个自己的centos

```shell
#1.编写Dockerfile的文件
[root@docker_host dockerfile]# vim mydockerfile
FROM centos

MAINTAINER charles<test@foxmail.com>

ENV MYPATH /usr/local
WORKDIR MYPATH

RUN yum -y install vim
RUN yun -y install net-tools

EXPOSE 80

CMD echo $MYPATH
CMD echo "-------end---------"
CMD /bin/bash

[root@docker_host dockerfile]# cat mydockerfile 

#2、通过这个文件构建镜像
#命令docker build -f dockerfile文件路径 -t 镜像名:[tag] 镜像保存位置 指定镜像构建过程中的上下文环境的目录
[root@docker_host dockerfile]# docker build -f mydockerfile -t mycentos:0.1 .
```

> CMD 和ENTRYPOINT区别

```shell
CMD			#指定这个容器启动的时侯要运行的命令,只有最后一个会生效，可被替代 cmd
ENTRYPOINT	#指定这个容器启动的时候要运行的命令,可以追加命令  entry point
```

### 实战tomcat镜像

1、准备镜像文件tomcat压缩包，jdk的压缩包

```shell
#docker-tomcat ls
apache-tomcat-9.0.35.tar.gzdockerfile jdk-8u231-linux-x64.tar.gz README
```

2、编写dockerfile文件，官方命名Dockerfile, build会自动寻找这个文件，就不需要-f指定了

```shell
#tomcat dockerfile
FROM centos #
MAINTAINER cheng<1204598429@qq.com>
COPY README /usr/local/README #复制文件
ADD jdk-8u231-linux-x64.tar.gz /usr/local/ #复制解压
ADD apache-tomcat-9.0.35.tar.gz /usr/local/ #复制解压
RUN yum -y install vim
ENV MYPATH /usr/local #设置环境变量
WORKDIR $MYPATH #设置工作目录
ENV JAVA_HOME /usr/local/jdk1.8.0_231 #设置环境变量
ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.35 #设置环境变量
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib #设置环境变量 分隔符是：
EXPOSE 8080 #设置暴露的端口
CMD /usr/local/apache-tomcat-9.0.35/bin/startup.sh && tail -F /usr/local/apache-
tomcat-9.0.35/logs/catalina.out # 设置默认命令
```
```shell
#项目镜像 示例
FROM tomcat-zxhk
LABEL maintainer xxxx@qq.com
RUN rm -rf /usr/local/tomcat/webapps/*
ADD target/*.war /usr/local/tomcat/webapps/ROOT.war 

```

3、构建镜像

```shell
#因为dockerfile命名使用默认命名因此不用使用-f指定文件
# docker build -t mytomcat:0.1 .
```

4.run镜像

```shell
# docker run -d -p 8080:8080 --name tomcat01 -v
/home/charles/build/tomcat/test:/usr/local/apache-tomcat-9.0.35/webapps/test -
v /home/charles/build/tomcat/tomcatlogs/:/usr/local/apache-tomcat-9.0.35/logs
mytomcat:0.1
```

5、访问测试
6、发布项目(由于做了卷挂载，我们直接在本地编写项目就可以发布了！)
发现：项目部署成功，可以直接访问！
我们以后开发的步骤：需要掌握Dockerfile的编写！我们之后的一切都是使用docker镜像来发布运行！



#### 发布镜像

1、地址 https://hub.docker.com/
2、确定这个账号可以登录
3、登录

```shell
# docker login --help
Usage: docker login [OPTIONS] [SERVER]

Log in to a Docker registry.
If no server is specified, the default is defined by the daemon.

Options:
 -p, --password string  Password
     --password-stdin  Take the password from stdin
 -u, --username string  Username
```

```shell
4.push镜像
# 会发现push不上去，因为如果没有前缀的话默认是push到 官方的library
# push镜像的问题?
[root@kuangshen tomcat]# docker push kuangshen/diytomcat:1.0
The push refers to repository [docker.io/kuangshen/diytomcat2]
An image does not exist loca1ly with the tag: kuangshen/diytomcat2

# 解决方法
# 第一种 build的时候添加你的dockerhub用户名，然后在push就可以放到自己的仓库了
$ docker build -t chengcoder/mytomcat:0.1 .
# 第二种 使用docker tag #然后再次push
$ docker tag 容器id chengcoder/mytomcat:1.0 #然后再次push
```

##### 阿里云镜像服务上
1.创建镜像仓库

2.创建命名空间

![image-20201130104814075](.\image-20201130104814075.png)

```shell
#  docker login --username=zchengx registry.cn-shenzhen.aliyuncs.com
#  docker tag [ImageId] registry.cn-shenzhen.aliyuncs.com/dsadxzc/cheng:[镜像
版本号]
# 修改id 和 版本
sudo docker tag a5ef1f32aaae registry.cn-shenzhen.aliyuncs.com/dsadxzc/cheng:1.0
# 修改版本
#  docker push registry.cn-shenzhen.aliyuncs.com/dsadxzc/cheng:[镜像版本号]
```

![image-20201130105231776](.\image-20201130105231776.png)

### Docker网络

#### 理解docker0

```shell
[root@docker_host ~]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo #本机回环地址
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:50:56:20:fe:14 brd ff:ff:ff:ff:ff:ff
    inet 192.168.31.68/24 brd 192.168.31.255 scope global noprefixroute ens33
       valid_lft forever preferred_lft forever#服务器内网地址
    inet6 fe80::e9ad:51bd:b7dd:2ba2/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default #dokcer0地址
    link/ether 02:42:85:98:26:9e brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever

```
###### 问题： docker 是如何处理容器网络访问的？

```shell
# 测试 运行一个tomcat
# docker run -d -P --name  tomcat01 tomcat
[root@docker_host ~]# docker exec -it 701491f95332 ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
4: eth0@if5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
       
       
# 思考？ 
#linux能不能ping通容器内部(172.17.0.2)！ 可以
[root@docker_host ~]# ping 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.043 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.026 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.027 ms


#容器内部可以ping通外界吗？			   可以


```

> 原理

1、我们每启动一个docker容器，docker就会给docker容器分配一个ip，我们只要安装了docker，就会有一个网卡dockero桥接模式，使用的技术是 evth-pair技术

```shell
#启动容器后宿主机多出一个vethd6f3a5c@if4
[root@docker_host ~]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:50:56:20:fe:14 brd ff:ff:ff:ff:ff:ff
    inet 192.168.31.68/24 brd 192.168.31.255 scope global noprefixroute ens33
       valid_lft forever preferred_lft forever
    inet6 fe80::e9ad:51bd:b7dd:2ba2/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:85:98:26:9e brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:85ff:fe98:269e/64 scope link 
       valid_lft forever preferred_lft forever
5: vethd6f3a5c@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default 
    link/ether aa:30:83:b9:60:59 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::a830:83ff:feb9:6059/64 scope link 
       valid_lft forever preferred_lft forever
```

```shell
# 我们发现这个容器带来网卡，都是一对对的
# evth-pair就是一对的虚拟设备接口，他们都是成对出现的，一段连着协议，一段彼此相连
# 正因为有这个特性，evth-pair充当一个桥续，连接各种虚拟网络设备的
# openstac. Docker容器之间的连接，ov5的连接，都是使用evth-pair技术
```

3、我们来测试下tomcat01和tomcat 02是否可以ping通

```shell
# docker run -d -P --name  tomcat02 tomcat
[root@docker_host ~]# docker exec -it tomcat02 ping 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.047 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.035 ms

```

![image-20201130114009967](.\image-20201130114009967.png)

Docker中的所有的网络接口都是虚拟的。虚拟的转发效率高

#### --link

> 思考一个场景，我们编写了一个微服务，database url-=ip:，项目不重启，数据库ip换掉了，我们希望可以处理这个问题，可以名字来进行访问容器?

```shell
[root@docker_host ~]# docker exec -it tomcat02 ping tomcat01
ping: tomcat01: Name or service not known


#如何解决?
[root@docker_host ~]# docker run -d -P  --name tomcat03 --link tomcat02  tomcat
ca323bd1143c08f2f446cc5c4d4e4ef6b3c356f4fe4bb9d5bd349ae7c665e1aa
[root@docker_host ~]# docker exec -it tomcat03 ping tomcat02
PING tomcat02 (172.17.0.3) 56(84) bytes of data.
64 bytes from tomcat02 (172.17.0.3): icmp_seq=1 ttl=64 time=0.052 ms
64 bytes from tomcat02 (172.17.0.3): icmp_seq=2 ttl=64 time=0.035 ms

#反向可以ping通吗?  不可以
[root@docker_host ~]# docker exec -it tomcat02 ping tomcat03
ping: tomcat03: Name or service not known


[root@docker_host ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
252423688cce        bridge              bridge              local
d9a9ca610f88        host                host                local
6154940e2b46        none                null                local

[root@docker_host ~]# docker inspect 252423688cce
[
    {
        "Name": "bridge",
        "Id": "252423688cce46eab0e7203d52b7f85b69f105e0c02917cb3e4858bb503694ff",
        "Created": "2020-11-29T21:07:23.355541017-05:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "701491f95332517ec3281baa55cf1e2245fb687a2a09707180e756c01a5adb85": {
                "Name": "tomcat01",
                "EndpointID": "e7475a9cc4cc16813e8523d5485146b7ca54b45e8c8a4c78de98ccc88efd1ecf",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            },
            "84fa8cc9d92747255dbe90fff0681b2223ee7490579229849539fa286487dc60": {
                "Name": "tomcat02",
                "EndpointID": "7c8b75afc786a8b06906a1f4163d7c7d9adddd68addffd75f35ca38d1979c7ad",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            },
            "ca323bd1143c08f2f446cc5c4d4e4ef6b3c356f4fe4bb9d5bd349ae7c665e1aa": {
                "Name": "tomcat03",
                "EndpointID": "00d515c112f62c35209d7e5141285458b73aec7598a0d28d9a47d6b16b8001bd",
                "MacAddress": "02:42:ac:11:00:04",
                "IPv4Address": "172.17.0.4/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]

#查看hostst配置
[root@docker_host ~]# docker exec -it tomcat03 cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.3	tomcat02 84fa8cc9d927
172.17.0.4	ca323bd1143c

#不建议使用--link

```

#### 自定义网络

###### 网络模式

bridge :	桥接docker(默认)
none:		不配置网络
host :		和宿主机共享网络
container :容器网络连通!（用的少   局限大)

```shell
#我们直接启动的命令--net bridge，而这个就是我们的docker0 
#docker run -d -P --name tomcat01 tomcat
#docker run -d -P --name tomcat01 --net bridge tomcat

#docker0特点，默认，容器名不能访问，--link可以打通连接!

[root@docker_host ~]# docker network create --help

Usage:	docker network create [OPTIONS] NETWORK

Create a network

Options:
      --attachable           Enable manual container attachment
      --aux-address map      Auxiliary IPv4 or IPv6 addresses used by Network driver (default map[])
      --config-from string   The network from which copying the configuration
      --config-only          Create a configuration only network
  -d, --driver string        Driver to manage the Network (default "bridge")
      --gateway strings      IPv4 or IPv6 Gateway for the master subnet
      --ingress              Create swarm routing-mesh network
      --internal             Restrict external access to the network
      --ip-range strings     Allocate container ip from a sub-range
      --ipam-driver string   IP Address Management Driver (default "default")
      --ipam-opt map         Set IPAM driver specific options (default map[])
      --ipv6                 Enable IPv6 networking
      --label list           Set metadata on a network
  -o, --opt map              Set driver specific options (default map[])
      --scope string         Control the network's scope
      --subnet strings       Subnet in CIDR format that represents a network segment

#我们可以自定义一个网络!
# --driver bridge 
# --subnet 192.168.0.0/16 
# --gateway 192.168.0.1 

[root@docker_host ~]# docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 
mynet1e80625e1b1a3be8ee0b5a752894d844f70926162fdb79c4ac6a702dbab395a7
[root@docker_host ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
252423688cce        bridge              bridge              local
d9a9ca610f88        host                host                local
1e80625e1b1a        mynet               bridge              local
6154940e2b46        none                null                local

[root@docker_host ~]# docker network inspect mynet
[
    {
        "Name": "mynet",
        "Id": "1e80625e1b1a3be8ee0b5a752894d844f70926162fdb79c4ac6a702dbab395a7",
        "Created": "2020-11-29T23:03:04.32354186-05:00",
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
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]


#启动容器使用自定义网络
[root@docker_host ~]# docker run -d -P --name tomcat01 --net mynet tomcat
52526b606e679e632fea61e1c3696bfa25a6d5c23188c264546553e5fae1eacd
[root@docker_host ~]# docker run -d -P --name tomcat02 --net mynet tomcat
6c71daf35716959fb711252d8ffd1ea6cc3c98415114be0fd209937ea570f76c
[root@docker_host ~]# docker network inspect mynet
[
    {
        "Name": "mynet",
        "Id": "1e80625e1b1a3be8ee0b5a752894d844f70926162fdb79c4ac6a702dbab395a7",
        "Created": "2020-11-29T23:03:04.32354186-05:00",
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
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "52526b606e679e632fea61e1c3696bfa25a6d5c23188c264546553e5fae1eacd": {
                "Name": "tomcat01",
                "EndpointID": "ea8eecf01542d3171c223cd903aa2f0edf7dec3634069d5c8260ba683fb9008f",
                "MacAddress": "02:42:c0:a8:00:02",
                "IPv4Address": "192.168.0.2/16",
                "IPv6Address": ""
            },
            "6c71daf35716959fb711252d8ffd1ea6cc3c98415114be0fd209937ea570f76c": {
                "Name": "tomcat02",
                "EndpointID": "35db3e06a8554ab75507368a24780d07db35068ba9092ede0cbf259ed1bef73a",
                "MacAddress": "02:42:c0:a8:00:03",
                "IPv4Address": "192.168.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]

#再次测试ping连接
[root@docker_host ~]# docker exec -it tomcat02 ping 192.168.0.2
PING 192.168.0.2 (192.168.0.2) 56(84) bytes of data.
64 bytes from 192.168.0.2: icmp_seq=1 ttl=64 time=0.052 ms
64 bytes from 192.168.0.2: icmp_seq=2 ttl=64 time=0.033 ms


#不适用--line也可以访问tomcat01
[root@docker_host ~]# docker exec -it tomcat02 ping tomcat01
PING tomcat01 (192.168.0.2) 56(84) bytes of data.
64 bytes from tomcat01.mynet (192.168.0.2): icmp_seq=1 ttl=64 time=0.028 ms
64 bytes from tomcat01.mynet (192.168.0.2): icmp_seq=2 ttl=64 time=0.034 ms

好处:
redis -不同的集群使用不同的网络，保证集群是安全和健康的
mysql -不同的集群使用不同的网络，保证集群是安全和健康的

```

#### 网络连通

![image-20201130121807530](.\image-20201130121807530.png)

```shell
[root@docker_host ~]# docker run -d -P --name tomcat03 tomcat
4cadec657c99b77bd4df978fcb5528c461c3dedf99db03b43bde82589f8c22d8
[root@docker_host ~]# docker exec -it tomcat03 ping tomcat01
ping: tomcat01: Name or service not known

#测试打通tomcat03 - mynet
[root@docker_host ~]# docker network connect mynet tomcat03
[root@docker_host ~]# docker network inspect mynet
#连通之后将tomcat03放到了mynet网络下
#一个容器两个ip
[
    {
        "Name": "mynet",
        "Id": "1e80625e1b1a3be8ee0b5a752894d844f70926162fdb79c4ac6a702dbab395a7",
        "Created": "2020-11-29T23:03:04.32354186-05:00",
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
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "4cadec657c99b77bd4df978fcb5528c461c3dedf99db03b43bde82589f8c22d8": {
                "Name": "tomcat03",
                "EndpointID": "ee10f8fbdd6788ae3991fdffc50dd96921d56fb7419a404d2a35934eddbc888a",
                "MacAddress": "02:42:c0:a8:00:04",
                "IPv4Address": "192.168.0.4/16",
                "IPv6Address": ""
            },
            "52526b606e679e632fea61e1c3696bfa25a6d5c23188c264546553e5fae1eacd": {
                "Name": "tomcat01",
                "EndpointID": "ea8eecf01542d3171c223cd903aa2f0edf7dec3634069d5c8260ba683fb9008f",
                "MacAddress": "02:42:c0:a8:00:02",
                "IPv4Address": "192.168.0.2/16",
                "IPv6Address": ""
            },
            "6c71daf35716959fb711252d8ffd1ea6cc3c98415114be0fd209937ea570f76c": {
                "Name": "tomcat02",
                "EndpointID": "35db3e06a8554ab75507368a24780d07db35068ba9092ede0cbf259ed1bef73a",
                "MacAddress": "02:42:c0:a8:00:03",
                "IPv4Address": "192.168.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]

[root@docker_host ~]# docker exec -it tomcat03 ping tomcat01
PING tomcat01 (192.168.0.2) 56(84) bytes of data.
64 bytes from tomcat01.mynet (192.168.0.2): icmp_seq=1 ttl=64 time=0.044 ms
64 bytes from tomcat01.mynet (192.168.0.2): icmp_seq=2 ttl=64 time=0.035 ms
#结论:假设要跨网络操作别人，就需要使用docker nefwork connect连通
```

### redis集群

![image-20201130121925483](.\image-20201130121925483.png)

```shell
#创建网卡
docker network create redis --subnet 172.38.0.0/16

#通过脚本创建六个redis配置
for port in $(seq 1 6);\
do \
mkdir -p /mydata/redis/node-${port}/conf
touch /mydata/redis/node-${port}/conf/redis.conf
cat << EOF >> /mydata/redis/node-${port}/conf/redis.conf
port 6379
bind 0.0.0.0
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
cluster-announce-ip 172.38.0.1${port}
cluster-announce-port 6379
cluster-announce-bus-port 16379
appendonly yes
EOF
done

# 通过脚本运行六个redis
for port in $(seq 1 6);\
do \
docker run -p 637${port}:6379 -p 1667${port}:16379 --name redis-${port} -v /mydata/redis/node-${port}/data:/data -v /mydata/redis/node-${port}/conf/redis.conf:/etc/redis/redis.conf -d --net redis --ip 172.38.0.1${port} redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf
done
 
#启动集群
docker exec -it redis-1 /bin/sh #redis默认没有bash
redis-cli --cluster create 172.38.0.11:6379 172.38.0.12:6379 172.38.0.13:6379 172.38.0.14:6379 172.38.0.15:6379 172.38.0.16:6379 --cluster-replicas 1

```



### Docker Compose

> 官方介绍

 定义运行多个文件

YAML file配置文件

single command 命令有哪些？

Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application’s services. Then, with a single command, you create and start all the services from your configuration. To learn more about all the features of Compose, see [the list of features](https://docs.docker.com/compose/#features).

所有的环境都可以使用Compose

Compose works in all environments: production, staging, development, testing, as well as CI workflows. You can learn more about each case in [Common Use Cases](https://docs.docker.com/compose/#common-use-cases).

三个步骤

Using Compose is basically a three-step process:

1. Define your app’s environment with a `Dockerfile` so it can be reproduced anywhere.

   ​	Dockerfile保证我们的项目在任何地方可以运行。

2. Define the services that make up your app in `docker-compose.yml` so they can be run together in an isolated environment.

   ​	services什么是服务

   ​	docker-compose.yml文件怎么写

3. Run `docker-compose up` and Compose starts and runs your entire app.

   ​	启动项目

作用:批量容器编排。

Compose 是Docker官方的开源项目。需要安装

Dokcerfile 在任何地方可以运行.web服务器 redis mysql nginx集群

```
version: "3.8"
services:
  web:
    build: .
    ports:
      - "5000:5000"
    volumes:
      - .:/code
      - logvolume01:/var/log
    links:
      - redis
  redis:
    image: redis
volumes:
  logvolume01: {}
```

Compose :重要的概念。
​		服务services，容器。应用。(web、redis、mysql....）
​		项目project。一组关联的容器。

#### 安装

1.下载 

```shell
sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

2.修改权限

```shell
sudo chmod +x /usr/local/bin/docker-compose
```

3.建立软连接

```shell
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

4.查看版本 测试是否安装成功

```shell
[root@docker_host ~]# docker-compose --version
docker-compose version 1.27.4, build 40524192
```



###### 体验https://docs.docker.com/compose/gettingstarted/

###### Step 1: Setup 

​	Define the application dependencies.

1. Create a directory for the project:

   ```shell
   $ mkdir composetest
   $ cd composetest
   ```

2. Create a file called `app.py` in your project directory and paste this in:

   ```shell
   import time
   
   import redis
   from flask import Flask
   
   app = Flask(__name__)
   cache = redis.Redis(host='redis', port=6379)
   
   def get_hit_count():
       retries = 5
       while True:
           try:
               return cache.incr('hits')
           except redis.exceptions.ConnectionError as exc:
               if retries == 0:
                   raise exc
               retries -= 1
               time.sleep(0.5)
   
   @app.route('/')
   def hello():
       count = get_hit_count()
       return 'Hello World! I have been seen {} times.\n'.format(count)
   ```

   In this example, `redis` is the hostname of the redis container on the application’s network. We use the default port for Redis, `6379`.

3.Create another file called `requirements.txt` in your project directory and paste this in:

```shell
flask
redis
```



###### Step 2: Create a Dockerfile

https://docs.docker.com/compose/gettingstarted/#step-2-create-a-dockerfile

​	In this step, you write a Dockerfile that builds a Docker image. The image contains all the dependencies 	the Python application requires, including Python itself.

​	In your project directory, create a file named `Dockerfile` and paste the following:

```shell
FROM python:3.7-alpine
WORKDIR /code
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
EXPOSE 5000
COPY . .
CMD ["flask", "run"]
```

This tells Docker to:

- Build an image starting with the Python 3.7 image.
- Set the working directory to `/code`.
- Set environment variables used by the `flask` command.
- Install gcc and other dependencies
- Copy `requirements.txt` and install the Python dependencies.
- Add metadata to the image to describe that the container is listening on port 5000
- Copy the current directory `.` in the project to the workdir `.` in the image.
- Set the default command for the container to `flask run`.

For more information on how to write Dockerfiles, see the [Docker user guide](https://docs.docker.com/develop/) and the [Dockerfile reference](https://docs.docker.com/engine/reference/builder/).

###### Step 3: Define services in a Compose file

​	Create a file called `docker-compose.yml` in your project directory and paste the following:

```shell
version: "3.8"
services:
  web:
    build: .
    ports:
      - "5000:5000"
  redis:
    image: "redis:alpine"
```

​	This Compose file defines two services: `web` and `redis`.

###### Web service

The `web` service uses an image that’s built from the `Dockerfile` in the current directory. It then binds the container and the host machine to the exposed port, `5000`. This example service uses the default port for the Flask web server, `5000`.

###### Redis service

The `redis` service uses a public [Redis](https://registry.hub.docker.com/_/redis/) image pulled from the Docker Hub registry.



###### Step 4: Build and run your app with Compose

1. From your project directory, start up your application by running `docker-compose up`.

   ```shell
   $ docker-compose up
   
   Creating network "composetest_default" with the default driver
   Creating composetest_web_1 ...
   Creating composetest_redis_1 ...
   Creating composetest_web_1
   Creating composetest_redis_1 ... done
   Attaching to composetest_web_1, composetest_redis_1
   web_1    |  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
   redis_1  | 1:C 17 Aug 22:11:10.480 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
   redis_1  | 1:C 17 Aug 22:11:10.480 # Redis version=4.0.1, bits=64, commit=00000000, modified=0, pid=1, just started
   redis_1  | 1:C 17 Aug 22:11:10.480 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
   web_1    |  * Restarting with stat
   redis_1  | 1:M 17 Aug 22:11:10.483 * Running mode=standalone, port=6379.
   redis_1  | 1:M 17 Aug 22:11:10.483 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
   web_1    |  * Debugger is active!
   redis_1  | 1:M 17 Aug 22:11:10.483 # Server initialized
   redis_1  | 1:M 17 Aug 22:11:10.483 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
   web_1    |  * Debugger PIN: 330-787-903
   redis_1  | 1:M 17 Aug 22:11:10.483 * Ready to accept connections
   ```

   Compose pulls a Redis image, builds an image for your code, and starts the services you defined. In this case, the code is statically copied into the image at build time.

2. Enter http://localhost:5000/ in a browser to see the application running.

   If you’re using Docker natively on Linux, Docker Desktop for Mac, or Docker Desktop for Windows, then the web app should now be listening on port 5000 on your Docker daemon host. Point your web browser to http://localhost:5000 to find the `Hello World` message. If this doesn’t resolve, you can also try http://127.0.0.1:5000.

   If you’re using Docker Machine on a Mac or Windows, use `docker-machine ip MACHINE_VM` to get the IP address of your Docker host. Then, open `http://MACHINE_VM_IP:5000` in a browser.

   You should see a message in your browser saying:



```
[root@docker_host ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
6d836d1690d4        redis:alpine        "docker-entrypoint.s…"   2 minutes ago       Up 2 minutes        6379/tcp                 composetest_redis_1
9f92589f183c        composetest_web     "flask run"              2 minutes ago       Up 2 minutes        0.0.0.0:5000->5000/tcp   composetest_web_1

```



###### 停止

```shell
docker-compose down 或者ctrl+c
```

#### YAML规则

###### YAML 配置文件

https://docs.docker.com/compose/compose-file/#compose-file-structure-and-examples

```shell
#3层
version: ""#版本
services: #服务
	服务1: web
		images
		build
		network
		.....
	服务2:
#其他配置 网络/卷、全局规则
volumes:
networks:
configs:


```

##### 开源项目

###### 博客https://docs.docker.com/compose/wordpress

1. Create an empty project directory.

   You can name the directory something easy for you to remember. This directory is the context for your application image. The directory should only contain resources to build that image.

   This project directory contains a `docker-compose.yml` file which is complete in itself for a good starter wordpress project.

   > **Tip**: You can use either a `.yml` or `.yaml` extension for this file. They both work.

2. Change into your project directory.

   For example, if you named your directory `my_wordpress`:

   ```shell
   mkdir my_wordpress
   cd my_wordpress/
   ```

3. Create a `docker-compose.yml` file that starts your `WordPress` blog and a separate `MySQL` instance with a volume mount for data persistence:

   ```shell
   version: '3.3'
   
   services:
      db:
        image: mysql:5.7
        volumes:
          - db_data:/var/lib/mysql
        restart: always
        environment:
          MYSQL_ROOT_PASSWORD: somewordpress
          MYSQL_DATABASE: wordpress
          MYSQL_USER: wordpress
          MYSQL_PASSWORD: wordpress
   
      wordpress:
        depends_on:
          - db
        image: wordpress:latest
        ports:
          - "8000:80"
        restart: always
        environment:
          WORDPRESS_DB_HOST: db:3306
          WORDPRESS_DB_USER: wordpress
          WORDPRESS_DB_PASSWORD: wordpress
          WORDPRESS_DB_NAME: wordpress
   volumes:
       db_data: {}
   ```

http://192.168.31.68:8000/



### Docker Swarm

https://docs.docker.com/engine/swarm/how-swarm-mode-works/nodes/

Docker Engine 1.12 introduces swarm mode that enables you to create a cluster of one or more Docker Engines called a swarm. A swarm consists of one or more nodes: physical or virtual machines running Docker Engine 1.12 or later in swarm mode.

There are two types of nodes: [**managers**](https://docs.docker.com/engine/swarm/how-swarm-mode-works/nodes/#manager-nodes) and [**workers**](https://docs.docker.com/engine/swarm/how-swarm-mode-works/nodes/#worker-nodes).

![image-20201130152915372](.\image-20201130152915372.png)



初始化节点`docker swarm init`

docker swarm init --advertise-addr 192.168.31.71

加入一个节点`docker swarm join`

```shell
#获取令牌
docker swarm join-token manager #添加一个管理节点
docker swarm join-tolen worker #添加一个工作节点
```

##### Raft协议
​	双主双从:假设一个节点挂了!其他节点是否可以用!
​	Raft协议:保证大多数节点存活才可以用。只要>1，集群至少大于3台

​	实验:
​	1、将docker1机器停止。宕机!双主，另外一个主节点也不能使用了



`docker service`

​	体验:创建服务、动态扩展服务、动态更新服务。

​	灰度发布:金丝雀发布!

```shell
#docker service create -p 8888:80 --name my-nginx nginx

docker run容器启动!不具有扩缩容器
docker service服务 具有扩缩容器，滚动更新 

查看服务
#docker service ps
#docker service inspect my-nginx

服务，集群中任意的节点都可以访问。服务可以有多个副本动态扩缩容实现高可用!
动态括缩容
#docker service update --replicas 3 my-nginx

#docker service scale my-nginx=5 #和上面一样
```

##### 概念总结

swarm

集群的管理和编号。docker可以初始化一个swarm集群，其他节点可以加入。(管理、工作者)

node
就是一个docker节点。多个节点就组成了一个网络集群。(管理、工作者)

service
任务，可以在管理节点或者工作节点来运行。核心。!用户访问!

Task
容器内的命令，细节任务!



![image-20201130160200481](.\image-20201130160200481.png)



![image-20201130160421648](.\image-20201130160257513.png)

![image-20201130160325633](.\image-20201130160325633.png)

![image-20201130160515200](.\image-20201130160515200.png)



调用service以什么方式运行 --mode

```shell
#docker service create --mode relicated --name mytom tomcat:7 默认的
#docker service create --mode global --name mytom tomcat:7  
```



### Dock Stack

docker-compose 单机部署项目!
docker stack部署，热群部署!

```shell
#单机
docker-compose up -d wordpress.yaml
#集群
docker stack deploy wordpress.yaml
```



### Docker Secret

安全证书



### Docker Config

配置

