## 原理及源码解析

该部分为帮助学者更好的理解和使用



## 学习章节：

<ul>
    <li><a href="https://github.com/ben1234560/k8s_PaaS/blob/master/%E5%8E%9F%E7%90%86%E5%8F%8A%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/Docker%E5%9F%BA%E7%A1%80.md">第一章——Docker</a>
        <ul>
            <li><a href="https://github.com/ben1234560/k8s_PaaS/blob/master/%E5%8E%9F%E7%90%86%E5%8F%8A%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/Docker%E5%9F%BA%E7%A1%80.md#%E5%AE%B9%E5%99%A8%E6%98%AF%E6%80%8E%E4%B9%88%E9%9A%94%E7%A6%BB%E7%9A%84">Docker容器是怎么隔离的</a></li>
            <li><a href="https://github.com/ben1234560/k8s_PaaS/blob/master/%E5%8E%9F%E7%90%86%E5%8F%8A%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/Docker%E5%9F%BA%E7%A1%80.md#%E5%85%B3%E4%BA%8Enamespace">关于namespace</a></li>
            <li><a href="https://github.com/ben1234560/k8s_PaaS/blob/master/%E5%8E%9F%E7%90%86%E5%8F%8A%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/Docker%E5%9F%BA%E7%A1%80.md#%E8%99%9A%E6%8B%9F%E6%9C%BA%E5%92%8C%E5%AE%B9%E5%99%A8%E7%9A%84%E5%AF%B9%E6%AF%94%E5%9B%BE">虚拟机和容器的对比图</a></li>
            <li><a href="https://github.com/ben1234560/k8s_PaaS/blob/master/%E5%8E%9F%E7%90%86%E5%8F%8A%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/Docker%E5%9F%BA%E7%A1%80.md#%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3%E5%AE%B9%E5%99%A8%E9%95%9C%E5%83%8F">深入理解容器镜像</a></li>
        </ul>
    </li>
    <li><a href="https://github.com/ben1234560/k8s_PaaS/blob/master/%E5%8E%9F%E7%90%86%E5%8F%8A%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/Kubernetes%E5%9F%BA%E6%9C%AC%E6%A6%82%E5%BF%B5.md">Kubernetes基本概念.md</a>
        <ul>
            <li><a href="https://github.com/ben1234560/k8s_PaaS/blob/master/%E5%8E%9F%E7%90%86%E5%8F%8A%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/Kubernetes%E5%9F%BA%E6%9C%AC%E6%A6%82%E5%BF%B5.md#%E5%88%9D%E8%AF%86pod">初识Pod</a></li>
            <li><a href="https://github.com/ben1234560/k8s_PaaS/blob/master/%E5%8E%9F%E7%90%86%E5%8F%8A%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/Kubernetes%E5%9F%BA%E6%9C%AC%E6%A6%82%E5%BF%B5.md#pod%E4%B8%AD%E5%87%A0%E4%B8%AA%E9%87%8D%E8%A6%81%E5%AD%97%E6%AE%B5%E7%9A%84%E5%90%AB%E4%B9%89%E5%92%8C%E7%94%A8%E6%B3%95">Pod中几个重要字段的含义和用法</a></li>
            <li><a href="https://github.com/ben1234560/k8s_PaaS/blob/master/%E5%8E%9F%E7%90%86%E5%8F%8A%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/Kubernetes%E5%9F%BA%E6%9C%AC%E6%A6%82%E5%BF%B5.md#pod%E7%9A%84%E5%87%A0%E7%A7%8D%E7%8A%B6%E6%80%81">Pod的几种状态</a></li>
            <li><a href="https://github.com/ben1234560/k8s_PaaS/blob/master/%E5%8E%9F%E7%90%86%E5%8F%8A%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/Kubernetes%E5%9F%BA%E6%9C%AC%E6%A6%82%E5%BF%B5.md#%E6%B0%B4%E5%B9%B3%E6%89%A9%E5%B1%95%E5%92%8C%E6%BB%9A%E5%8A%A8%E5%8D%87%E7%BA%A7">水平扩展和滚动升级</a></li>
            <li><a href="https://github.com/ben1234560/k8s_PaaS/blob/master/%E5%8E%9F%E7%90%86%E5%8F%8A%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/Kubernetes%E5%9F%BA%E6%9C%AC%E6%A6%82%E5%BF%B5.md#rbac%E5%9F%BA%E4%BA%8E%E8%A7%92%E8%89%B2%E7%9A%84%E6%9D%83%E9%99%90%E6%8E%A7%E5%88%B6">RBAC:基于角色的权限控制</a></li>
            <li><a href="https://github.com/ben1234560/k8s_PaaS/blob/master/%E5%8E%9F%E7%90%86%E5%8F%8A%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/Kubernetes%E5%9F%BA%E6%9C%AC%E6%A6%82%E5%BF%B5.md#operator-%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86">Operator 工作原理</a></li>
            <li><a href="https://github.com/ben1234560/k8s_PaaS/blob/master/%E5%8E%9F%E7%90%86%E5%8F%8A%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/Kubernetes%E5%9F%BA%E6%9C%AC%E6%A6%82%E5%BF%B5.md#kubernetes%E6%8A%80%E8%83%BD%E5%9B%BE%E8%B0%B1">kubernetes技能图谱</a></li>
        </ul>
    </li>
    <li><a href="https://github.com/ben1234560/k8s_PaaS/blob/master/%E5%8E%9F%E7%90%86%E5%8F%8A%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/Kubernetes%E8%B0%83%E5%BA%A6%E6%9C%BA%E5%88%B6.md">Kubernetes调度机制</a>
        <ul>
            <li><a href="https://github.com/ben1234560/k8s_PaaS/blob/master/%E5%8E%9F%E7%90%86%E5%8F%8A%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/Kubernetes%E8%B0%83%E5%BA%A6%E6%9C%BA%E5%88%B6.md#kubernetes%E7%9A%84%E8%B5%84%E6%BA%90%E6%A8%A1%E5%9E%8B%E4%B8%8E%E8%B5%84%E6%BA%90%E7%AE%A1%E7%90%86">Kubernetes的资源模型与资源管理</a></li>
            <li><a href="https://github.com/ben1234560/k8s_PaaS/blob/master/%E5%8E%9F%E7%90%86%E5%8F%8A%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/Kubernetes%E8%B0%83%E5%BA%A6%E6%9C%BA%E5%88%B6.md#kubernetes%E9%BB%98%E8%AE%A4%E7%9A%84%E8%B0%83%E5%BA%A6%E7%AD%96%E7%95%A5">Kubernetes默认的调度策略</a></li>
            <li><a href="https://github.com/ben1234560/k8s_PaaS/blob/master/%E5%8E%9F%E7%90%86%E5%8F%8A%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/Kubernetes%E8%B0%83%E5%BA%A6%E6%9C%BA%E5%88%B6.md#%E8%B0%83%E5%BA%A6%E5%99%A8%E7%9A%84%E4%BC%98%E5%85%88%E7%BA%A7%E4%B8%8E%E5%BC%BA%E5%88%B6%E6%9C%BA%E5%88%B6">调度器的优先级与强制机制</a></li>
        </ul>
    </li>
    <li><a href="https://github.com/ben1234560/k8s_PaaS/blob/master/%E5%8E%9F%E7%90%86%E5%8F%8A%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/Kubernetes%E7%9B%B8%E5%85%B3%E7%94%9F%E6%80%81.md">Kubernetes相关生态.md</a>
        <ul>
            <li><a href="https://github.com/ben1234560/k8s_PaaS/blob/master/%E5%8E%9F%E7%90%86%E5%8F%8A%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/Kubernetes%E7%9B%B8%E5%85%B3%E7%94%9F%E6%80%81.md#prometheusmetrics-server%E4%B8%8Ekubernetes%E7%9B%91%E6%8E%A7%E4%BD%93%E7%B3%BB">Prometheus、Metrics Server与Kubernetes监控体系</a></li>
            <li><a href="https://github.com/ben1234560/k8s_PaaS/blob/master/%E5%8E%9F%E7%90%86%E5%8F%8A%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/Kubernetes%E7%9B%B8%E5%85%B3%E7%94%9F%E6%80%81.md#%E6%97%A5%E5%BF%97%E6%94%B6%E9%9B%86%E4%B8%8E%E7%AE%A1%E7%90%86">日志收集与管理</a></li>
        </ul>
    </li>
</ul>





## 说明

<p>邮箱：909336740@qq.com


