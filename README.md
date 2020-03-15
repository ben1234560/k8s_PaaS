# k8s_PaaS
[![image](https://img.shields.io/badge/google-kubernetes-blue.svg)](https://kubernetes.io/) [![image](https://img.shields.io/badge/ctripcorp-apollo-gray.svg)](https://github.com/ctripcorp/apollo) [![image](https://img.shields.io/badge/CNCD-Spinnaker-skyblue.svg)](https://www.spinnaker.io/) [![image](https://img.shields.io/badge/JAVA-Jenkins-orange.svg)](https://jenkins.io/zh/) [![image](https://img.shields.io/badge/Git-Gitee-red.svg)](https://gitee.com) [![image](https://img.shields.io/badge/Git-GitLab-orange.svg)]() [![image](https://img.shields.io/badge/Apache-zookeeper-Crimson.svg)](http://zookeeper.apache.org/) [![image](https://img.shields.io/badge/used-Harbor-green.svg)](https://goharbor.io/)

[![image](https://img.shields.io/badge/used-docker-blue.svg)](https://www.docker.com/) [![image](https://img.shields.io/badge/used-Prometheus-red.svg)](https://prometheus.io/) [![image](https://img.shields.io/badge/used-etcd-blue.svg)](https://etcd.io/) [![image](https://img.shields.io/badge/used-Grafana-orange.svg)](https://grafana.com)

如何基于K8S部署成PaaS（一套完整的软件研发和部署平台）——教程（实战代码/欢迎讨论/大量注释/操作配图），你将习得部署如：K8S、dashboard、Harbor、Jenkins、本地gitlab、Apollo框架、promtheus、grafana、spinnaker。

注释及配图覆盖率达80%以上，旨在帮助快速入门。

并将告诉你：是什么（WHAT）、为什么这么做(WHY)、怎么做(HOW)。

建议学习时长1个月+。

## PaaS架构图

![K8S_PaaS架构图](assets/K8S_PaaS架构图.png)

> 橙色框内软件皆部署在K8S集群中

## <a href="https://github.com/ben1234560/k8s_PaaS/blob/master/Features.md">Features</a>

- 对做的事情进行说明是什么（WHAT），为什么要做（WHY）
- 对相关文件进行解析，并配图避免学习出错
- 指明在哪部机器操作，及容易报错点添加解决办法
- 匹配对应文件，避免无法被下架无法下载等情况

## 学习章节：

<ul>
  <li>第一章——Docker
    <ul>
      <li>安装Docker
      <li>开启我们的第一个docker容器
      <li>Dockerhub注册（自己的远程仓库）
      <li>Docker镜像管理实战
      <li>docker容器高级操作
      <li>dockerfile 综合实验
    </ul>
  </li>
  <li>第二章——企业部署实战_K8S
    <ul>
      <li>K8S前置准备工作——bind9安装部署（DNS服务）
      <li>K8S前置工作——部署docker环境
      <li>Dockerhub注册（自己的远程仓库）
      <li>K8S前置工作——部署harbor仓库
      <li>安装部署主控节点服务etcd
      <li>部署API-server集群
      <li>安装部署主控节点L4反代服务
      <li>安装部署controller-managerv
      <li>安装部署运算节点服务
    </ul>
  </li>
  <li>第三章——k8s集群
    <ul>
      <li>安装部署flanneld
      <li>flannel之SNAT规则优化
      <li>安装部署coredns（服务发现）
      <li>K8S的服务暴露ingress
    </ul>
  </li>
  <li>第四章——dashboard插件及k8s实战交付
    <ul>
      <li>dashboard安装部署
      <li>K8S仪表盘鉴权
      <li>dashboard——heapster
      <li>K8S平滑升级技巧
    </ul>
  </li>
  <li>第五章——K8S结合CI&CD持续交付和集中管理配置
    <ul>
      <li>安装部署zookeeper
      <li>安装部署Jenkins
      <li>安装maven
      <li>制作dubbo微服务的底包镜像
      <li>安装maven
      <li>使用Jenkins持续构建交付dubbo服务的提供者
      <li>借助BlueOcean插件回顾Jenkins流水线构建原理
      <li>交付dubbo-monitor到k8s集群
      <li>实现dubbo集群的日常维护
      <li>实战K8S集群毁灭性测试
    </ul>
  </li>
  <li>第六章——在K8S中集成Apollo配置中心
    <ul>
      <li>configmap使用详解
      <li>交付Apollo-ConfigService到K8S
      <li>Apollo-ConfigService连接数据库IP分析
      <li>交付Apollo-Portal前，数据库初始化
      <li>制作Portal的docker镜像，并交付
      <li>dubbo服务提供者连接Apollo实战
      <li>dubbo服务消费者连接Apollo实战
      <li>实战Apollo分环境管理dubbo服务-交付Apollo-configservice
      <li>实战使用Apollo分环境管理dubbo服务——交付Apollo-portal和adminservice
      <li>实战发布dubbo连接Apollo到不同环境
      <li>实战演示项目提测，发版流程
    </ul>
  </li>
  <li>第七章——Promtheus监控k8s企业家应用
    <ul>
      <li>Prometheus监控软件概述
      <li>交付kube-state-metric
      <li>交付node-exporter
      <li>交付cadvisor
      <li>交付blackbox-exporter
      <li>安装部署Prometheus-server
      <li>配置Prometheus监控业务容器
      <li>安装部署配置Grafana
      <li>安装部署alertmanager
      <li>测试alertmanager报警功能
      <li>通过K8S部署dubbo微服务接入ELK架构
      <li>制作tomcat容器的底包镜像
      <li>交付tomcat形式的dubbo服务消费者到K8S集群
      <li>二进制安装部署elasticsearch
      <li>安装部署kafka和kafka-manager
      <li>制作filebeat底包并接入dubbo服务消费者
      <li>部署logstash镜像
      <li>交付kibana到K8S集群
      <li>详解Kibana生产实践方法
    </ul>
  </li>
  <li>第八章——spinaker部署与应用
    <ul>
      <li>部署Spinnaker的Amory发行版
      <li>安装部署redis
      <li>安装部署clouddriver
      <li>安装部署spinnaker其余组件
      <li>使用spinnaker结合Jenkins构建镜像
      <li>使用spinnaker配置dubbo服务提供者发布至K8S
      <li>使用spinnaker配置dubbo服务消费者到K8S
      <li>模拟生产上代码迭代
    </ul>
  </li>
</ul>

## 说明
<p> 本专题并不用于商业用途，转载请注明本专题地址，如有侵权，请务必邮件通知作者。
<p> 本人水平有限，代码搬到外部环境难免有遗漏错误的地方，望不吝赐教，万分感谢。
<p> 有代码疑惑的地方也请找我。
<p> Email：909336740@qq.com
<p> QQ：909336740


