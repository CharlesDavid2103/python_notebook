### K8s

##### 概念:

###### 	什么是k8s：

​		k8s是一组服务器集群

​		k8s所管理的是集群节点上的容器

##### k8s的功能

​		自我修复:容器宕机会自动重启
​		弹性伸缩:实时根据服务器的并发情况，增加或缩减容器的数量
​		自动部署和回滚
​		服务发现和负载均衡
​		机密和配置共享管理
​		存储编排
​		批处理

##### k8s集群分为两类节点
​		master node :主
​		woker node:工作

###### 		master节点的组件(程序)

| 名称                    | 功能                                                         |
| ----------------------- | ------------------------------------------------------------ |
| kube-apiserver          | Kubernetes API，集群的统一入口，各组件协调者，以RESTful API提供接口服务，所有对象资源的增删改查和监听操作都交给APIServer处理后再提交给Etcd存储。 |
| schduler                | 根据调度算法为新创建的Pod选择一个Node节点，可以任意部署,可以部署在同一个节点上也可以部署在不同的节点上。 |
| kube-controller-manager | 处理集群中常规后台任务，一个资源对应一个控制器，而ControllerManager就是负责管理这些控制器的。 |
| etcd                    | 分布式键值存储系统。用于保存集群状态数据，比如Pod、Service等对象信息。 |

###### 		worker节点主键

| 名称           | 功能                                                         |
| -------------- | ------------------------------------------------------------ |
| kubelet        | kubelet是Master在Node节点上的Agent，管理本机运行容器的生命周期，比如创建容器、Pod挂载数据卷、下载secret、获取容器和节点状态等工作。kubelet将每个Pod转换成一组容器。 |
| kuber-proxy    | 在Node节点上实现Pod网络代理，维护网络规则和四层负载均衡工作。 |
| docker或rocket | 容器引擎，运行容器。                                         |

#### Kubernets核心概念

##### Pod

​		最小部署单元
​		一组容器的集合
​		一个Pod中的容器共享网络命名空间
​		Pod是短暂的

##### Controllers

​		ReplicaSet :确保预期的Pod副本数量
​		Deployment :无状态应用部署
​		StatefulSet :有状态应用部署
​		DaemonSet :确保所有Node运行同一个Pod
​		Job :一次性任务
​		Cronjob :定时任务

##### Service

​		防止Pod失联
​		定义一组Pod的访问策略

##### Label
​		标签，附加到某个资源上，用于关联对象、查询和筛选
##### Namespace
​		命名空间，将对象逻辑上隔离



### 搭建一个完整的Kubernetes集群

##### 1、生产环境K8S平台规划

###### 	架构类别

​		单master
​		多master - 生成环境

###### 	生成环境k8s规划

​		master建议3台
​		etcd至少3台(3、5、7)选择基数是为了etcd自动能够投票选出主etcd
​		worker越多越好

##### 2、服务器硬件配置推荐

| 实验环境 | k8s master/node | 2Cpu 2G |      |
| :------: | :-------------: | :-----: | :--: |
| 测试环境 |   k8s-master    |   cpu   | 2核  |
|          |                 |  内存   |  4G  |
|          |                 |  硬盘   | 20G  |
|          |    k8s-node     |   cpu   | 4核  |
|          |                 |  内存   |  8G  |
|          |                 |  硬盘   | 20G  |
| 生产环境 |   k8s-master    |   cpu   | 8核  |
|          |                 |  内存   | 16G  |
|          |                 |  硬盘   | 100G |
|          |    k8s-node     |   cpu   | 16核 |
|          |                 |  内存   | 64G  |
|          |                 |  硬盘   | 500G |



##### 3、官方提供三种部署方式

###### minikube

​		Minikube是一个工具，可以在本地快速运行一个单点的Kubernetes，
​		仅用于尝试Kubernetes·部署地址: https://kubernetes.io/docs/setup/minikube/

###### kubeadm
​		Kubeadm也是工具，提供kubeadm init和kubeadm join，用于快速部署Kubernetes集群。
​		部署地址: https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm/

###### 二进制
​		推荐，从官方下载发行版的二进制包，手动部署每个组件，组成Kubernetes集群。
​		下载地址: https://github.com/kubernetes/kubernetes/releases

##### 4、为Etcd和APISever自签SSL证书

​		**加解密**

​			对称加密:加密解密使用相同的密钥

​			非对称加密:加密解密使用密钥对(公钥私钥)公钥加密、私钥解密

​			单项加密:智能加密不能解密MD5 md5sum md5sum -c能够查看文件校验值是否相同

##### 	SSL证书和相关概念
​		PKI (Public Key Infrastructure公钥基础设)

###### 		一个完整的PKI包括以下几个部分

​		1、端实体（申请者）
​		2、注册结构（RC)
​		3、签证机构（CA）
​		4、证书撤销列表（CRL)
​		5、证书存取库
​				**证书内容:**
​				数字证书版本号
​				证序列号证书签名算法证书颁发者
​				证书有效期
​				对象名称
​				对象的公开密钥
​				(其他扩展信息)
​				数字签名

###### 	**ssl证书来源**

