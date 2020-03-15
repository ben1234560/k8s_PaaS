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
    <li><a herf="https://github.com/ben1234560/k8s_PaaS/blob/master/%E7%AC%AC%E4%B8%80%E7%AB%A0%E2%80%94%E2%80%94Docker.md">第一章——Docker</a>
    <ul>
        <li><a herf="https://github.com/ben1234560/k8s_PaaS/blob/master/%E7%AC%AC%E4%B8%80%E7%AB%A0%E2%80%94%E2%80%94Docker.md#%E5%AE%89%E8%A3%85docker">安装Docker</a>
      <li><a herf="https://github.com/ben1234560/k8s_PaaS/blob/master/%E7%AC%AC%E4%B8%80%E7%AB%A0%E2%80%94%E2%80%94Docker.md#%E5%BC%80%E5%90%AF%E6%88%91%E4%BB%AC%E7%9A%84%E7%AC%AC%E4%B8%80%E4%B8%AAdocker%E5%AE%B9%E5%99%A8">开启我们的第一个docker容器</a>
      <li><a herf="https://github.com/ben1234560/k8s_PaaS/blob/master/%E7%AC%AC%E4%B8%80%E7%AB%A0%E2%80%94%E2%80%94Docker.md#dockerhub%E6%B3%A8%E5%86%8C%E8%87%AA%E5%B7%B1%E7%9A%84%E8%BF%9C%E7%A8%8B%E4%BB%93%E5%BA%93">Dockerhub注册（自己的远程仓库）</a>
      <li><a herf="https://github.com/ben1234560/k8s_PaaS/blob/master/%E7%AC%AC%E4%B8%80%E7%AB%A0%E2%80%94%E2%80%94Docker.md#docker%E9%95%9C%E5%83%8F%E7%AE%A1%E7%90%86%E5%AE%9E%E6%88%98">Docker镜像管理实战</a>
      <li><a herf="https://github.com/ben1234560/k8s_PaaS/blob/master/%E7%AC%AC%E4%B8%80%E7%AB%A0%E2%80%94%E2%80%94Docker.md#docker%E5%AE%B9%E5%99%A8%E5%9F%BA%E6%9C%AC%E6%93%8D%E4%BD%9C">docker容器操作</a>
      <li><a herf="https://github.com/ben1234560/k8s_PaaS/blob/master/%E7%AC%AC%E4%B8%80%E7%AB%A0%E2%80%94%E2%80%94Docker.md#dockerfile-%E7%BB%BC%E5%90%88%E5%AE%9E%E9%AA%8C">dockerfile 综合实验</a>
    </ul>
  </li>
    <li><a herf="https://github.com/ben1234560/k8s_PaaS/blob/master/%E7%AC%AC%E4%BA%8C%E7%AB%A0%E2%80%94%E2%80%94%E4%BC%81%E4%B8%9A%E9%83%A8%E7%BD%B2%E5%AE%9E%E6%88%98_K8S.md">第二章——企业部署实战_K8S</a>
    <ul>
      <li><a herf="https://github.com/ben1234560/k8s_PaaS/blob/master/%E7%AC%AC%E4%BA%8C%E7%AB%A0%E2%80%94%E2%80%94%E4%BC%81%E4%B8%9A%E9%83%A8%E7%BD%B2%E5%AE%9E%E6%88%98_K8S.md#%E6%88%91%E4%BB%AC%E9%83%A8%E7%BD%B2%E7%9A%84%E6%9E%B6%E6%9E%84%E5%9B%BE%E6%88%91%E4%BB%AC%E9%83%A8%E7%BD%B2%E7%9A%84%E6%98%AF%E4%B8%80%E5%A5%97%E5%AE%8C%E6%95%B4%E7%9A%84paas%E6%9C%8D%E5%8A%A1">K8S前置准备工作——bind9安装部署（DNS服务）</a>
      <li><a herf="">K8S前置工作——准备签发证书环境</a>
      <li><a herf="https://github.com/ben1234560/k8s_PaaS/blob/master/%E7%AC%AC%E4%BA%8C%E7%AB%A0%E2%80%94%E2%80%94%E4%BC%81%E4%B8%9A%E9%83%A8%E7%BD%B2%E5%AE%9E%E6%88%98_K8S.md#k8s%E5%89%8D%E7%BD%AE%E5%87%86%E5%A4%87%E5%B7%A5%E4%BD%9Cbind9%E5%AE%89%E8%A3%85%E9%83%A8%E7%BD%B2dns%E6%9C%8D%E5%8A%A1">K8S前置工作——部署docker环境</a>
      <li><a herf="https://github.com/ben1234560/k8s_PaaS/blob/master/%E7%AC%AC%E4%BA%8C%E7%AB%A0%E2%80%94%E2%80%94%E4%BC%81%E4%B8%9A%E9%83%A8%E7%BD%B2%E5%AE%9E%E6%88%98_K8S.md#k8s%E5%89%8D%E7%BD%AE%E5%B7%A5%E4%BD%9C%E9%83%A8%E7%BD%B2harbor%E4%BB%93%E5%BA%93">K8S前置工作——部署harbor仓库</a>
      <li><a herf="https://github.com/ben1234560/k8s_PaaS/blob/master/%E7%AC%AC%E4%BA%8C%E7%AB%A0%E2%80%94%E2%80%94%E4%BC%81%E4%B8%9A%E9%83%A8%E7%BD%B2%E5%AE%9E%E6%88%98_K8S.md#%E5%AE%89%E8%A3%85%E9%83%A8%E7%BD%B2%E4%B8%BB%E6%8E%A7%E8%8A%82%E7%82%B9%E6%9C%8D%E5%8A%A1etcd">安装部署主控节点服务etcd</a>
      <li><a herf="https://github.com/ben1234560/k8s_PaaS/blob/master/%E7%AC%AC%E4%BA%8C%E7%AB%A0%E2%80%94%E2%80%94%E4%BC%81%E4%B8%9A%E9%83%A8%E7%BD%B2%E5%AE%9E%E6%88%98_K8S.md#%E9%83%A8%E7%BD%B2api-server%E9%9B%86%E7%BE%A4">部署API-server集群</a>
      <li><a herf="https://github.com/ben1234560/k8s_PaaS/blob/master/%E7%AC%AC%E4%BA%8C%E7%AB%A0%E2%80%94%E2%80%94%E4%BC%81%E4%B8%9A%E9%83%A8%E7%BD%B2%E5%AE%9E%E6%88%98_K8S.md#%E5%AE%89%E8%A3%85%E9%83%A8%E7%BD%B2%E4%B8%BB%E6%8E%A7%E8%8A%82%E7%82%B9l4%E5%8F%8D%E4%BB%A3%E6%9C%8D%E5%8A%A1">安装部署主控节点L4反代服务</a>
      <li><a herf="https://github.com/ben1234560/k8s_PaaS/blob/master/%E7%AC%AC%E4%BA%8C%E7%AB%A0%E2%80%94%E2%80%94%E4%BC%81%E4%B8%9A%E9%83%A8%E7%BD%B2%E5%AE%9E%E6%88%98_K8S.md#%E5%AE%89%E8%A3%85%E9%83%A8%E7%BD%B2controller-managerv%E8%8A%82%E7%82%B9%E6%8E%A7%E5%88%B6%E5%99%A8%E8%B0%83%E5%BA%A6%E5%99%A8%E6%9C%8D%E5%8A%A1">安装部署controller-managerv</a>
      <li><a herf="https://github.com/ben1234560/k8s_PaaS/blob/master/%E7%AC%AC%E4%BA%8C%E7%AB%A0%E2%80%94%E2%80%94%E4%BC%81%E4%B8%9A%E9%83%A8%E7%BD%B2%E5%AE%9E%E6%88%98_K8S.md#%E5%AE%89%E8%A3%85%E9%83%A8%E7%BD%B2%E8%BF%90%E7%AE%97%E8%8A%82%E7%82%B9%E6%9C%8D%E5%8A%A1kubelet">安装部署运算节点服务</a>
    </ul>
  </li>
    <li><a herf="https://github.com/ben1234560/k8s_PaaS/blob/master/%E7%AC%AC%E4%B8%89%E7%AB%A0%E2%80%94%E2%80%94k8s%E9%9B%86%E7%BE%A4.md">第三章——k8s集群</a>
    <ul>
        <li><a herf="https://github.com/ben1234560/k8s_PaaS/blob/master/%E7%AC%AC%E4%B8%89%E7%AB%A0%E2%80%94%E2%80%94k8s%E9%9B%86%E7%BE%A4.md#%E5%AE%89%E8%A3%85%E9%83%A8%E7%BD%B2flanneld">安装部署flanneld</a>
      <li><a herf="https://github.com/ben1234560/k8s_PaaS/blob/master/%E7%AC%AC%E4%B8%89%E7%AB%A0%E2%80%94%E2%80%94k8s%E9%9B%86%E7%BE%A4.md#flannel%E4%B9%8Bsnat%E8%A7%84%E5%88%99%E4%BC%98%E5%8C%96">flannel之SNAT规则优化</a>
      <li><a herf="https://github.com/ben1234560/k8s_PaaS/blob/master/%E7%AC%AC%E4%B8%89%E7%AB%A0%E2%80%94%E2%80%94k8s%E9%9B%86%E7%BE%A4.md#%E5%AE%89%E8%A3%85%E9%83%A8%E7%BD%B2coredns%E6%9C%8D%E5%8A%A1%E5%8F%91%E7%8E%B0">安装部署coredns（服务发现）</a>
      <li><a herf="https://github.com/ben1234560/k8s_PaaS/blob/master/%E7%AC%AC%E4%B8%89%E7%AB%A0%E2%80%94%E2%80%94k8s%E9%9B%86%E7%BE%A4.md#k8s%E7%9A%84%E6%9C%8D%E5%8A%A1%E6%9A%B4%E9%9C%B2ingress">K8S的服务暴露ingress</a>
    </ul>
  </li>
  <li><a herf="https://github.com/ben1234560/k8s_PaaS/blob/master/%E7%AC%AC%E5%9B%9B%E7%AB%A0%E2%80%94%E2%80%94dashboard%E6%8F%92%E4%BB%B6%E5%8F%8Ak8s%E5%AE%9E%E6%88%98%E4%BA%A4%E4%BB%98.md">第四章——dashboard插件及k8s实战交付</a>
    <ul>
        <li><a herf="https://github.com/ben1234560/k8s_PaaS/blob/master/%E7%AC%AC%E5%9B%9B%E7%AB%A0%E2%80%94%E2%80%94dashboard%E6%8F%92%E4%BB%B6%E5%8F%8Ak8s%E5%AE%9E%E6%88%98%E4%BA%A4%E4%BB%98.md#dashboard%E5%AE%89%E8%A3%85%E9%83%A8%E7%BD%B2">dashboard安装部署</a>
      <li><a herf="https://github.com/ben1234560/k8s_PaaS/blob/master/%E7%AC%AC%E5%9B%9B%E7%AB%A0%E2%80%94%E2%80%94dashboard%E6%8F%92%E4%BB%B6%E5%8F%8Ak8s%E5%AE%9E%E6%88%98%E4%BA%A4%E4%BB%98.md#k8s%E4%BB%AA%E8%A1%A8%E7%9B%98%E9%89%B4%E6%9D%83">K8S仪表盘鉴权</a>
      <li><a herf="https://github.com/ben1234560/k8s_PaaS/blob/master/%E7%AC%AC%E5%9B%9B%E7%AB%A0%E2%80%94%E2%80%94dashboard%E6%8F%92%E4%BB%B6%E5%8F%8Ak8s%E5%AE%9E%E6%88%98%E4%BA%A4%E4%BB%98.md#dashboardheapster">dashboard——heapster</a>
      <li><a herf="https://github.com/ben1234560/k8s_PaaS/blob/master/%E7%AC%AC%E5%9B%9B%E7%AB%A0%E2%80%94%E2%80%94dashboard%E6%8F%92%E4%BB%B6%E5%8F%8Ak8s%E5%AE%9E%E6%88%98%E4%BA%A4%E4%BB%98.md#k8s%E5%B9%B3%E6%BB%91%E5%8D%87%E7%BA%A7%E6%8A%80%E5%B7%A7">K8S平滑升级技巧</a>
    </ul>
  </li>
  <li><a herf="https://github.com/ben1234560/k8s_PaaS/blob/master/%E7%AC%AC%E4%BA%94%E7%AB%A0%E2%80%94%E2%80%94K8S%E7%BB%93%E5%90%88CI%26CD%E6%8C%81%E7%BB%AD%E4%BA%A4%E4%BB%98%E5%92%8C%E9%9B%86%E4%B8%AD%E7%AE%A1%E7%90%86%E9%85%8D%E7%BD%AE.md">第五章——K8S结合CI&CD持续交付和集中管理配置</a>
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
  <li><a herf="https://github.com/ben1234560/k8s_PaaS/blob/master/%E7%AC%AC%E5%85%AD%E7%AB%A0%E2%80%94%E2%80%94%E5%9C%A8K8S%E4%B8%AD%E9%9B%86%E6%88%90Apollo%E9%85%8D%E7%BD%AE%E4%B8%AD%E5%BF%83.md">第六章——在K8S中集成Apollo配置中心</a>
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
  <li><a herf="https://github.com/ben1234560/k8s_PaaS/blob/master/%E7%AC%AC%E4%B8%83%E7%AB%A0%E2%80%94%E2%80%94Promtheus%E7%9B%91%E6%8E%A7k8s%E4%BC%81%E4%B8%9A%E5%AE%B6%E5%BA%94%E7%94%A8.md">第七章——Promtheus监控k8s企业家应用</a>
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
  <li><a herf="https://github.com/ben1234560/k8s_PaaS/blob/master/%E7%AC%AC%E5%85%AB%E7%AB%A0%E2%80%94%E2%80%94spinaker%E9%83%A8%E7%BD%B2%E4%B8%8E%E5%BA%94%E7%94%A8.md">第八章——spinaker部署与应用</a>
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


