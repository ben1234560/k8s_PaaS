## 第二章——企业部署实战_K8S【公有云版】

### 前言（推荐使用非公有云版，因为多次适配过后续内容）：

注：公有云版未测试是否适配第三章往后内容，理论上是完全可行的。

**考虑到部分同学机器配置并无法满足多开VMware，这里推出公有云版本（以阿里云为例）。必须有梯子（vpn/翻墙）**

> **WHY：**为什么要有梯子，后面我们会创建一些网站，而国内域云服务器创建的网站需要ICP备案，每次备案都需要等审批非常麻烦。而国外域云服务器创建的网站则不需要备案，用起来非常舒服。
>
> <img src="assets/WX20230511-145214@2x.png" alt="image-ICPFiling" style="zoom:25%;" />
>
> **HOW：**梯子推荐lestvpn(非广告，这个一个月要40多rmb，能选代理地址我因为也用chatGPT所以开了，有更便宜的一定要推荐我😭，mac/window都测过可以用)
>
> 下载链接（推荐使用浏览器访问）：https://bitbucket.org/letsgogo/letsgogo_19/src/master/README.md 
>
> 备用链接（推荐使用浏览器访问）：https://github.com/LetsGo666/LetsGo_4
>
> 安装后打开填写我的ID：118149732 你还能多得3天会员！（可以跟朋友互相推荐互相得免费天数）



##### 如果你是新手，机器的名字及各种账户密码一定要和我的一样，先学一遍，再自己改

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
> 这是理论上的架构，考虑到公有云对网络的限制等问题（如ipv6不能随意增加），21、22机器也会追加反向代理。

### 实验机器安装详解

我们将申请5个节点，如下图

|      | 整体  | 11机器 | 12机器 | 21机器 | 22机器 | 200机器 |
| ---- | ----- | ------ | ------ | ------ | ------ | ------- |
| 低配 | 4C32G | 2C2G   | 2C2G   | 2C8G   | 2C8G   | 2C2G    |
| 标配 | 8C64G | 2C4G   | 2C4G   | 2C16G  | 2C16G  | 2C2G    |

> 低配能够满足Jenkins之前的内容，做到Jenkins的时候就已经很卡了，到时候再追加购买新的配置，当然也可以直接购买标配，前期我还是希望各位能在理解上花些时间，慢慢的操作，否则报错都不知道怎么解决

### 如何购买公有云机器（以阿里云为例）

> 注：未选的则是默认选项。

如下图，选择“自定义购买”、“抢占式实例”（这里使用抢占式即可/省钱，担心被回收的可以选择按量付费）、“新加坡”或其它外网服务器地域（外网服务器，方便后续使用网站等时不需要备案/最好准备个梯子，作为一个程序员梯子是必备的）、“可用区”（必须选择其中一个如我选的可用区B，不能随机分配，否则多台机器的内网ip不一致）

![image-20230510093039761](assets/WX20230511-144757@2x.png)

> **WHY：**为什么选新加坡，因为我的梯子可以到新加坡

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
> 可以看到网段在172.29.238.0/254，我的11机器内网ip是172.29.238.166，后续将会持续用到，你需要填成你自己的11机器内网ip

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



### K8S前置准备工作——bind9安装部署（DNS服务）

> **WHAT**：DNS（域名系统）说白了，就是把一个域和IP地址做了一下绑定，如你在里机器里面输入 nslookup www.qq.com，出来的Address是一堆IP，IP是不容易记的，所以DNS让IP和域名做一下绑定，这样你输入域名就可以了
>
> **WHY**：我们要用ingress，在K8S里要做7层调度，而且无论如何都要用域名（如之前的那个百度页面的域名，那个是host的方式），但是问题是我们怎么给K8S里的容器绑host，所以我们必须做一个DNS，然后容器服从我们的DNS解析调度

~~~
# 在11机器：
~]# yum install bind -y
~]# rpm -qa bind
# out: bind-9.11.4-9.P2.el7.x86_64
# 配置主配置文件，11机器
~]# vi /etc/named.conf
listen-on port 53 { 172.29.238.166; };  # 原本是127.0.0.1，我的11内网ip是172.29.238.166，你填你的11内网ip
# listen-on-v6 port 53 { ::1; };  # 需要删掉
allow-query     { any; };  # 原本是locall
forwarders      { 172.29.238.254; };  #另外添加的
dnssec-enable no;  # 原本是yes
dnssec-validation no;  # 原本是yes