​		网络第三方机构购买，通常这种证书是用于让外部用户访问使用
​		自己给自己发证书-自签证书

​		**自建CA**
​			openssl
​			cfssl

##### 5、Etcd数据库集群部署

​		二进制包下载地址

​			https://github.com/etcd-io/etcd/releases

​		查看集群状态

​			/opt/etcd/bin/etcdctl --ca-file=/opt/etcd/ssl/ca.pem --cert-file=/opt/etcd/ssl/server.pem --key-		file=/opt/etcd/ssl/server-key.pem --endpoints="https://192.168.31.63:2379,https://192.168.31.64:2379,https://192.168.31.65:2379" cluster-health

##### 6、 部署Master组件

​		部署Master组件
​		1.kube-apiserver
​		2.kube-controller-manager
​		3.kube-scheduler
​		配置文件-> systemd管理组件->启动

##### 7、 部署Node组件

​		1.docker
​		2. kubelet
​		3. kube-proxy
​		配置文件-> systemd管理组件->启动

##### 8、 部署K8S集群网络

##### 9、部署Web UI (Dashboard)

##### 10、部署集群内部DNS解析服务(CoreDNS)



###### 部署单master集群

###### 	一、集群规划

​			master1
​				主机名:k8s-master1
​				IP:192.168.31.63
​			worker1
​				主机名:k8s-node1
​				IP:192.168.31.65
​			worker2
​				主机名:k8s-node2
​				IP:192.168.31.66

​			k8s版本:1.16
​			安装方式:二进制
​			操作系统版本:CentOs7.7



###### 	二、初始化服务器

```shell
		修改ip
​		# vim /etc/sysconfig/network-scripts/ifcfg-ens33

​		# service network restart

​		1 关闭防火墙
​			【所有节点都要执行】
​			systemctl stop firewalld
​			systemctl disable firewalld
​		2 关闭selinux
​			【所有节点都要执行】
​			#setenforce 0
​			vim /etc/selinux/config
​			修改SETLINUX=enforcing为SETLINUX=disabled
​		3 配置主机名
​			【所有节点都要执行】
​			hostnamectl set-hostname k8s-master1
​			hostnamectl set-hostname k8s-node1
​			hostnamectl set-hostname k8s-node2
​		4 配置名称解析
​			【所有节点都要执行】
​			#vim /etc/hosts
​			所有节点添加如下四行
​				192.168.31.63			k8s-master1
​				192.168.31.64			k8s-master2
​				192.168.31.65			k8s-node1
​				192.168.31.66			k8s-node2
​		5 配置时间同步
​			选择一个节点作为服务端，剩下的作为客户端
​			master1为时间服务暑器的服务端
​			其他的为时间服务器的客户端
​			1) 配置k8s-master
​				#yum install chrony -y
​				#vim /etc/chrony.conf
​				#修改三项
​				server 127.0.0.1 iburst
​				allow 192.168.31.0/24
​				local stratum 10
​				#启动chrony
​				#systemctl start chronyd
​				#systemctl enable chronyd
​				#ss  -unl | grep 123
​			3)配置k8s-node1和k8s-node2
​				#yum install chrony -y
​				#vim /etc/chrony.conf
​				#修改三项
​				server  192.168.31.63  iburst
​				#启动chrony
​				#systemctl start chronyd
​				#systemctl enable chronyd
​		6 关闭交换分区
​			swapoff -a
​			vim /etc/fstab #删除最后一行/dev/mapper/centos-swap swap
​			检查是否关闭成功
​			free -m
```



###### 	三、给etcd颁发证书

​		流程简介
​			1) 创建证书颁发机构
​			2) 填写表单--写明etcd所在节点的IP
​			3) 向证书颁发机构申请证书

```shell
		第一步︰上传TLS安装包
​				传到/root下
​		第二步︰
​				#tar xvf /root/TLS.tar.gz
​				#cd /root/TLS
​				#./cfssl.sh
​				#cd etcd
​				#vim server-csr.json
​						修改host中的IP地址，这里的IP是etcd所在节点的IP地址
​						{
​						"CN" : "etcd",
​						"hosts": [
​								"192.168.31.63",
​								"192.168.31.65",
​								"192.168.31.66"
​						],"key" : {
​								"algo" :"rsa",
​								"size": 2048
​						},
​						"names" :[
​									{
​									"c":"CN",
​									"L":“BeiJing",
​									"ST":“BeiJing"
​									}
​							]
​					}
​			# ./generate_etcd_cert.sh 
​			# ls *pem
​				ca-key. pem ca.pem server-key. pem server.pem
```

###### 		四、部署etcd

​			etcd需要三台虚拟机
​			在master、node1、node2上分别安装一个etcd

