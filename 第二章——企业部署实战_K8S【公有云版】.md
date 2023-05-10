## 第二章——企业部署实战_K8S【公有云版】

### 前言：

注：公有云版未测试是否适配第三章往后内容，理论上是完全可行的。

**考虑到部分同学机器配置并无法满足多开VMware，这里推出公有云版本（以阿里云为例）**



##### 前言：如果你是新手，机器的名字及各种账户密码一定要和我的一样，先学一遍，再自己改

> **WHAT**：用于管理云平台中多个主机上的容器化的应用，Kubernetes的目标是让部署容器化的应用简单并且高效（powerful）,Kubernetes提供了应用部署，规划，更新，维护的一种机制
>
> **WHY**：为什么使用它，因为它是管理docker容器最主流的编排工具

- Pod
  - Pod是K8S里能够被运行的最小的逻辑单元（原子单元）
  - 1个Pod里面可以运行多个容器，它们共享UTS+NET+IPC名称空间
  - 可以把Pod理解成豌豆荚，而同一Pod内的每个容器是一颗颗豌豆
  - 一个Pod里运行多个容器，又叫边车（SideCar）模式
- Pod控制器（关于更多<a href="https://github.com/ben1234560/k8s_PaaS/blob/master/%E5%8E%9F%E7%90%86%E5%8F%8A%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/Kubernetes%E5%9F%BA%E6%9C%AC%E6%A6%82%E5%BF%B5.md#%E5%88%9D%E8%AF%86pod">初识Pod</a>）
  - Pod控制器是Pod启动的一种模板，用来保证在K8S里启动的Pod始终按照人们的预期运行（副本数、生命周期、健康状态检查...）
  - Pod内提供了众多的Pod控制器，常用的有以下几种：
    - Deployment
    - DaemonSet
    - ReplicaSet
    - StatefulSet
    - Job
    - Cronjob
- Name
  - 由于K8S内部，使用“资源”来定义每一种逻辑概念（功能），故每种“资源”，都应该有自己的“名称”
  - “资源”有api版本（apiVersion）类别（kind）、元数据（metadata）、定义清单（spec）、状态（status）等配置信息
  - “名称”通常定义在“资源”的“元数据”信息里
- <a href="https://github.com/ben1234560/k8s_PaaS/blob/master/%E5%8E%9F%E7%90%86%E5%8F%8A%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/Docker%E5%9F%BA%E7%A1%80.md#%E5%85%B3%E4%BA%8Enamespace">namespace</a>
  - 随着项目增多、人员增加、集群规模的扩大，需要一种能够隔离K8S内各种“资源”的方法，这就是名称空间
  - 名称空间可以理解为K8S内部的虚拟集群组
  - 不同名称空间内的“资源”名称可以相同，相同名称空间内的同种“资源”、“名称”不能相同
  - 合理的使用K8S名称空间，使得集群管理员能够更好的对交付到K8S里的服务进行分类管理和浏览
  - K8S内默认存在的名称空间有：default、kube-system、kube-public
  - 查询K8S里特定“资源”要带上相应的名称空间
- Label
  - 标签是K8S特色的管理方式，便于分类管理资源对象
  - 一个标签可以对应多个资源，一个资源也可以有多个标签，它们是多对多的关系
  - 一个资源拥有多个标签，可以实现不同维度的管理
  - 标签的组成：key=value
  - 与标签类似的，还有一种“注解”（annotations）
- Label选择器
  - 给资源打上标签后，可以使用标签选择器过滤指定的标签
  - 标签选择器目前有两个：基于等值关系（等于、不等于）和基于集合关系（属于、不属于、存在）
  - 许多资源支持内嵌标签选择器字段
    - matchLabels
    - matchExpressions
- Service
  - 在K8S的世界里，虽然每个Pod都会被分配一个单独的IP地址，但这个IP地址会随着Pod的销毁而消失
  - Service（服务）就是用来解决这个问题的核心概念
  - 一个Service可以看作一组提供相同服务的Pod的对外访问接口
  - Service作用与哪些Pod是通过标签选择器来定义的
- Ingress
  - Ingress是K8S集群里工作在OSI网络参考模型下，第7层的应用，对外暴露的接口
  - Service只能进行L4流量调度，表现形式是ip+port
  - Ingress则可以调度不同业务域、不同URL访问路径的业务流量

简单理解：Pod可运行的原子，name定义名字，namespace名称空间（放一堆名字），label标签（另外的名字），service提供服务，ingress通信

### K8S架构图（并非传统意义上的PaaS服务，而是IaaS服务）