# 检查修改情况，没有报错即可（即没有信息）
~]# named-checkconf
~~~

> **rpm**：软件包管理器
>
> - **-qa**：查看已安装的所有软件包
>
> **rpm和yum安装的区别**：前者不检查相依性问题，后者检查（即相关依赖包）
>
> **named.conf文件内容解析：**
>
> - **listen-on**：监听端口，改为监听在内网，这样其它机器也可以用
> - **allow-query**：哪些客户端能通过自建的DNS查
> - **forwarders**：上级DNS是什么

![image-20230510105344849](/Users/xueweiguo/Library/Application Support/typora-user-images/image-20230510105344849.png)

~~~
# 11机器，经验：主机域一定得跟业务是一点关系都没有，如host.com，而业务用的是od.com，因为业务随时可能变
# 区域配置文件，加在最下面，需要修改两处：allow-update
~]# vi /etc/named.rfc1912.zones
zone "host.com" IN {
        type  master;
        file  "host.com.zone";
        allow-update { 172.29.238.166; };
};

zone "od.com" IN {
        type  master;
        file  "od.com.zone";
        allow-update { 172.29.238.166; };
};

~~~

<img src="/Users/xueweiguo/Library/Application Support/typora-user-images/image-20230510105858429.png" alt="image-20230510105858429" style="zoom: 50%;" />

> **注意：**当配置11机器内网ip后，该机器应保存运行状态，重启后其它机器可能无法连接外网。@https://github.com/xinzhuxiansheng感谢建议！

~~~
# 11机器：
# 注意serial行的时间，代表今天的时间+第一条记录：20230510+01，需要修改serial、各个ip、
7-11 ~]# vi /var/named/host.com.zone
$ORIGIN host.com.
$TTL 600	; 10 minutes
@       IN SOA	dns.host.com. dnsadmin.host.com. (
				2023051001 ; serial
				10800      ; refresh (3 hours)
				900        ; retry (15 minutes)
				604800     ; expire (1 week)
				86400      ; minimum (1 day)
				)
			NS   dns.host.com.
$TTL 60	; 1 minute
dns                A    172.29.238.166
HDSS7-11           A    172.29.238.166
HDSS7-12           A    172.29.238.164
HDSS7-21           A    172.29.238.165
HDSS7-22           A    172.29.238.163
HDSS7-200          A    172.29.238.162

7-11 ~]# vi /var/named/od.com.zone
$ORIGIN od.com.
$TTL 600	; 10 minutes
@   		IN SOA	dns.od.com. dnsadmin.od.com. (
				2023051001 ; serial
				10800      ; refresh (3 hours)
				900        ; retry (15 minutes)
				604800     ; expire (1 week)
				86400      ; minimum (1 day)
				)
				NS   dns.od.com.
$TTL 60	; 1 minute
dns                A    172.29.238.166

# 看一下有没有报错
7-11 ~]# named-checkconf
7-11 ~]# systemctl start named
7-11 ~]# netstat -luntp|grep 53
~~~

> **TTL 600**：指定IP包被路由器丢弃之前允许通过的最大网段数量
>
> - **10 minutes**：过期时间10分钟
>
> **SOA**：一个域权威记录的相关信息，后面有5组参数分别设定了该域相关部分
>
> - **dnsadmin.od.com.**  一个假的邮箱
> - **serial**：记录的时间
>
> **$ORIGIN**：即下列的域名自动补充od.com，如dns，外面看来是dns.od.com
>
> **netstat -luntp**：显示 tcp,udp 的端口和进程等相关情况

![image-20230510110314995](/Users/xueweiguo/Library/Application Support/typora-user-images/image-20230510110314995.png)

~~~
# 11机器，检查主机域是否解析
7-11 ~]# dig -t A hdss7-21.host.com @172.29.238.166 +short
# 配置服务器能使用这个配置，在最下面追加11机器的内网ip
7-11 ~]# vi /etc/sysconfig/network-scripts/ifcfg-eth0
DNS1=172.29.238.166
7-11 ~]# systemctl restart network
# 重启网络后，马上ping可能无法连接，需要等个十秒钟
7-11 ~]# ping www.baidu.com
7-11 ~]# ping hdss7-21.host.com
~~~