```shell
注意∶
	解压之后会生成一个文件和一个目录
    #tar xvf etcd.tar.gz
	#mv etcd.service /usr/lib/systemd/system
	#mv etcd  /opt/
	#vim /opt/etcd/cfg/etcd.conf
		#[Member]
        ETCD_NAME="etcd-1"
        ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
        ETCD_LISTEN_PEER_URLS="https://192.168.31.63:2380"
        ETCD_LISTEN_CLIENT_URLS="https://192.168.31.63:2379"

        #[Clustering]
        ETCD_INITIAL_ADVERTISE_PEER_URLS="https://192.168.31.63:2380"
        ETCD_ADVERTISE_CLIENT_URLS="https://192.168.31.63:2379"
        ETCD_INITIAL_CLUSTER="etcd-1=https://192.168.31.63:2380,etcd-					2=https://192.168.31.65:2380,etcd-3=https://192.168.31.66:2380"
        ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
        ETCD_INITIAL_CLUSTER_STATE="new"
	# rm -rf /opt/etcd/ssl/*
	# cd /root/TLS/etcd/
	# \cp -fv ca.pem server.pem server-key.pem /opt/etcd/ssl/
	
	将etc管理程序和程序目录发送到node1 和node2
	# scp /usr/lib/systemd/system/etcd.service root@k8s-node1:/usr/lib/systemd/system/
	# scp /usr/lib/systemd/system/etcd.service root@k8s-node2:/usr/lib/systemd/system/
	# scp -r /opt/etcd/ root@k8s-node1:/opt/
	# scp -r /opt/etcd/ root@k8s-node2:/opt/
	
	在node1上修改etcd的配置文件
	#vim /opt/etcd/cfg/etcd.conf
	ETCD_NAME="etcd-2"
    ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
    ETCD_LISTEN_PEER_URLS="https://192.168.31.65:2380"
    ETCD_LISTEN_CLIENT_URLS="https://192.168.31.65:2379"

    #[Clustering]
    ETCD_INITIAL_ADVERTISE_PEER_URLS="https://192.168.31.65:2380"
    ETCD_ADVERTISE_CLIENT_URLS="https://192.168.31.65:2379"
    ETCD_INITIAL_CLUSTER="etcd-1=https://192.168.31.63:2380,etcd-2=https://192.168.31.65:2380,etcd-3=https://192.168.31.66:2380"
    ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
    ETCD_INITIAL_CLUSTER_STATE="new"
    
	在node2上修改etcd的配置文件
	#vim /opt/etcd/cfg/etcd.conf
	ETCD_NAME="etcd-3"
    ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
    ETCD_LISTEN_PEER_URLS="https://192.168.31.66:2380"
    ETCD_LISTEN_CLIENT_URLS="https://192.168.31.66:2379"

    #[Clustering]
    ETCD_INITIAL_ADVERTISE_PEER_URLS="https://192.168.31.66:2380"
    ETCD_ADVERTISE_CLIENT_URLS="https://192.168.31.66:2379"
    ETCD_INITIAL_CLUSTER="etcd-1=https://192.168.31.63:2380,etcd-2=https://192.168.31.65:2380,etcd-3=https://192.168.31.66:2380"
    ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
    ETCD_INITIAL_CLUSTER_STATE="new"

	在三个节点一次后动etcd服务
	#systemctl start etcd
	#systemctl enable etcd
	检查是否后动成功
# /opt/etcd/bin/etcdctl --ca-file=/opt/etcd/ssl/ca.pem --cert-file=/opt/etcd/ssl/server.pem --key-file=/opt/etcd/ssl/server-key.pem --endpoints="https://192.168.31.63:2379,https://192.168.31.64:2379,https://192.168.31.65:2379" cluster-health
member 72130f86e474b7bb is healthy: got healthy result from https://192.168.31.66:2379
member b46624837acedac9 is healthy: got healthy result from https://192.168.31.63:2379
member fd9073b56d4868cb is healthy: got healthy result from https://192.168.31.65:2379
cluster is healthy
```

​			centos7
​				systemd服务管理脚本在哪个目录?/usr/lib/systemd/system
​			centos6
​				sysV风格服务管理脚本在哪?/etc/rc.d/rcN.d
​									N代表管理级别0 1 2 3 4 5 6

###### 	五、为api server签发证书

```shell
#cd /root/TLS/k8s/
#./generate_k8s_cert.sh
```



###### 	六、部署master服务	

```shell
# cd /root/
#tar xvf k8s-master.tar.gz
#mv kube-apiserver.service kube-controller-manager.service kube-scheduler.service /usr/lib/systemd/system/
#mv kubernetes /opt/
#cp /root/TLS/k8s/{ca*pem,server.pem,server-key.pem} /opt/kubernetes/ssl/ -rvf
```

​		修改apiserver的配置文件

