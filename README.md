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
- 匹配对应文件，避免无法被下架无法下载等情况，百度云https://pan.baidu.com/s/1arE2LdtAbcR80gmIQtIELw 提取码：ouy1

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
</ul>



## 说明
<p> 本专题并不用于商业用途，转载请注明本专题地址，如有侵权，请务必邮件通知作者。
<p> 本人水平有限，代码搬到外部环境难免有遗漏错误的地方，望不吝赐教，万分感谢。
<p> 有代码疑惑或者缺少什么文件请找我。
<p> Email：909336740@qq.com
<p> QQ：909336740