> **dig -t A**：指的是找DNS里标记为A的相关记录，而后面会带上相关的域，如上面的hdss7-21.host.com，为什么外面配了HDSS7-21后面还会自动接上.host.com就是因为$ORIGIN，后面则是对应的IP
>
> - **+short**：表示只返回IP

![image-20230510110840167](/Users/xueweiguo/Library/Application Support/typora-user-images/image-20230510110840167.png)

~~~
# 在非11机器上，最下面追加dns1=11机器内网ip
~]# vi /etc/sysconfig/network-scripts/ifcfg-eth0
DNS1=172.29.238.166

~]# systemctl restart network
# 试下网络是否正常
~]# ping baidu.com
# 其它机器尝试ping7-11机器或200机器等任意机器，这时候短域名是可以使用了
7-12 ~]# ping hdss7-11.host.com
7-12 ~]# ping hdss7-200.host.com
~~~

> 让其它机器的DNS全部改成11机器的好处是，全部的机器访问外网就只有通过11端口，更好控制

机器准备工作完成:tada:



### K8S前置工作——准备签发证书环境

> **WHAT**： 证书，可以用来审计也可以保障安全，k8S组件启动的时候，则需要有对应的证书，证书的详解你也可以在网上搜到，这里就不细细说明了
>
> **WHY**：当然是为了让我们的组件能正常运行

~~~
# cfssl方式做证书，需要三个软件（如果网络无法连通，则参考下面我已下载好的软件包），按照我们的架构图，我们部署在200机器:
200 ~]# wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64 -O /usr/bin/cfssl
200 ~]# wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64 -O /usr/bin/cfssl-json
200 ~]# wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64 -O /usr/bin/cfssl-certinfo
200 ~]# chmod +x /usr/bin/cfssl*
200 ~]# which cfssl
200 ~]# which cfssl-json
200 ~]# which cfssl-certinfo
~~~

> **wget**：从网络上自动下载文件的自由工具
>
> **chmod**：给对应的文件添加权限（[菜鸟教程](https://www.runoob.com/linux/linux-comm-chmod.html)）
>
> - **+x**：给当前用户增加可执行该文件的权限
>
> **which**：查看相应的东西在哪里

##### 报错：wget报错网络问题

~~~
# 针对下载cfssl网络无法连通的情况，百度云https://pan.baidu.com/s/1arE2LdtAbcR80gmIQtIELw 提取码：ouy1。找到名为”第二章涉及的软件包“中的cfssl的3个文件，上传到/root目录下
200 ~]# cp cfssl_linux-amd64 /usr/bin/cfssl
200 ~]# cp cfssljson_linux-amd64 /usr/bin/cfssl-json
200 ~]# cp cfssl-certinfo_linux-amd64 /usr/bin/cfssl-certinfo
200 ~]# chmod +x /usr/bin/cfssl*
200 ~]# which cfssl
200 ~]# which cfssl-json
200 ~]# which cfssl-certinfo
~~~

![image-20230510133419709](/Users/xueweiguo/Library/Application Support/typora-user-images/image-20230510133419709.png)



~~~
200 ~]# mkdir /opt/certs
200 ~]# cd /opt/certs
certs]# vi ca-csr.json
{
    "CN": "ben123123",
    "hosts": [
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "beijing",
            "L": "beijing",
            "O": "od",
            "OU": "ops"
        }
    ],
    "ca": {
        "expiry": "1752000h"
    }
}
certs]# cfssl gencert -initca ca-csr.json | cfssl-json -bare ca
certs]# cat ca.pem
# 如果你这时候显示段错误或者json错误，就是之前克隆虚拟机的时候200机器地址和别的机器地址重叠冲突了，需要重建200虚拟机
~~~