```shell
#vim /opt/kubernetes/cfg/kube-apiserver.conf 

KUBE_APISERVER_OPTS="--logtostderr=false \
--v=2 \
--log-dir=/opt/kubernetes/logs \
#这里修改一行修改为各etcd的ip地址
--etcd-servers=https://192.168.31.63:2379,https://192.168.31.65:2379,https://192.168.31.66:2379 \
#这里修改一行修改为master的ip地址
--bind-address=192.168.31.63 \
--secure-port=6443 \
#这里修改一行修改为master的ip地址
--advertise-address=192.168.31.63 \
--allow-privileged=true \
--service-cluster-ip-range=10.0.0.0/24 \
--enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,ResourceQuota,NodeRestriction \
--authorization-mode=RBAC,Node \
--enable-bootstrap-token-auth=true \
--token-auth-file=/opt/kubernetes/cfg/token.csv \
--service-node-port-range=30000-32767 \
--kubelet-client-certificate=/opt/kubernetes/ssl/server.pem \
--kubelet-client-key=/opt/kubernetes/ssl/server-key.pem \
--tls-cert-file=/opt/kubernetes/ssl/server.pem  \
--tls-private-key-file=/opt/kubernetes/ssl/server-key.pem \
--client-ca-file=/opt/kubernetes/ssl/ca.pem \
--service-account-key-file=/opt/kubernetes/ssl/ca-key.pem \
--etcd-cafile=/opt/etcd/ssl/ca.pem \
--etcd-certfile=/opt/etcd/ssl/server.pem \
--etcd-keyfile=/opt/etcd/ssl/server-key.pem \
--audit-log-maxage=30 \
--audit-log-maxbackup=3 \
--audit-log-maxsize=100 \
--audit-log-path=/opt/kubernetes/logs/k8s-audit.log"
```

 		启动master

```shell
# systemctl start kube-apiserver
# systemctl enable kube-apiserver
# systemctl start kube-scheduler
# systemctl enable kube-scheduler
# systemctl start kube-controller-manager
# systemctl enable kube-controller-manager
# ps aux | grep kub
# ps aux | grep kub | wc -l
#cp /opt/kubernetes/bin/kubectl /bin/
# kubectl get cs
# cat /var/log/messages|grep kube-apiserver|grep Error

配置tls基于bootstrap自动颁发证书
#kubectl create clusterrolebinding kubelet-bootstrap --clusterrole=system:node-bootstrapper --user=kubelet-bootstrap

```

###### 七、部署Node组件

```shell
#安装worker node节点
	#docker:启动容器
	#kubelet:接受apiserver的指令，然后控制docker容器
	#kube-proxy:为worker上的容器配置网络工作
	第一步:安装配置docker
        # tar xvf k8s-node.tar.gz 
        # mv docker.service /usr/lib/systemd/system
        # mkdir /etc/docker
        # cp daemon.json /etc/docker/
        # tar xf docker-18.09.6.tgz 
        # mv docker/* /bin/
        # systemctl start docker
        # systemctl enable docker
        # docker info
        出现警告
        WARNING: bridge-nf-call-iptables is disabled
        WARNING: bridge-nf-call-ip6tables is disabled

        #vi /etc/sysctl.conf
        添加下面两行
        net.bridge.bridge-nf-call-ip6tables = 1
		net.bridge.bridge-nf-call-iptables = 1
		
		#sysctl -p
		
    第二步:安装kubelet和kube-proxy
		1)生成程序目录和管理脚本
		# mv kubelet.service kube-proxy.service /usr/lib/systemd/system/
		# mv kubernetes /opt/
		
		2)修改配置文件(4个)
		#vim /opt/kubernetes/cfg/kube-proxy.kubeconfig
            apiVersion: v1
            clusters:
            - cluster:
                certificate-authority: /opt/kubernetes/ssl/ca.pem
                #这里修改一行修改为master的ip地址
                server: https://192.168.31.63:6443         
              name: kubernetes
            contexts:
            - context:
                cluster: kubernetes
                user: kube-proxy
              name: default
            current-context: default
            kind: Config
            preferences: {}
            users:
            - name: kube-proxy
              user:
                client-certificate: /opt/kubernetes/ssl/kube-proxy.pem
                client-key: /opt/kubernetes/ssl/kube-proxy-key.pem
                
          #vim /opt/kubernetes/cfg/kube-proxy-config.yml
          	kind: KubeProxyConfiguration
            apiVersion: kubeproxy.config.k8s.io/v1alpha1
            address: 0.0.0.0
            metricsBindAddress: 0.0.0.0:10249
            clientConnection:
              kubeconfig: /opt/kubernetes/cfg/kube-proxy.kubeconfig
            #这里修改一行修改为当前node主机名
            hostnameOverride: k8s-node1
            clusterCIDR: 10.0.0.0/24
            mode: ipvs
            ipvs:
              scheduler: "rr"
            iptables:
              masqueradeAll: true
              
            #vim /opt/kubernetes/cfg/kubelet.conf 
              KUBELET_OPTS="--logtostderr=false \
            --v=2 \
            --log-dir=/opt/kubernetes/logs \
             #这里修改一行修改为当前node主机名
            --hostname-override=k8s-node1 \
            --network-plugin=cni \
            --kubeconfig=/opt/kubernetes/cfg/kubelet.kubeconfig \
            --bootstrap-kubeconfig=/opt/kubernetes/cfg/bootstrap.kubeconfig \
            --config=/opt/kubernetes/cfg/kubelet-config.yml \
            --cert-dir=/opt/kubernetes/ssl \
            --pod-infra-container-image=lizhenliang/pause-amd64:3.0"
		
			#vim /opt/kubernetes/cfg/bootstrap.kubeconfig
			apiVersion: v1
            clusters:
            - cluster:
            certificate-authority: /opt/kubernetes/ssl/ca.pem
            #这里修改一行修改为master的ip地址
            server: https://192.168.31.63:6443
            name: kubernetes
            contexts:
            - context:
            cluster: kubernetes
            user: kubelet-bootstrap
            name: default
            current-context: default
            kind: Config
            preferences: {}
            users:
            - name: kubelet-bootstrap
            user:
            token: c47ffb939f5ca36231d9e3121a252940
		3)从master节点复制证书到worker节点
			#scp /root/TLS/k8s/{ca.pem,kube-proxy-key.pem,kube-proxy.pem} root@k8s-node1:/opt/kubernetes/ssl/
			
		4)启动kubelet和kube-proxy服务
           	# systemctl start kubelet
            # systemctl start kube-proxy
            # systemctl enable kubelet
            # systemctl enable kube-proxy
            #tail -f /opt/kubernetes/logs/kubelet.INFO
            如果看到最好一行信息是如下内容，就表示启动服务正常
            I1127 04:02:12.739317   12044 bootstrap.go:150] No valid private key and/or certificate found, reusing existing private key or creating a new one
		5)在master节点为worker节点颁发证书
			# kubectl get csr
			#注意名称使用worker名称
			# kubectl certificate approve node-csr-CrnUKpLuLNVktpxCJDjlGQGo3IYAJq2kOV_YNVm0s18
		6)给Worker节点颁发证书之后，就可以在master上看到worker节点了
			#kubectl get nodes
			
			
		先修改node1和node2的docker配置文件
		#vim /etc/ docker/daemon.json
		{
            "registry-mirrors": [
            "https://docker.mirrors.ustc.edu.cn",	
            "http://bc437cce.m.daocloud.io"
            ],
            "insecure-registries": ["192.168.31.70"]
    }
    #systemctl daemon-reload
	#systemctl restart docker
		
	第三步:安装网络插件
		1)确认启用CNI
			# grep "cni" /opt/kubernetes/cfg/kubelet.conf 
		2)安装CNI
			# mkdir -pv /opt/cni/bin /etc/cni/net.d
			# tar xf cni-plugins-linux-amd64-v0.8.2.tgz -C /opt/cni/bin
		3）在master执行yaml脚本，实现在worker节点安装启动网络插件功能
			# cd /root/YAML/
			# kubectl delete -f kube-flannel.yaml
			# kubectl apply -f kube-flannel.yaml
			注意:
				kubectl apply 操作受限于网络，可能会需要5~10分钟才能执行成功如果网上太慢，会导致超时
			# kubectl get pods -n kube-system
			# kubectl get nodes
	第四步:授权apiserver可以访问kubelet
			# kubectl apply -f apiserver-to-kubelet-rbac.yaml
	查看节点描述信息
	# kubectl describe node k8s-node1

```