![1582188308711](assets/1582188308711.png)

> **kubectl**：Kubernetes集群的命令行接口
>
> **API Server**：的核心功能是对核心对象（例如：Pod，Service，RC）的增删改查操作，同时也是集群内模块之间数据交换的枢纽
>
> **Etcd**：包含在 APIServer 中，用来存储资源信息
>
> **Controller Manager **：负责维护集群的状态，比如故障检测、自动扩展、滚动更新等
>
> **Scheduler**：负责资源的调度，按照预定的调度策略将Pod调度到相应的机器上。可以通过这些有更深的了解：
>
> - <a href="https://github.com/ben1234560/k8s_PaaS/blob/master/%E5%8E%9F%E7%90%86%E5%8F%8A%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/Kubernetes%E8%B0%83%E5%BA%A6%E6%9C%BA%E5%88%B6.md">Kubernetes调度机制</a>
> - <a href="https://github.com/ben1234560/k8s_PaaS/blob/master/%E5%8E%9F%E7%90%86%E5%8F%8A%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/Kubernetes%E8%B0%83%E5%BA%A6%E6%9C%BA%E5%88%B6.md#kubernetes%E7%9A%84%E8%B5%84%E6%BA%90%E6%A8%A1%E5%9E%8B%E4%B8%8E%E8%B5%84%E6%BA%90%E7%AE%A1%E7%90%86">Kubernetes的资源模型与资源管理</a>
> - <a href="https://github.com/ben1234560/k8s_PaaS/blob/master/%E5%8E%9F%E7%90%86%E5%8F%8A%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/Kubernetes%E8%B0%83%E5%BA%A6%E6%9C%BA%E5%88%B6.md#kubernetes%E9%BB%98%E8%AE%A4%E7%9A%84%E8%B0%83%E5%BA%A6%E7%AD%96%E7%95%A5">Kubernetes默认的调度策略</a>
> - <a href="https://github.com/ben1234560/k8s_PaaS/blob/master/%E5%8E%9F%E7%90%86%E5%8F%8A%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/Kubernetes%E8%B0%83%E5%BA%A6%E6%9C%BA%E5%88%B6.md#%E8%B0%83%E5%BA%A6%E5%99%A8%E7%9A%84%E4%BC%98%E5%85%88%E7%BA%A7%E4%B8%8E%E5%BC%BA%E5%88%B6%E6%9C%BA%E5%88%B6">调度器的优先级与强制机制</a>
>
> **kube-proxy**：负责为Service提供cluster内部的服务发现和负载均衡
>
> **Kubelet**：在Kubernetes中，应用容器彼此是隔离的，并且与运行其的主机也是隔离的，这是对应用进行独立解耦管理的关键点。[Kubelet工作原理解析](https://github.com/ben1234560/k8s_PaaS/blob/master/%E5%8E%9F%E7%90%86%E5%8F%8A%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/Kubernetes%E8%B0%83%E5%BA%A6%E6%9C%BA%E5%88%B6.md#kubelet)
>
> **Node**：运行容器应用，由Master管理

### 我们部署的K8S架构图

![1584700891603](assets/1584700891603.png)

> 可以简单理解成：
>
> 11机器：反向代理
>
> 12机器：反向代理
>
> 21机器：主控+运算节点（即服务群都是跑在21和22上）
>
> 22机器：主控+运算节点（生产上我们会把主控和运算分开）
>
> 200机器：运维主机（放各种文件资源）
>
> 这样控节点有两个，运算节点有两个，就是小型的分布式，现在你可能没办法理解这些内容，我们接着做下去，慢慢的，你就理解了
>
> 这是理论上的架构，考虑到公有云对网络的限制等问题，21、22机器也会追加反向代理。

### 实验机器安装详解

我们将申请5个节点，如下图

|      | 整体  | 11机器 | 12机器 | 21机器 | 22机器 | 200机器 |
| ---- | ----- | ------ | ------ | ------ | ------ | ------- |
| 低配 | 4C32G | 2C2G   | 2C2G   | 2C8G   | 2C8G   | 2C2G    |
| 标配 | 8C64G | 2C4G   | 2C4G   | 2C16G  | 2C16G  | 2C2G    |

> 低配能够满足Jenkins之前的内容，做到Jenkins的时候就已经很卡了，到时候再追加购买新的配置，当然也可以直接购买标配，前期我还是希望各位能在理解上花些时间，慢慢的操作，否则报错都不知道怎么解决

### 如何购买公有云机器（以阿里云为例）

> 注：未选的则是默认选项。

如下图，选择“自定义购买”、“抢占式实例”（这里使用抢占式即可/省钱，担心被回收的可以选择按量付费）、“新加坡”或其它外网服务器地域（外网服务器，方便后续使用网站等时不需要备案/最好准备个梯子，作为一个程序员梯子是必备的）、“可用区”（必须选择其中一个如我选的可用区A，不能随机分配，否则多台机器的内网ip不一致）

![image-20230510093039761](/Users/xueweiguo/Library/Application Support/typora-user-images/image-20230510093039761.png)

选择“共享型”（也可以选择通用型，共享型会更便宜）、选择2核8G内存的配置，

![image-20230510100214922](/Users/xueweiguo/Library/Application Support/typora-user-images/image-20230510100214922.png)

选择CentOS，7.6版本

![image-20230510100559866](/Users/xueweiguo/Library/Application Support/typora-user-images/image-20230510100559866.png)

网络分配公网IPv4地址，默认选择，其它不变，这样创建下来的服务器是公用同一安全组配置

选择“自定义密码“，建议设置相对复杂的密码，如包含大小写，否则容易被攻破使用

![image-20230510100759504](/Users/xueweiguo/Library/Application Support/typora-user-images/image-20230510100759504.png)

最终订单内容

![image-20230510100913556](/Users/xueweiguo/Library/Application Support/typora-user-images/image-20230510100913556.png)

购买完成后的内容，建议给每个实例设置节点含义，如下图的k8s_11、k8s_12

![image-20230510101304296](/Users/xueweiguo/Library/Application Support/typora-user-images/image-20230510101304296.png)

如下是我的内网及节点对应情况

> |        | 11机器         | 12机器         | 21机器         | 22机器         | 200机器        |
> | ------ | -------------- | -------------- | -------------- | -------------- | -------------- |
> | 内网ip | 172.29.238.166 | 172.29.238.164 | 172.29.238.165 | 172.29.238.163 | 172.29.238.162 |
>
> 可以看到网段在172.29.238.0/254

### 机器前期准备

使用xshell或者自选的shell连接工具，[xshell下载](https://xshell.en.softonic.com/download)

> 当然我也有提供软件包：https://pan.baidu.com/s/1mkIzua1XQmew240XBbvuFA 提取码：7p6h 。里面还有Xftp，是用来进行本地电脑和虚拟机的文件传输

~~~
# 全部机器，设置名字，11是hdss7-11,12是hdss7-12,以此类推
~]# hostnamectl set-hostname hdss7-11.host.com
~]# exit
# 下面的修改，11机器对应11机器的内网ip，12对应12机器的内网ip，以此类推共两处：IPADDR、GATEWAY
~]# vi /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE=eth0
BOOTPROTO=dhcp
ONBOOT=yes
IPADDR=172.29.238.166
GATEWAY=172.29.238.254
NETMASK=255.255.255.0
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=eth0
DEVICE=eth0
IPV6_PEERNDS=yes
IPV6_PEERROUTES=yes
IPV6_PRIUACY=no