> **cd **：切换当前工作目录到 dirName
>
> - 语法：cd [dirName]
>
> **mkdir**：建立名称为 dirName 之子目录
>
> - 语法：mkdir [-p] dirName
> - **-p:** 确保目录存在，不存在则建一个，如mkdir empty/empty1/empty2
>
> **cfssl**：证书工具
>
> - gencert：生成的意思
>
> **ca-csr.json解析：**
>
> - CN：Common Name，浏览器使用该字段验证网址是否合法，一般写域名，非常重要
> - ST：State，省
> - L：Locality，地区
> - O：Organization Name，组织名称
> - OU：Organization Unit Name，组织单位名称
>
> <a href="https://github.com/ben1234560/k8s_PaaS/blob/master/%E5%8E%9F%E7%90%86%E5%8F%8A%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/Kubernetes%E5%9F%BA%E6%9C%AC%E6%A6%82%E5%BF%B5.md#pod%E4%B8%AD%E5%87%A0%E4%B8%AA%E9%87%8D%E8%A6%81%E5%AD%97%E6%AE%B5%E7%9A%84%E5%90%AB%E4%B9%89%E5%92%8C%E7%94%A8%E6%B3%95">Pod中几个重要字段的含义和用法</a>

![image-20230510133747012](/Users/xueweiguo/Library/Application Support/typora-user-images/image-20230510133747012.png)



### K8S前置工作——部署docker环境

> **WHAT**：是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的镜像中，然后发布到任何流行的 Linux或Windows 机器上，也可以实现虚拟化。
>
> **WHY**：Pod里面就是由数个docker容器组成，Pod是豌豆荚，docker容器是里面的豆子。

~~~
# 如我们架构图所示，运算节点是21/22机器（没有docker则无法运行pod），运维主机是200机器（没有docker则没办法下载docker存入私有仓库），所以在三台机器安装（21/22/200）
~]# curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
# 上面的下载可能网络有问题，需要多试几次，这些部署我已经不同机器试过很多次了，实在不行的使用下面提供的报错解决方法。
~]# mkdir -p /data/docker /etc/docker
# # 注意，172.7.21.1，这里得21是指在hdss7-21得机器，如果是22得机器，就是172.7.22.1，共一处需要改机器名："bip": "172.7.21.1/24"
~]# vi /etc/docker/daemon.json
{
  "data-root": "/data/docker",
  "storage-driver": "overlay2",
  "insecure-registries": ["registry.access.redhat.com","quay.io","harbor.od.com"],
  "registry-mirrors": ["https://q2gr04ke.mirror.aliyuncs.com"],
  "bip": "172.7.21.1/24",
  "exec-opts": ["native.cgroupdriver=systemd"],
  "live-restore": true
}

~]# systemctl start docker
~]# docker version
~]# docker ps -a
~~~

> **mkdir -p：**前面有讲到过（上一级目录没有则创建），而这次后面带了两个目录，意思是同时创建两个目录
>
> **daemon.json：**为什么配aliyuncs的环境，是因为默认是连接到外网的，速度比较慢，所以我们可以直接使用国内的阿里云镜像源，当然还有腾讯云等
>
> **docker version：**查看docker的版本
>
> **daemon.json解析：**重点说一下这个为什么10.4.7.21机器对应着172.7.21.1/24，这里可以看到10的21对应得是172的21，这样做的好处就是，当你的pod出现问题时，你可以马上定位到是在哪台机器出现的问题，是21还是22还是其它的，这点在生产上非常重要，有时候你的dashboard（后面会安装）宕掉了，你没办法只能去机器找，而这时候你又找不到的时候，你老板会拿你祭天的

##### 报错：curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun

报错信息：

http://mirrors.cloud.aliyuncs.com/centos/7/updates/x86_64/Packages/libxml2-2.9.1-6.el7_9.6.x86_64.rpm: [Errno 14] curl#6 - "Could not resolve host: mirrors.cloud.aliyuncs.com; Unknown error"
Trying other mirror.

Error downloading packages:
  libxml2-2.9.1-6.el7_9.6.x86_64: [Errno 256] No more mirrors to try.
  python-kitchen-1.1.1-5.el7.noarch: [Errno 256] No more mirrors to try.
  yum-utils-1.1.31-54.el7_8.noarch: [Errno 256] No more mirrors to try.
  python-chardet-2.2.1-3.el7.noarch: [Errno 256] No more mirrors to try.
  libxml2-python-2.9.1-6.el7_9.6.x86_64: [Errno 256] No more mirrors to try.

原因：yum源出现问题

解决方法：

~~~
mv /etc/yum.repos.d/epel.repo /etc/yum.repos.d/epel.repo.bak
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo  # 这一步必须成功，没成功的再执行一遍即可
yum update -y  # 这一步耗时较长
yum install epel-release -y
yum install -y yum-utils
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum install -y docker-ce
~~~