###### 八、启动nginx容器

​	1)修改node1和node2的docker配置文件

​				#vim /etc/ docker/daemon.json

```json
{
    "registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn",	
    "http://bc437cce.m.daocloud.io"
    ],
    "insecure-registries": ["192.168.31.70"]
}
```
2)重启服务
	#systemctl daemon-reload
	#systemctl restart docker

3)在master上启动nginx

```shell
创建deployment，通过deployment来创建和管理nginx容器
	# kubectl create deployment myweb --image=nginx:1.8
查看一下deployment的状态
	# kubectl get deployment
查看pods的状态
	# kubectl get pods
	# kubectl describe pod myweb-5b79bf86d4-r9cdf


```

4) 暴露myweb的端口到物理机

```shell

# kubectl expose deployment myweb --port=80 --type=NodePort
# kubectl get svc

```
5) 访问集群任意节点的20828来访问nginx

```shell
# curl http://192.168.31.65:20828/
```

######  九、配置web界面

​		官方: kubernetes dashboard 

```shell
# kubectl apply -f /root/YAML/dashboard.yaml
# kubectl get pods -n kubernetes-dashboard
# kubectl get svc -n kubernetes-dashboard
```

​		第三方: kuboard

```shell
worker节点
	#docker load -i kuboard-1.tar.gz
master节点
    修改一行，指定希望kuboard运行在哪个节点:
    # vim start_kuboard.yaml
        nodeName: master修改为	nodeName: k8s-node1
    # kubectl apply -f start_kuboard.yaml
 	# kubectl get pods -n kube-system
 	查看一下kuboard的暴辉的端口
	# kubectl get svc -n kube-system
生成tocken
	#kubectl -n kube-system get secret $(kubectl -n kube-system get secret | grep kuboard-user | awk '{print $1}') -o go-template='{{.data.token}}' | base64 -d

```

###### 十、woker安装dns组件

```shell
k8s中用来实现名称解析的是：CoreDNS
# cd /root/YAML/
# kubectl apply -f coredns.yaml 
# kubectl get pods -n kube-system | grep 
```

###### 十一、远程管理k8s