~]# systemctl restart network
~]# ping baidu.com
~~~

> **ifcfg-eth0**：有些人的机器可能是ifcfg-esn33。使用ifconfig查看，命令输出中每个网卡接口都有一个名称，例如 "eth0"、"ens33" 等
>
> **systemctl restart**：重启某个服务
>
> **ping**：用于测试网络连接量的程序
>
> 公有云（阿里）的ifcfg-eth0原内容如下：
>
> DEVICE=eth0
> BOOTPROTO=dhcp
> ONBOOT=yes

![image-20230510102500398](/Users/xueweiguo/Library/Application Support/typora-user-images/image-20230510102500398.png)



~~~
# 查看enforce是否关闭，确保disabled状态，当然可能没有这个命令
~]# getforce
# 查看内核版本，确保在3.8以上版本
~]# uname -a
# 关闭firewalld
~]# systemctl stop firewalld
# 安装epel源及相关工具
~]# yum install epel-release -y
~]# yum install wget net-tools telnet tree nmap sysstat lrzsz dos2unix bind-utils -y
~~~

> **uname**:显示系统信息
>
> - **-a/-all**：显示全部
>
> **yum**：提供了查找、安装、删除某一个、一组甚至全部软件包的命令
>
> - **install**：安装
> - **-y**：当安装过程提示选择全部为"yes"

![image-20230510104702344](/Users/xueweiguo/Library/Application Support/typora-user-images/image-20230510104702344.png)