![image-20230510141020872](/Users/xueweiguo/Library/Application Support/typora-user-images/image-20230510141020872.png)



### K8S前置工作——部署harbor仓库

> **WHAT **：harbor仓库是可以部署到本地的私有仓库，也就是你可以把镜像推到这个仓库，然后需要用的时候再下载下来，这样的好处是：1、下载速度快，用到的时候能马上下载下来2、不用担心镜像改动或者下架等。
>
> **WHY**：因为我们的部署K8S涉及到很多镜像，制作相关包的时候如果网速问题会导致失败重来，而且我们在公司里也会建自己的仓库，所以必须按照harbor仓库

~~~
# 如架构图，我们安装在200机器：
200 ~]# mkdir /opt/src  && cd /opt/src
# 可以去这个地址下载，也可以直接用我用的软件包
https://github.com/goharbor/harbor/releases/tag/v1.8.3
200 src]# wget https://storage.googleapis.com/harbor-releases/release-1.8.0/harbor-offline-installer-v1.8.3.tgz
# 或者用较新的包（未测试后面的适配情况）wget https://github.com/goharbor/harbor/releases/download/v2.8.0/harbor-offline-installer-v2.8.0.tgz
7-200 src]# tar xf harbor-offline-installer-v1.8.3.tgz -C /opt/
~~~

> **tag**：可以加入，解开备份文件内的文件
>
> - **x**：解压
> - **f**： 使用档案名字
> - **-C**：切换到指定的目录
> - 整条命令合起来就是，把tgz文件以tgz文件名为名字解压到opt目录下，并保存tgz文件原样
>
> **关于版本**：一般人都是喜欢用比较新的版本，我们当然也支持比较新的版本，但对于公司而已，稳定是最要紧的，v1.8.3是用的比较稳定的版本，而后续的各个组件也会有更加新的版本，你可以尝试新版本，但有些由于兼容问题不能用新的，后续那些不能用我会标记清楚。

~~~
# 200机器：
200 src]# cd /opt/
200 opt]# mv harbor/ harbor-v1.8.3
200 opt]# ln -s /opt/harbor-v1.8.3/ /opt/harbor
200 opt]# cd harbor
200 harbor]# ll
200 harbor]# vi harbor.yml  # 如果这里是用的2.8.0版本，则需要cp harbor.yml.tmpl harbor.yml
hostname: harbor.od.com  # 原reg.mydomain.com
http:
  port: 180  # 原80
data_volume: /data/harbor
location: /data/harbor/logs

200 harbor]# mkdir -p /data/harbor/logs
200 harbor]# yum install docker-compose -y
200 harbor]# rpm -qa docker-compose
# out: docker-compose-1.18.0-4.el7.noarch
# yum install docker-compose 包无法下载的，可以查看下面提供的报错解决方法
200 harbor]# ./install.sh
~~~

> **提示**：harbor v2.3.3版本安装也需要将https相关的配置注释掉
>
> 如图#https:
>
> ![1635775214057](/Users/xueweiguo/Desktop/GitHub/k8s_PaaS/assets/1635775214057.png)
>
> 感谢@https://github.com/xinzhuxiansheng
>
> **mv**：为文件或目录改名、或将文件或目录移入其它位置。
>
> - 这里的命令是有斜杠的，所以是移动到某个目录下
>
> **ln**：为某一个文件在另外一个位置建立一个同步的链接
>
> - `语法:ln [参数][源文件或目录][目标文件或目录]`
> - `**-s：**软连接，可以对整个目录进行链接`
>
> **harbor.yml解析：**
>
> - port为什么改成180：因为后面我们要装nginx，nginx用的80，所以要把它们错开
> - data_volume：数据卷，即docker镜像放在哪里
> - location：日志文件
> - **./install.sh**：启动shell脚本

##### 报错：yum install docker-compose -y报错No package docker-compose available.

使用我提供的官方软件包（放在百度网盘），放到harbor路径下

~~~
200 harbor]# sudo cp docker-compose-Linux-x86_64 /usr/local/bin/docker-compose
200 harbor]# sudo chmod +x /usr/local/bin/docker-compose
200 harbor]# docker-compose --version
# out: docker-compose version 1.29.2, build 5becea4c
200 harbor]# ./install.sh
~~~