```shell
默认情况下，k8s仅仅可以在master节点进行管理操作
1）将管理工具复制到worker节点【master执行】
	[root@k8s-master1 ~]# scp /bin/kubectl root@k8s-node2:/bin

2）生成管理员证书【master执行】
    [root@k8s-master1 ~]# cd /root/TLS/k8s/
    [root@k8s-master1 k8s]# vim admin-csr.json

    写入如下内容：
        {
          "CN": "admin",
          "hosts": [],
          "key": {
            "algo": "rsa",
            "size": 2048
          },
          "names": [
            {
              "C": "CN",
              "L": "BeiJing",
              "ST": "BeiJing",
              "O": "system:masters",
              "OU": "System"
            }
          ]
        }

    颁发admin证书
    [root@k8s-master1 k8s]# cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes admin-csr.json | cfssljson -bare admin

3）创建kubeconfig文件【master执行】
    设置集群参数
        [root@k8s-master1 k8s]# kubectl config set-cluster kubernetes \
       --server=https://192.168.31.63:6443 \
       --certificate-authority=ca.pem \
       --embed-certs=true \
       --kubeconfig=config

    设置客户端认证参数
        [root@k8s-master1 k8s]# kubectl config set-credentials cluster-admin \
        --certificate-authority=ca.pem \
        --embed-certs=true \
        --client-key=admin-key.pem \
        --client-certificate=admin.pem \
        --kubeconfig=config

    设置上下文参数
        [root@k8s-master1 k8s]# kubectl config set-context default \
           --cluster=kubernetes \
           --user=cluster-admin \
           --kubeconfig=config
	设置默认上下文
        [root@k8s-master1 k8s]# kubectl config use-context default --kubeconfig=config

 4）将生产的config文件发送到worker节点【master执行】
    	[root@k8s-master1 k8s]# scp config root@k8s-node2:/root/

 5）在worker节点，基于config实现执行kubectl命令【worker执行】
	[root@k8s-node2 ~]# kubectl get nodes --kubeconfig=/root/config
	
	NAME        STATUS   ROLES    AGE    VERSION
	k8s-node1   Ready    <none>   2d4h   v1.16.0
	k8s-node2   Ready    <none>   47h    v1.16.0
	
	[root@k8s-node2 ~]# mkdir ~/.kube
	[root@k8s-node2 ~]# mv /root/config /root/.kube/
	[root@k8s-node2 ~]# kubectl get nodes
	NAME        STATUS   ROLES    AGE    VERSION
	k8s-node1   Ready    <none>   2d4h   v1.16.0
	k8s-node2   Ready    <none>   47h    v1.16.0
```

###### k8s使用入门

​	案例：管理和使用deployment

```shell
deployment
	创建指定数量的pod
	检查pod健康状态和数量

1）基于deployment创建nginx pod，有一个副本
	方法1：
		[root@k8s-master1 k8s]# kubectl run nginx-dep1 --image=nginx:1.8 --replicas=1
	方法2：
		通过kuboard创建pod，http://k8s-node1:32567/
		1.名称（应用）空间选择default
		2.创建工作负载
	方法3：
		首先创建yaml文件ngx_dep.yaml
		#cd /root/
		#vim ngx-dep.yaml
		写入以下内容
		apiVersion: apps/v1
        kind: Deployment
        metadata:
            name: ngx-dep3
            labels:
                app: nginx
                type: webservice
        spec:
            replicas: 1
            selector: 
                matchLabels:
                     app: ngx
            template:
                 metadata:
                   labels:
                      app: ngx
                 spec:
                  containers:
                    - name: nginx
                      image: nginx:1.8

		然后执行yaml文件
			# kubectl apply -f ngx-dep.yaml 

2）查看k8s对象状态
	# kubectl get 资源类型
	# kubectl describe 资源类型
	# kubectl logs 显示pod中的容器中运行过程中产生的日志信息
	# kubectl exec -it pod对象 /bin/sh
	资源类型：
				node
				pod
				service
				deployment
				ns
				....
			选项：
			-n namesapce
			-A
	
    例子：显示所有名称空间中的pod
    # kubectl get pod -A -o wide
    
3）发布应用
     用service来发布服务
		
     首先创建yaml
     # vim ngx_svc.yaml
			apiVersion: v1
            kind: Service
            metadata:
                name: ngx-svc
                labels:
                  app: ngx
            spec:
                selector:
                  app: ngx
                ports:
                  - name: nginx-ports
                    protocol: TCP
                    port: 80
                    nodePort: 32002
                    targetPort: 80
                type: NodePort
	
    执行yaml
    	# kubectl apply -f ngx_svc.yaml

    此时可以看到service的端口映射信息
   		# kubectl get svc

    此时，可以访问集群中的任意一个node节点，来访问web页面
		http://k8s-node1:32002/

 4）服务伸缩（scalling）
	根据客户端的请求流量实现弹性管理
	修改yaml中的replicas
	# vim ngx-dep.yaml 
	# kubectl apply -f ngx-dep.yaml 
 5）滚动更新
 	修改yaml中的image版本 image: nginx:1.7.9
 	# vim ngx-dep.yaml 
	# kubectl apply -f ngx-dep.yaml  
```

![image-20201128164635280](.\image-20201128164635280.png)

![image-20201128164737652](.\image-20201128164737652.png)

 

###### 案例：将一个JAVA项目迁移到k8s

```shell
	注意：
		在k8s环境中，开发交付镜像，而不是源码
	规划
        k8s
            master：192.168.31.63
            worker：192.168.31.63  192.168.31.65   192.168.31.66
        mysql
            mariadb: 192.168.31.67
        项目：
            java
        代码：
            java代码


第一步：制作镜像
	运行环境镜像：tomcat-base-image.tar.gz
	1）在所有的node节点导入
		[root@k8s-node1 ~]# docker load -i tomcat-base-image.tar.gz 
		[root@k8s-node2 ~]# docker load -i tomcat-base-image.tar.gz 

	2）在67节点安装mysql，并导入数据库
        [root@sql ~]# yum install mariadb-server -y
        [root@sql ~]# systemctl start mariadb
        [root@sql ~]# systemctl enable mariadb

		[root@sql ~]# mysql
        MariaDB [(none)]> grant all on mydb.* to "test"@"%" identified by "123";
        Query OK, 0 rows affected (0.00 sec)

        MariaDB [(none)]> flush privileges;
        Query OK, 0 rows affected (0.00 sec)


        MariaDB [(none)]> use test;
        Database changed
        MariaDB [test]> show tables;
        Empty set (0.00 sec)

        MariaDB [test]> source /root/tables_ly_tomcat.sql
        Query OK, 1 row affected, 1 warning (0.00 sec)

        Database changed
        Query OK, 0 rows affected (0.01 sec)

        Query OK, 0 rows affected (0.00 sec)

        MariaDB [test]> show tables;
        +----------------+
        | Tables_in_test |
        +----------------+
        | user           |
        +----------------+
        1 row in set (0.00 sec)

        MariaDB [test]> select * from user;
        Empty set (0.00 sec)

        MariaDB [test]> describe user
        -> ;
        +-------+--------------+------+-----+---------+----------------+
        | Field | Type         | Null | Key | Default | Extra          |
        +-------+--------------+------+-----+---------+----------------+
        | id    | int(11)      | NO   | PRI | NULL    | auto_increment |
        | name  | varchar(100) | NO   |     | NULL    |                |
        | age   | int(3)       | NO   |     | NULL    |                |
        | sex   | char(1)      | YES  |     | NULL    |                |
        +-------+--------------+------+-----+---------+----------------+
        4 rows in set (0.00 sec)
		
        MariaDB [test]> quit
        Bye
        如果不是test库需要授权
        MariaDB [(none)]> grant all on mydb.* to "test"@"%" identified by "123";
        MariaDB [(none)]> flush privileges;

        在master节点测试是否可以登录mysql
        # yum install mysql -y
        mysql -utest -p -h192.168.31.67

	3）在master节点使用java demo
        [root@k8s-master1 ~]# tar xvf java-demo.tar.gz 
        [root@k8s-master1 ~]# cd tomcat-java-demo-master/
        [root@k8s-master1 ~]# cd tomcat-java-demo-master/src/main/resources/
        [root@k8s-master1 resources]# vim application.yml 
	
	修改三行
        url: jdbc:mysql://192.168.31.67:3306/mydb?characterEncoding=utf-8
        username: test
        password: 123


	4）将javademo构建到docker 镜像中
        [root@k8s-master1 ~]# yum install maven java-1.8.0-openjdk -y
        [root@k8s-master1 ~]# cd tomcat-java-demo-master/
        [root@k8s-master1 ~]# mvn clean package -Dmaven.test-skip=true

	5）使用dockerfile构建镜像
        [root@k8s-master1 ~]# cd tomcat-java-demo-master/
        [root@k8s-master1 tomcat-java-demo-master]# docker build -t javapro .
        [root@k8s-master1 tomcat-java-demo-master]# docker image ls
	
	
	6)保存javapro到本地
	[root@k8s-master1 ~]# docker save javapro>./javapro.tar.gz
	
	7)从master拷贝到node, 

	[root@k8s-master1 ~]# scp javapro.tar.gz root@k8s-node1:/root/
	[root@k8s-master1 ~]# scp javapro.tar.gz root@k8s-node2:/root/
 	8)所有的node节点导入 
    [root@k8s-node1 ~]# docker load -i javapro.tar.gz
    [root@k8s-node2 ~]# docker load -i javapro.tar.gz 
	
	
第二步：启动pod	
	1) aml文件，然后进行修改
        修改
            修改副本数   replicas: 2
            修改镜像拉取策略   imagePullPolicy: Never
	[root@k8s-master1 ~]# kubectl create deployment javapro1 --image=javapro --dry-run -o yaml>javapro.yaml
	apiVersion: apps/v1
    kind: Deployment
    metadata:
      labels:
        app: javapro1
      name: javapro1
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: javapro1
      strategy: {}
      template:
        metadata:
          labels:
            app: javapro1
        spec:
          containers:
          - image: javapro
            imagePullPolicy: Never
            name: javapro


2)启动
[root@k8s-master1 tomcat-java-demo-master]# kubectl apply -f javapro.yaml 
 3)映射端口
[ root@k8s-master1 ~]# kubectl expose deployment javapro1 --target-port=8080 --type=NodePort --port=80 --dry-run -o yaml>javasvc.yaml
[root@k8s-master1 ~]# kubectl apply -f javasvc.yaml 
[root@k8s-master1 ~]# kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
javapro1     NodePort    10.0.0.148   <none>        80:30243/TCP   3s
kubernetes   ClusterIP   10.0.0.1     <none>        443/TCP        22h
myweb        NodePort    10.0.0.196   <none>        80:30073/TCP   7h45m
ngx-svc      NodePort    10.0.0.203   <none>        80:32002/TCP   4h5m

访问任意node的30243端口
http://k8s-node1:30243/

```