![image-20230510143719007](/Users/xueweiguo/Library/Application Support/typora-user-images/image-20230510143719007.png)

![image-20230510143950978](/Users/xueweiguo/Library/Application Support/typora-user-images/image-20230510143950978.png)

> 这张图是2.8.0版本的，1.8.3版本是测试过能正常适配的

~~~
# 200机器：
200 harbor]# docker-compose ps
200 harbor]# docker ps -a
200 harbor]# yum install nginx -y  # 报错的参考下面报错对应的解决方法

###相关报错问题：
yum的时候报：/var/run/yum.pid 已被锁定，PID 为 1610 的另一个程序正在运行。
另外一个程序锁定了 yum；等待它退出……
网上统一的解决办法：直接在终端运行 rm -f /var/run/yum.pid 将该文件删除，然后再次运行yum。
###
~~~

##### 报错：yum install nginx -y没有对应包

~~~
200 ~]# yum install nginx -y
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * extras: mirrors.aliyun.com
 * updates: mirrors.aliyun.com
No package nginx available.
Error: Nothing to do
~~~

解决方法

~~~
# 编写nginx.repo
200 harbor]# sudo vi /etc/yum.repos.d/nginx.repo
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/7/$basearch/
gpgcheck=0
enabled=1

# 保存退出后即可安装nginx了
200 harbor]# sudo yum install nginx -y
~~~



解决完接着走

~~~
200 harbor]# vi /etc/nginx/conf.d/harbor.od.com.conf
server {
    listen       80;
    server_name  harbor.od.com;

    client_max_body_size 1000m;

    location / {
        proxy_pass http://127.0.0.1:180;
    }
}

200 harbor]# nginx -t
200 harbor]# systemctl start nginx
200 harbor]# systemctl enable nginx
~~~

> **nginx -t**：测试*nginx*.conf配置文件中是否存在语法错误
>
> **systemctl enable nginx**：开机自动启动

追加域名解析

~~~
# 在11机器解析域名：
~]# vi /var/named/od.com.zone
# 注意serial前滚一个序号
# 最下面添加域名
harbor             A    172.29.238.162
~]# systemctl restart named
~]# dig -t A harbor.od.com +short
# out:172.29.238.162
~~~

![image-20230510171927368](/Users/xueweiguo/Library/Application Support/typora-user-images/image-20230510171927368.png)

~~~
# 200机器上curl：
harbor]# curl harbor.od.com
~~~

> 注意：
>
> getenforce得是关闭状态，而不是enforcing，否则会报502
> 暂时关闭setenforce 0
> 永久关闭，改配置文件 vi /etc/selinux/config
> SELINUX=disabled

![image-20230510171956492](/Users/xueweiguo/Library/Application Support/typora-user-images/image-20230510171956492.png)

公有云的安全组添加tcp 180端口权限![image-20230508143611983](/Users/xueweiguo/Library/Application Support/typora-user-images/image-20230508143611983.png)

在本机（window/mac）hosts，在文件末尾添加新的 DNS 域名解析，如：

~~~
公网ip harbo.od.com  # 公网ip对应200机器的公网ip
~~~

[访问harbo.od.com](harbor.od.com)

![1578831134362](/Users/xueweiguo/Desktop/GitHub/k8s_PaaS/assets/1578831134362.png)

~~~
账号：admin
密码：Harbor12345
新建一个public公开项目
~~~

![1578831458247](/Users/xueweiguo/Desktop/GitHub/k8s_PaaS/assets/1578831458247.png)

~~~
# 200机器，尝试下是否能push成功到harbor仓库
harbor]# docker pull nginx:1.7.9
harbor]# docker images|grep 1.7.9
harbor]# docker tag 84581e99d807 harbor.od.com/public/nginx:v1.7.9
harbor]# docker login harbor.od.com
账号：admin
密码：Harbor12345
harbor]# docker push harbor.od.com/public/nginx:v1.7.9
# 报错：如果发现登录不上去了，过一阵子再登录即可，大约5分钟左右
~~~

![1578832524891](/Users/xueweiguo/Desktop/GitHub/k8s_PaaS/assets/1578832524891.png)

成功，此时你已成功建立了自己的本地私有仓库