###############################################################
devops
	python
	docker+k8s
	git+jenkins
	
k8s
	docker
	docker 三剑客（compose，swam、machine）

pod 与 pod控制器
	一个pod中是一个或者多个容器
	kod控制器是用来控制、管理pod的数量、状态

service
	由于pod的地址会发生改变，通过service 可以为pod提供一个统一的访问入口

############################################################		

k8s集群中的lb的实现方式
	自建lb：
	使用运营商的lb：如果你用阿里云主机部署k8s集群，那么建议用运营商

自建lb方式
	lvs
	HAproxy
	nginx

当前解决方案
	nginx+keepalived





###### 实现k8s的高可用

​	添加一个master节点，解决前面配置中的master节点的单点故障问题

```shell
大概流程：
	再启动三个虚拟机
	一个作为master
	另外两个作为lb的主和备
			第一步：配置master2
			
			1）初始化服务器
                [root@k8s-master2 ~]# hostnamectl set-hostname k8s-master2 
                [root@k8s-master2 ~]# systemctl stop firewalld
                [root@k8s-master2 ~]# systemctl disable firewalld
                [root@k8s-master2 ~]# getenforce 0
                [root@k8s-master2 ~]# vim /etc/selinux/config 
                [root@k8s-master2 ~]#  
                [root@k8s-master2 ~]# swapoff -a
                [root@k8s-master2 ~]# vim /etc/fstab 
                [root@k8s-master2 ~]# vim /etc/hosts
                [root@k8s-master2 ~]# yum install chrony -y
                [root@k8s-master2 ~]# vim /etc/chrony.conf 
                [root@k8s-master2 ~]# systemctl restart chronyd
                [root@k8s-master2 ~]# systemctl enable chroynd
                [root@k8s-master2 ~]# systemctl enable chronyd
                [root@k8s-master2 ~]# 
                [root@k8s-master2 ~]# chronyc sources
			【如果不清楚初始化过程，请查看前面的笔记】
			
		安装k8s组件
        	为了简单，从master1上复制配置好的k8s组件
                [root@k8s-master1 ~]# scp -r /opt/etcd/ root@k8s-maste2:/opt/
                [root@k8s-master1 ~]# scp -r /opt/kubernetes root@k8s-master2:/opt/
                [root@k8s-master1 ~]# cd /usr/lib/systemd/system
                [root@k8s-master1 system]# scp kube-apiserver.service  kube-controller-manager.service  kube-scheduler.service root@k8s-master2:/usr/lib/systemd/system
                [root@k8s-master1 system]# scp /bin/kubectl root@k8s-master2:/bin/
                
       		在master2上修改apiserver的配置文件
            	[root@k8s-master2 ~]# vi /opt/kubernetes/cfg/kube-apiserver.conf
        	修改两行：分别制定当前节点自己监听的IP
                --bind-address=192.168.31.64 \
                --advertise-address=192.168.31.64 \
                
		 	启动master2的服务了
                [root@k8s-master2 ~]# systemctl daemon-reload
                [root@k8s-master2 ~]# systemctl restart kube-apiserver
                [root@k8s-master2 ~]# systemctl restart kube-controller-manager
                [root@k8s-master2 ~]# systemctl restart kube-scheduler
                [root@k8s-master2 ~]# systemctl enable kube-apiserver
                [root@k8s-master2 ~]# systemctl enable kube-controller-manager
                [root@k8s-master2 ~]# systemctl enable kube-scheduler
                
      		 检查是否启动成功
                [root@k8s-master2 ssl]# ps aux | grep kube
                [root@k8s-master2 ssl]# kubectl get nodes
                
        2）安装部署lb
            启用两个节点，分布安装nginx、keepalived

            首先进行两个节点的初始化
            略

            然后，在两个节点分布安装nginx
                [root@lb-backup ~]# rpm ivh nginx-1.16.1-1.el7.ngx.x86_64.rpm 
                [root@lb-backup ~]# vim /etc/nginx/nginx.conf 
                    stream {
                        log_format  main  '$remote_addr $upstream_addr - [$time_local] $status $upstream_bytes_sent';
                        access_log  /var/log/nginx/k8s-access.log  main;
                        upstream k8s-apiserver {
                                    server 192.168.31.63:6443;
                                    server 192.168.31.64:6443;
                                }
                        server {
                           listen 6443;
                           proxy_pass k8s-apiserver;
                        }
                    }

                [root@lb-backup ~]# systemctl restart nginx
                [root@lb-backup ~]# systemctl enable nginx
                [root@lb-backup ~]# ss -tnl | grep 80
                LISTEN     0      128          *:80      *:*                 
        	
        	安装keepalived，实现高可用
                [root@lb-backup ~]# yum instll keepalived -y

            用给你的配置文件，替换master和backup上的配置文件
          		  略

            修改配置文件

            查看是有vip
```

###### 