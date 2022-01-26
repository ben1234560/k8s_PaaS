## 第七章——Promtheus监控k8s企业家应用

##### 前言：

> 配置是独立于程序的可配变量，同一份程序在不同配置下会有不同的行为。

##### 云原生（Cloud Native）程序的特点：

> - 程序的配置，通过设置环境变了传递到容器内部
> - 程序的配置，通过程序启动参数配置生效
> - 程序的配置，通过集中在配置中心进行统一换了（CRUD）

##### Devops工程师应该做什么？

> - 容器化公司自研的应用程序（通过Docker进行二次封装）
> - 推动容器化应用，转变为云原生应用（一次构建，到处使用）
> - 使用容器编排框架（kubernetes），合理、规范、专业的编排业务容器



### Prometheus监控软件概述

> 开源监控告警解决方案，[推荐文章](https://www.jianshu.com/p/60a50539243a)
>
> 当然一时半会你可能没那么快的去理解，那就跟我们先做下去你就会慢慢理解什么是时间序列数据
>
> [https://github.com/prometheus](https://github.com/prometheus)
>
> [https://prometheus.io](https://prometheus.io)

#### Prometheus的特点：

- 多维数据模型：由度量名称和键值对标识的时间序列数据
- 内置时间序列数据库：TSDB
- promQL：一种灵活的查询语言，可以利用多维数据完成复杂查询
- 基于HTTP的pull（拉取）方式采集时间序列数据
- 同时支持PushGateway组件收集数据
- 通过服务发现或静态配置发现目标
- 支持作为数据源接入Grafana

##### 我们将使用的官方架构图

![1582697010557](assets/1582697010557.png)

> **Prometheus Server**：服务核心组件，通过pull metrics从 Exporter 拉取和存储监控数据,并提供一套灵活的查询语言（PromQL）。
>
> **pushgateway**：类似一个中转站，Prometheus的server端只会使用pull方式拉取数据，但是某些节点因为某些原因只能使用push方式推送数据，那么它就是用来接收push而来的数据并暴露给Prometheus的server拉取的中转站，这里我们不做它。
>
> **Exporters/Jobs**：负责收集目标对象（host, container…）的性能数据，并通过 HTTP 接口供 Prometheus Server 获取。
>
> **Service Discovery**：服务发现，Prometheus支持多种服务发现机制：文件，DNS，Consul,Kubernetes,OpenStack,EC2等等。基于服务发现的过程并不复杂，通过第三方提供的接口，Prometheus查询到需要监控的Target列表，然后轮训这些Target获取监控数据。
>
> **Alertmanager**：从 Prometheus server 端接收到 alerts 后，会进行去除重复数据，分组，并路由到对方的接受方式，发出报警。常见的接收方式有：电子邮件，pagerduty 等。
>
> **UI页面的三种方法**：
>
> - Prometheus web UI：自带的（不怎么好用）
> - Grafana：美观、强大的可视化监控指标展示工具 
> - API clients：自己开发的监控展示工具
>
> **工作流程**：Prometheus Server定期从配置好的Exporters/Jobs中拉metrics，或者来着pushgateway发过来的metrics，或者其它的metrics，收集完后运行定义好的alert.rules（这个文件后面会讲到），记录时间序列或者向Alertmanager推送警报。更多了解<a href="https://github.com/ben1234560/k8s_PaaS/blob/master/%E5%8E%9F%E7%90%86%E5%8F%8A%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/Kubernetes%E7%9B%B8%E5%85%B3%E7%94%9F%E6%80%81.md#prometheusmetrics-server%E4%B8%8Ekubernetes%E7%9B%91%E6%8E%A7%E4%BD%93%E7%B3%BB">Prometheus、Metrics Server与Kubernetes监控体系</a>

##### 和zabbixc对比

| Prometheus                                                   | Zabbix                                                       |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 后端用golang开发，K8S也是go开发                              | 后端用C开发，界面用PHP开发                                   |
| 更适合云环境的监控，尤其是对K8S有着更好的支持                | 更适合监控物理机，虚拟机环境                                 |
| 监控数据存储在基于时间序列的数据库内，便于对已有数据进行新的聚合 | 监控数据存储在关系型数据库内，如MySQL，很难从现有数据中扩展维度 |
| 自身界面相对较弱，很多配置需要修改配置文件，但可以借由Grafana出图 | 图形化界面相对比较成熟                                       |
| 支持更大的集群规模，速度也更快                               | 集群规模上线为10000个节点                                    |
| 2015年后开始快速发展，社区活跃，使用场景越来越多             | 发展实际更长，对于很多监控场景，都有现成的解决方案           |

由于资源问题，我已经把不用的服务关掉了

![1583464421747](assets/1583464421747.png)

![1583464533840](assets/1583464533840.png)

![1583465087708](assets/1583465087708.png)



### 交付kube-state-metric

> **WHAT**：为prometheus采集k8s资源数据的exporter，能够采集绝大多数k8s内置资源的相关数据，例如pod、deploy、service等等。同时它也提供自己的数据，主要是资源采集个数和采集发生的异常次数统计

https://quay.io/repository/coreos/kube-state-metrics?tab=tags

~~~~
# 200机器，下载包：
~]# docker pull quay.io/coreos/kube-state-metrics:v1.5.0
~]# docker images|grep kube-state
~]# docker tag 91599517197a harbor.od.com/public/kube-state-metrics:v1.5.0
~]# docker push harbor.od.com/public/kube-state-metrics:v1.5.0
~~~~

![1583394715770](assets/1583394715770.png)

~~~~shell
# 200机器，准备资源配置清单：
~]# mkdir /data/k8s-yaml/kube-state-metrics
~]# cd /data/k8s-yaml/kube-state-metrics
kube-state-metrics]# vi rbac.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
    kubernetes.io/cluster-service: "true"
  name: kube-state-metrics
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
    kubernetes.io/cluster-service: "true"
  name: kube-state-metrics
rules:
- apiGroups:
  - ""
  resources:
  - configmaps
  - secrets
  - nodes
  - pods
  - services
  - resourcequotas
  - replicationcontrollers
  - limitranges
  - persistentvolumeclaims
  - persistentvolumes
  - namespaces
  - endpoints
  verbs:
  - list
  - watch
- apiGroups:
  - policy
  resources:
  - poddisruptionbudgets
  verbs:
  - list
  - watch
- apiGroups:
  - extensions
  resources:
  - daemonsets
  - deployments
  - replicasets
  verbs:
  - list
  - watch
- apiGroups:
  - apps
  resources:
  - statefulsets
  verbs:
  - list
  - watch
- apiGroups:
  - batch
  resources:
  - cronjobs
  - jobs
  verbs:
  - list
  - watch
- apiGroups:
  - autoscaling
  resources:
  - horizontalpodautoscalers
  verbs:
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
    kubernetes.io/cluster-service: "true"
  name: kube-state-metrics
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: kube-state-metrics
subjects:
- kind: ServiceAccount
  name: kube-state-metrics
  namespace: kube-system

kube-state-metrics]# vi dp.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "2"
  labels:
    grafanak8sapp: "true"
    app: kube-state-metrics
  name: kube-state-metrics
  namespace: kube-system
spec:
  selector:
    matchLabels:
      grafanak8sapp: "true"
      app: kube-state-metrics
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        grafanak8sapp: "true"
        app: kube-state-metrics
    spec:
      containers:
      - name: kube-state-metrics
        image: harbor.od.com/public/kube-state-metrics:v1.5.0
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8080
          name: http-metrics
          protocol: TCP
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /healthz
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 5
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 5
      serviceAccountName: kube-state-metrics
~~~~

![1583396103539](assets/1583396103539.png)

~~~~
# 应用清单，22机器：
~]# kubectl apply -f http://k8s-yaml.od.com/kube-state-metrics/rbac.yaml
~]# kubectl apply -f http://k8s-yaml.od.com/kube-state-metrics/dp.yaml
# 查询kube-metrics是否正常启动，curl哪个是在dashboard里看到的
~]# curl 172.7.21.8:8080/healthz
# out:ok
# 该命令是查看取出来的信息
~]# curl 172.7.21.8:8080/metric
~~~~

![1583396394541](assets/1583396394541.png)

![1583396300678](assets/1583396300678.png)

完成



### 交付node-exporter

> **WHAT:** 用来监控运算节点上的宿主机的资源信息，需要部署到所有运算节点

[node-exporter官方dockerhub地址](https://hub.docker.com/r/prom/node-exporter)

~~~
# 200机器，下载镜像并准备资源配置清单：
~]# docker pull prom/node-exporter:v0.15.0
~]# docker images|grep node-exporter
~]# docker tag 12d51ffa2b22 harbor.od.com/public/node-exporter:v0.15.0
~]# docker push harbor.od.com/public/node-exporter:v0.15.0
~]# mkdir /data/k8s-yaml/node-exporter/
~]# cd /data/k8s-yaml/node-exporter/
node-exporter]# vi ds.yaml
kind: DaemonSet
apiVersion: extensions/v1beta1
metadata:
  name: node-exporter
  namespace: kube-system
  labels:
    daemon: "node-exporter"
    grafanak8sapp: "true"
spec:
  selector:
    matchLabels:
      daemon: "node-exporter"
      grafanak8sapp: "true"
  template:
    metadata:
      name: node-exporter
      labels:
        daemon: "node-exporter"
        grafanak8sapp: "true"
    spec:
      volumes:
      - name: proc
        hostPath: 
          path: /proc
          type: ""
      - name: sys
        hostPath:
          path: /sys
          type: ""
      containers:
      - name: node-exporter
        image: harbor.od.com/public/node-exporter:v0.15.0
        imagePullPolicy: IfNotPresent
        args:
        - --path.procfs=/host_proc
        - --path.sysfs=/host_sys
        ports:
        - name: node-exporter
          hostPort: 9100
          containerPort: 9100
          protocol: TCP
        volumeMounts:
        - name: sys
          readOnly: true
          mountPath: /host_sys
        - name: proc
          readOnly: true
          mountPath: /host_proc
      hostNetwork: true
~~~



~~~
# 22机器，应用：
# 先看一下宿主机有没有9100端口，发现什么都没有
~]# netstat -luntp|grep 9100
~]# kubectl apply -f http://k8s-yaml.od.com/node-exporter/ds.yaml
# 创建完再看端口，可能启动的慢些，我是刷了3次才有
~]# netstat -luntp|grep 9100
~]# curl localhost:9100
# 该命令是查看取出来的信息
~]# curl localhost:9100/metrics
~~~

![1583396713104](assets/1583396713104.png)

![1583396743927](assets/1583396743927.png)

完成



### 交付cadvisor

> **WHAT**： 用来监控容器内部使用资源的信息

[cadvisor官方dockerhub镜像](https://hub.docker.com/r/google/cadvisor/tags)

~~~
# 200机器，下载镜像：
~]# docker pull google/cadvisor:v0.28.3
~]# docker images|grep cadvisor
~]# docker tag 75f88e3ec33 harbor.od.com/public/cadvisor:v0.28.3
~]# docker push harbor.od.com/public/cadvisor:v0.28.3
~]# mkdir /data/k8s-yaml/cadvisor/
~]# cd /data/k8s-yaml/cadvisor/
cadvisor]# vi ds.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: cadvisor
  namespace: kube-system
  labels:
    app: cadvisor
spec:
  selector:
    matchLabels:
      name: cadvisor
  template:
    metadata:
      labels:
        name: cadvisor
    spec:
      hostNetwork: true
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoExecute
      containers:
      - name: cadvisor
        image: harbor.od.com/public/cadvisor:v0.28.3
        imagePullPolicy: IfNotPresent
        volumeMounts:
        - name: rootfs
          mountPath: /rootfs
          readOnly: true
        - name: var-run
          mountPath: /var/run
        - name: sys
          mountPath: /sys
          readOnly: true
        - name: docker
          mountPath: /var/lib/docker
          readOnly: true
        ports:
          - name: http
            containerPort: 4194
            protocol: TCP
        readinessProbe:
          tcpSocket:
            port: 4194
          initialDelaySeconds: 5
          periodSeconds: 10
        args:
          - --housekeeping_interval=10s
          - --port=4194
      terminationGracePeriodSeconds: 30
      volumes:
      - name: rootfs
        hostPath:
          path: /
      - name: var-run
        hostPath:
          path: /var/run
      - name: sys
        hostPath:
          path: /sys
      - name: docker
        hostPath:
          path: /data/docker
~~~

![1583457963534](assets/1583457963534.png)

> 此时我们看到大多数节点都运行在21机器上，我们人为的让pod调度到22机器（当然即使你的大多数节点都运行在22机器上也没关系）

[^tolerations]: 可人为影响调度策略的方法。为什么需要它：kube-schedule是主控节点的策略，有预选节点和优选节点的策略，但往往生活中调度策略可能不是我们想要的。
[^tolerations-key]: 是否调度的是某节点，污点可以有多个
[^tolerations-effect-NoExecute]: 容忍NoExecute，其它的不容忍（如：NoSchedule等），即如过节点上的污点不是NoExecute，就不调度到该节点上，如果是，就可以调度。反之，如果是NoSchedule，那么节点上的污点如果是NoSchedule则可以容器，如果不是，则不可以。

> 可人为影响K8S调度策略的三种方法：
>
> - 污点、容忍方法：
>   - 污点：运算节点node上的污点（先在运算节点上打标签等 kubectl taint nodes node1 key1=value1:NoSchedule），污点可以有多个
>   - 容忍度：pod是否能够容忍污点
>   - 参考[kubernetes官网](https:kubernetes.io/zh/docs/concepts/configuration/taint-and-toleration/)
> - nodeName：让Pod运行再指定的node上
> - nodeSelector：通过标签选择器，让Pod运行再指定的一类node上

~~~
# 给21机器打个污点，22机器：
~]# kubectl taint node hdss7-21.host.com node-role.kubernetes.io/master=master:NoSchedule
~~~

![1581470696125](assets/1581470696125.png)

![1583458119938](assets/1583458119938.png)

~~~
# 21/22两个机器，修改软连接：
~]# mount -o remount,rw /sys/fs/cgroup/
~]# ln -s /sys/fs/cgroup/cpu,cpuacct /sys/fs/cgroup/cpuacct,cpu
~]# ls -l /sys/fs/cgroup/
~~~

> **mount -o remount, rw /sys/fs/cgroup**：重新以可读可写的方式挂载为已经挂载/sys/fs/cgroup
>
> **ln -s**：创建对应的软链接
>
> **ls -l**：显示不隐藏的文件与文件夹的详细信息

![1583458181722](assets/1583458181722.png)

~~~
# 22机器，应用资源清单：
~]# kubectl apply -f http://k8s-yaml.od.com/cadvisor/ds.yaml
~]# kubectl get pods -n kube-system -o wide
~~~

只有22机器上有，跟我们预期一样

![1583458264404](assets/1583458264404.png)

~~~
# 21机器，我们删掉污点：
~]# kubectl taint node hdss7-21.host.com node-role.kubernetes.io/master-
# out: node/hdss7-21.host.com untainted
~~~

看dashboard，污点已经没了

![1583458299094](assets/1583458299094.png)

在去Pods看，污点没了，pod就自动起来了

![1583458316885](assets/1583458316885.png)

完成

再修改下

![1583458394302](assets/1583458394302.png)





### 交付blackbox-exporter

> **WHAT**：监控业务容器存活性

~~~
# 200机器，下载镜像
~]# docker pull prom/blackbox-exporter:v0.15.1
~]# docker images|grep blackbox-exporter
~]# docker tag 81b70b6158be harbor.od.com/public/blackbox-exporter:v0.15.1
~]# docker push harbor.od.com/public/blackbox-exporter:v0.15.1
~]# mkdir /data/k8s-yaml/blackbox-exporter
~]# cd /data/k8s-yaml/blackbox-exporter
blackbox-exporter]# vi cm.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  labels:
    app: blackbox-exporter
  name: blackbox-exporter
  namespace: kube-system
data:
  blackbox.yml: |-
    modules:
      http_2xx:
        prober: http
        timeout: 2s
        http:
          valid_http_versions: ["HTTP/1.1", "HTTP/2"]
          valid_status_codes: [200,301,302]
          method: GET
          preferred_ip_protocol: "ip4"
      tcp_connect:
        prober: tcp
        timeout: 2s

blackbox-exporter]# vi dp.yaml
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  name: blackbox-exporter
  namespace: kube-system
  labels:
    app: blackbox-exporter
  annotations:
    deployment.kubernetes.io/revision: 1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: blackbox-exporter
  template:
    metadata:
      labels:
        app: blackbox-exporter
    spec:
      volumes:
      - name: config
        configMap:
          name: blackbox-exporter
          defaultMode: 420
      containers:
      - name: blackbox-exporter
        image: harbor.od.com/public/blackbox-exporter:v0.15.1
        imagePullPolicy: IfNotPresent
        args:
        - --config.file=/etc/blackbox_exporter/blackbox.yml
        - --log.level=info
        - --web.listen-address=:9115
        ports:
        - name: blackbox-port
          containerPort: 9115
          protocol: TCP
        resources:
          limits:
            cpu: 200m
            memory: 256Mi
          requests:
            cpu: 100m
            memory: 50Mi
        volumeMounts:
        - name: config
          mountPath: /etc/blackbox_exporter
        readinessProbe:
          tcpSocket:
            port: 9115
          initialDelaySeconds: 5
          timeoutSeconds: 5
          periodSeconds: 10
          successThreshold: 1
          failureThreshold: 3

blackbox-exporter]# vi svc.yaml
kind: Service
apiVersion: v1
metadata:
  name: blackbox-exporter
  namespace: kube-system
spec:
  selector:
    app: blackbox-exporter
  ports:
    - name: blackbox-port
      protocol: TCP
      port: 9115

blackbox-exporter]# vi ingress.yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: blackbox-exporter
  namespace: kube-system
spec:
  rules:
  - host: blackbox.od.com
    http:
      paths:
      - path: /
        backend:
          serviceName: blackbox-exporter
          servicePort: blackbox-port
~~~

![1583460099035](assets/1583460099035.png)

~~~
# 11机器，解析域名：
~]# vi /var/named/od.com.zone
serial 前滚一位
blackbox           A    10.4.7.10

~]# systemctl restart named
# 22机器
~]# dig -t A blackbox.od.com @192.168.0.2 +short
# out: 10.4.7.10
~~~

![1583460226033](assets/1583460226033.png)

~~~
# 22机器，应用：
~]# kubectl apply -f http://k8s-yaml.od.com/blackbox-exporter/cm.yaml
~]# kubectl apply -f http://k8s-yaml.od.com/blackbox-exporter/dp.yaml
~]# kubectl apply -f http://k8s-yaml.od.com/blackbox-exporter/svc.yaml
~]# kubectl apply -f http://k8s-yaml.od.com/blackbox-exporter/ingress.yaml
~~~

![1583460413279](assets/1583460413279.png)

[blackbox.od.com](blackbox.od.com)

![1583460433145](assets/1583460433145.png)

完成



### 安装部署Prometheus-server

> **WHAT**：服务核心组件，通过pull metrics从 Exporter 拉取和存储监控数据,并提供一套灵活的查询语言（PromQL）

[prometheus-server官网docker地址](https://hub.docker.com/r/prom/prometheus)

~~~~
# 200机器，准备镜像、资源清单：
~]# docker pull prom/prometheus:v2.14.0
~]# docker images|grep prometheus
~]# docker tag 7317640d555e harbor.od.com/infra/prometheus:v2.14.0
~]# docker push harbor.od.com/infra/prometheus:v2.14.0
~]# mkdir /data/k8s-yaml/prometheus
~]# cd /data/k8s-yaml/prometheus
prometheus]# vi rbac.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
    kubernetes.io/cluster-service: "true"
  name: prometheus
  namespace: infra
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
    kubernetes.io/cluster-service: "true"
  name: prometheus
rules:
- apiGroups:
  - ""
  resources:
  - nodes
  - nodes/metrics
  - services
  - endpoints
  - pods
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - configmaps
  verbs:
  - get
- nonResourceURLs:
  - /metrics
  verbs:
  - get
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
    kubernetes.io/cluster-service: "true"
  name: prometheus
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: prometheus
subjects:
- kind: ServiceAccount
  name: prometheus
  namespace: infra

prometheus]# vi dp.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "5"
  labels:
    name: prometheus
  name: prometheus
  namespace: infra
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 7
  selector:
    matchLabels:
      app: prometheus
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      containers:
      - name: prometheus
        image: harbor.od.com/infra/prometheus:v2.14.0
        imagePullPolicy: IfNotPresent
        command:
        - /bin/prometheus
        args:
        - --config.file=/data/etc/prometheus.yml
        - --storage.tsdb.path=/data/prom-db
        - --storage.tsdb.min-block-duration=10m
        - --storage.tsdb.retention=72h
        ports:
        - containerPort: 9090
          protocol: TCP
        volumeMounts:
        - mountPath: /data
          name: data
        resources:
          requests:
            cpu: "1000m"
            memory: "1.5Gi"
          limits:
            cpu: "2000m"
            memory: "3Gi"
      imagePullSecrets:
      - name: harbor
      securityContext:
        runAsUser: 0
      serviceAccountName: prometheus
      volumes:
      - name: data
        nfs:
          server: hdss7-200
          path: /data/nfs-volume/prometheus

prometheus]# vi svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: prometheus
  namespace: infra
spec:
  ports:
  - port: 9090
    protocol: TCP
    targetPort: 9090
  selector:
    app: prometheus

prometheus]# vi ingress.yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: traefik
  name: prometheus
  namespace: infra
spec:
  rules:
  - host: prometheus.od.com
    http:
      paths:
      - path: /
        backend:
          serviceName: prometheus
          servicePort: 9090

# 准备prometheus的配置文件：
prometheus]# mkdir /data/nfs-volume/prometheus
prometheus]# cd /data/nfs-volume/prometheus
prometheus]# mkdir {etc,prom-db}
prometheus]# cd etc/
etc]# cp /opt/certs/ca.pem .
etc]# cp -a /opt/certs/client.pem .
etc]# cp -a /opt/certs/client-key.pem .
etc]# prometheus.yml
global:
  scrape_interval:     15s
  evaluation_interval: 15s
scrape_configs:
- job_name: 'etcd'
  tls_config:
    ca_file: /data/etc/ca.pem
    cert_file: /data/etc/client.pem
    key_file: /data/etc/client-key.pem
  scheme: https
  static_configs:
  - targets:
    - '10.4.7.12:2379'
    - '10.4.7.21:2379'
    - '10.4.7.22:2379'
- job_name: 'kubernetes-apiservers'
  kubernetes_sd_configs:
  - role: endpoints
  scheme: https
  tls_config:
    ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
  bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
  relabel_configs:
  - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
    action: keep
    regex: default;kubernetes;https
- job_name: 'kubernetes-pods'
  kubernetes_sd_configs:
  - role: pod
  relabel_configs:
  - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
    action: keep
    regex: true
  - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
    action: replace
    target_label: __metrics_path__
    regex: (.+)
  - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
    action: replace
    regex: ([^:]+)(?::\d+)?;(\d+)
    replacement: $1:$2
    target_label: __address__
  - action: labelmap
    regex: __meta_kubernetes_pod_label_(.+)
  - source_labels: [__meta_kubernetes_namespace]
    action: replace
    target_label: kubernetes_namespace
  - source_labels: [__meta_kubernetes_pod_name]
    action: replace
    target_label: kubernetes_pod_name
- job_name: 'kubernetes-kubelet'
  kubernetes_sd_configs:
  - role: node
  relabel_configs:
  - action: labelmap
    regex: __meta_kubernetes_node_label_(.+)
  - source_labels: [__meta_kubernetes_node_name]
    regex: (.+)
    target_label: __address__
    replacement: ${1}:10255
- job_name: 'kubernetes-cadvisor'
  kubernetes_sd_configs:
  - role: node
  relabel_configs:
  - action: labelmap
    regex: __meta_kubernetes_node_label_(.+)
  - source_labels: [__meta_kubernetes_node_name]
    regex: (.+)
    target_label: __address__
    replacement: ${1}:4194
- job_name: 'kubernetes-kube-state'
  kubernetes_sd_configs:
  - role: pod
  relabel_configs:
  - action: labelmap
    regex: __meta_kubernetes_pod_label_(.+)
  - source_labels: [__meta_kubernetes_namespace]
    action: replace
    target_label: kubernetes_namespace
  - source_labels: [__meta_kubernetes_pod_name]
    action: replace
    target_label: kubernetes_pod_name
  - source_labels: [__meta_kubernetes_pod_label_grafanak8sapp]
    regex: .*true.*
    action: keep
  - source_labels: ['__meta_kubernetes_pod_label_daemon', '__meta_kubernetes_pod_node_name']
    regex: 'node-exporter;(.*)'
    action: replace
    target_label: nodename
- job_name: 'blackbox_http_pod_probe'
  metrics_path: /probe
  kubernetes_sd_configs:
  - role: pod
  params:
    module: [http_2xx]
  relabel_configs:
  - source_labels: [__meta_kubernetes_pod_annotation_blackbox_scheme]
    action: keep
    regex: http
  - source_labels: [__address__, __meta_kubernetes_pod_annotation_blackbox_port,  __meta_kubernetes_pod_annotation_blackbox_path]
    action: replace
    regex: ([^:]+)(?::\d+)?;(\d+);(.+)
    replacement: $1:$2$3
    target_label: __param_target
  - action: replace
    target_label: __address__
    replacement: blackbox-exporter.kube-system:9115
  - source_labels: [__param_target]
    target_label: instance
  - action: labelmap
    regex: __meta_kubernetes_pod_label_(.+)
  - source_labels: [__meta_kubernetes_namespace]
    action: replace
    target_label: kubernetes_namespace
  - source_labels: [__meta_kubernetes_pod_name]
    action: replace
    target_label: kubernetes_pod_name
- job_name: 'blackbox_tcp_pod_probe'
  metrics_path: /probe
  kubernetes_sd_configs:
  - role: pod
  params:
    module: [tcp_connect]
  relabel_configs:
  - source_labels: [__meta_kubernetes_pod_annotation_blackbox_scheme]
    action: keep
    regex: tcp
  - source_labels: [__address__, __meta_kubernetes_pod_annotation_blackbox_port]
    action: replace
    regex: ([^:]+)(?::\d+)?;(\d+)
    replacement: $1:$2
    target_label: __param_target
  - action: replace
    target_label: __address__
    replacement: blackbox-exporter.kube-system:9115
  - source_labels: [__param_target]
    target_label: instance
  - action: labelmap
    regex: __meta_kubernetes_pod_label_(.+)
  - source_labels: [__meta_kubernetes_namespace]
    action: replace
    target_label: kubernetes_namespace
  - source_labels: [__meta_kubernetes_pod_name]
    action: replace
    target_label: kubernetes_pod_name
- job_name: 'traefik'
  kubernetes_sd_configs:
  - role: pod
  relabel_configs:
  - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scheme]
    action: keep
    regex: traefik
  - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
    action: replace
    target_label: __metrics_path__
    regex: (.+)
  - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
    action: replace
    regex: ([^:]+)(?::\d+)?;(\d+)
    replacement: $1:$2
    target_label: __address__
  - action: labelmap
    regex: __meta_kubernetes_pod_label_(.+)
  - source_labels: [__meta_kubernetes_namespace]
    action: replace
    target_label: kubernetes_namespace
  - source_labels: [__meta_kubernetes_pod_name]
    action: replace
    target_label: kubernetes_pod_name
~~~~

> **cp -a**：在复制目录时使用，它保留链接、文件属性，并复制目录下的所有内容

![1583461823355](assets/1583461823355.png)

~~~
# 11机器， 解析域名，有ingress就有页面就需要解析：
~]# vi /var/named/od.com.zone
serial 前滚一位
prometheus         A    10.4.7.10

~]# systemctl restart named
~]# dig -t A prometheus.od.com @10.4.7.11 +short
# out:10.4.7.10
~~~

![1582704423890](assets/1582704423890.png)

~~~
# 22机器，应用配置清单：
~]# kubectl apply -f http://k8s-yaml.od.com/prometheus/rbac.yaml
~]# kubectl apply -f http://k8s-yaml.od.com/prometheus/dp.yaml
~]# kubectl apply -f http://k8s-yaml.od.com/prometheus/svc.yaml
~]# kubectl apply -f http://k8s-yaml.od.com/prometheus/ingress.yaml
~~~

![1583461941453](assets/1583461941453.png)

![1583462134250](assets/1583462134250.png)

[prometheus.od.com](prometheus.od.com)

> 这就是Prometheus自带的UI页面，现在你就知道为什么我们需要Grafana来替代了，如果你还不清楚，等下看Grafana的页面你就知道了

![1583462164465](assets/1583462164465.png)

![1583462217169](assets/1583462217169.png)

完成



### 配置Prometheus监控业务容器

##### 先配置traefik

![1583462282296](assets/1583462282296.png)

~~~
# Edit a Daemon Set，添加以下内容，记得给上面加逗号:
"annotations": {
  "prometheus_io_scheme": "traefik",
  "prometheus_io_path": "/metrics",
  "prometheus_io_port": "8080"
}
# 直接加进去update，会自动对齐 
~~~

![1583462379871](assets/1583462379871.png)

删掉两个对应的pod让它重启

![1583462451073](assets/1583462451073.png)

~~~
# 22机器，查看下，如果起不来就用命令行的方式强制删除：
~]# kubectl get pods -n kube-system
~]# kubectl delete pods traefik-ingress-g26kw -n kube-system --force --grace-period=0
~~~

![1583462566364](assets/1583462566364.png)

启动成功后，去Prometheus查看

刷新后，可以看到是traefik2/2，已经有了

![1583462600531](assets/1583462600531.png)

完成



##### blackbox

我们起一个dubbo-service，之前我们最后做的是Apollo的版本，现在我们的Apollo已经关了（因为消耗资源），现在需要起更早之前不是Apollo的版本。

我们去harbor里面找

![1583465190219](assets/1583465190219.png)

> 我的Apollo的版本可能比你的多一个，不用在意，那是做实验弄的

修改版本信息

![1583466214230](assets/1583466214230.png)

![1583466251914](assets/1583466251914.png)

在把scale改成1

![1583466284890](assets/1583466284890.png)

查看POD的LOGS日志

![1583466310189](assets/1583466310189.png)

翻页查看，已经启动

![1583466328146](assets/1583466328146.png)

如何监控存活性，只需要修改配置

![1584699708597](assets/1584699708597.png)

~~~
# Edit a Deployment（TCP），添加以下内容
"annotations": {
  "blackbox_port": "20880",
  "blackbox_scheme": "tcp"
}
# 直接加进去update，会自动对齐 
~~~

![1583466938931](assets/1583466938931.png)

UPDATE后，已经running起来了

![1583467301614](assets/1583467301614.png)

[prometheus.od.com](prometheus.od.com)刷新，自动发现业务

![1583466979716](assets/1583466979716.png)

[blackbox.od.com](blackbox.od.com) 刷新

![1583467331128](assets/1583467331128.png)

同样的，我们把dubbo-consumer也弄进来

先去harbor找一个不是Apollo的版本（为什么要用不是Apollo的版本前面已经说了）

![1583503611435](assets/1583503611435.png)

修改版本信息，并添加annotations

~~~
# Edit a Deployment(http)，添加以下内容，记得前面的逗号
"annotations":{
  "blackbox_path": "/hello?name=health",
  "blackbox_port": "8080",
  "blackbox_scheme": "http"
}
# 直接加进去update，会自动对齐 
~~~

![1583503670313](assets/1583503670313.png)

![1583504095794](assets/1583504095794.png)

UPDATE后，把scale改成1

![1583503796291](assets/1583503796291.png)

确保起来了

![1583503811855](assets/1583503811855.png)

![1583503829457](assets/1583503829457.png)

[prometheus.od.com](prometheus.od.com)刷新，自动发现业务

![1583503935815](assets/1583503935815.png)

[blackbox.od.com](blackbox.od.com) 刷新

![1583504112078](assets/1583504112078.png)



### 安装部署配置Grafana

> **WHAT**：美观、强大的可视化监控指标展示工具 
>
> **WHY**：用来代替prometheus原生UI界面

~~~
# 200机器，准备镜像、资源配置清单：
~]# docker pull grafana/grafana:5.4.2
~]# docker images|grep grafana
~]# docker tag 6f18ddf9e552 harbor.od.com/infra/grafana:v5.4.2
~]# docker push harbor.od.com/infra/grafana:v5.4.2
~]# mkdir /data/k8s-yaml/grafana/ /data/nfs-volume/grafana
~]# cd /data/k8s-yaml/grafana/
grafana]# vi rbac.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
    kubernetes.io/cluster-service: "true"
  name: grafana
rules:
- apiGroups:
  - "*"
  resources:
  - namespaces
  - deployments
  - pods
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
    kubernetes.io/cluster-service: "true"
  name: grafana
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: grafana
subjects:
- kind: User
  name: k8s-node

grafana]# vi dp.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    app: grafana
    name: grafana
  name: grafana
  namespace: infra
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 7
  selector:
    matchLabels:
      name: grafana
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: grafana
        name: grafana
    spec:
      containers:
      - name: grafana
        image: harbor.od.com/infra/grafana:v5.4.2
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 3000
          protocol: TCP
        volumeMounts:
        - mountPath: /var/lib/grafana
          name: data
      imagePullSecrets:
      - name: harbor
      securityContext:
        runAsUser: 0
      volumes:
      - nfs:
          server: hdss7-200
          path: /data/nfs-volume/grafana
        name: data

grafana]# vi svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: grafana
  namespace: infra
spec:
  ports:
  - port: 3000
    protocol: TCP
    targetPort: 3000
  selector:
    app: grafana

grafana]# vi ingress.yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: grafana
  namespace: infra
spec:
  rules:
  - host: grafana.od.com
    http:
      paths:
      - path: /
        backend:
          serviceName: grafana
          servicePort: 3000

~~~

![1583504719781](assets/1583504719781.png)

~~~
# 11机器，解析域名:
~]# vi /var/named/od.com.zone
serial 前滚一位

grafana            A    10.4.7.10
~]# systemctl restart named
~]# ping grafana.od.com
~~~

![1582705048800](assets/1582705048800.png)

~~~~
# 22机器，应用配置清单：
~]# kubectl apply -f http://k8s-yaml.od.com/grafana/rbac.yaml
~]# kubectl apply -f http://k8s-yaml.od.com/grafana/dp.yaml
~]# kubectl apply -f http://k8s-yaml.od.com/grafana/svc.yaml
~]# kubectl apply -f http://k8s-yaml.od.com/grafana/ingress.yaml
~~~~

![1583504865941](assets/1583504865941.png)

[grafana.od.com](grafana.od.com)

默认账户和密码都是admin

修改密码：admin123

![1583504898029](assets/1583504898029.png)

修改配置，修改如下图

![1583505029816](assets/1583505029816.png)

##### 装插件

进入容器

![1583505097409](assets/1583505097409.png)

~~~
# 第一个：kubenetes App
grafana# grafana-cli plugins install grafana-kubernetes-app
# 第二个：Clock Pannel
grafana# grafana-cli plugins install grafana-clock-panel
# 第三个：Pie Chart
grafana# grafana-cli plugins install grafana-piechart-panel
# 第四个：D3Gauge
grafana# grafana-cli plugins install briangann-gauge-panel
# 第五个：Discrete
grafana# grafana-cli plugins install natel-discrete-panel
~~~

![1583505305939](assets/1583505305939.png)

装完后，可以在200机器查看

~~~
# 200机器：
cd /data/nfs-volume/grafana/plugins/
plugins]# ll
~~~

![1583505462177](assets/1583505462177.png)

删掉让它重启

![1583505490948](assets/1583505490948.png)

重启完成后



查看[grafana.od.com](grafana.od.com)，刚刚安装的5个插件都在里面了（记得检查是否在里面了）

![1583505547061](assets/1583505547061.png)

##### 添加数据源：Add data source

![1583505581557](assets/1583505581557.png)

![1583505600031](assets/1583505600031.png)

~~~
# 填入参数：
URL:http://prometheus.od.com
TLS Client Auth✔    With CA Cert✔
~~~

![1583505840252](assets/1583505840252.png)

~~~
# 填入参数对应的pem参数：
# 200机器拿ca等：
~]# cat /opt/certs/ca.pem
~]# cat /opt/certs/client.pem
~]# cat /opt/certs/client-key.pem
~~~

![1583505713033](assets/1583505713033.png)

![1583505856093](assets/1583505856093.png)

保存

然后我们去配置plugins里面的kubernetes

![1583505923109](assets/1583505923109.png)

![1583505938700](assets/1583505938700.png)

右侧就多了个按钮，点击进去

![1583505969865](assets/1583505969865.png)

~~~
# 按参数填入：
Name:myk8s
URL:https://10.4.7.10:7443
Access:Server
TLS Client Auth✔    With CA Cert✔
~~~

![1583506058483](assets/1583506058483.png)

~~~
# 填入参数：
# 200机器拿ca等：
~]# cat /opt/certs/ca.pem
~]# cat /opt/certs/client.pem
~]# cat /opt/certs/client-key.pem
~~~

![1583506131529](assets/1583506131529.png)

save后再点击右侧框的图标，并点击Name

![1583506163546](assets/1583506163546.png)

可能抓取数据的时间会稍微慢些（两分钟左右）

![1583506184293](assets/1583506184293.png)

![1583506503213](assets/1583506503213.png)

点击右上角的K8s Cluster，选择你要看的东西

![1583506545308](assets/1583506545308.png)

由于K8s Container里面数据不全，如下图

![1583506559069](assets/1583506559069.png)

我们改下，把Cluster删了

![1583506631392](assets/1583506631392.png)

![1583506645982](assets/1583506645982.png)

container也删了

![1583506675876](assets/1583506675876.png)

deployment也删了

![1583506695618](assets/1583506695618.png)

node也删了

![1583506709705](assets/1583506709705.png)

![1583506730713](assets/1583506730713.png)

![1583506744138](assets/1583506744138.png)

把我给你准备的dashboard的json文件import进来

![1583543092886](assets/1583543092886.png)

![1583543584476](assets/1583543584476.png)

![1583543602130](assets/1583543602130.png)

用同样的方法把node、deployment、cluster、container这4个分别import进来

![1583543698727](assets/1583543698727.png)

可以都看一下，已经正常了

然后把etcd、generic、traefik也import进来

![1583543809740](assets/1583543809740.png)

![1583543831830](assets/1583543831830.png)

还有另外一种import的方法（使用官网的）：

[grafana官网](https://grafana.com/grafana/dashboards)

找一个别人写好的点进去

![1584241883144](assets/1584241883144.png)

这个编号可以直接用

![1584241903882](assets/1584241903882.png)

如下图，我们装blackbox的编号是9965

![1584242072703](assets/1584242072703.png)

![1584242093513](assets/1584242093513.png)

把名字和Prometheus修改一下

![1584242164621](assets/1584242164621.png)

或者，你也可以用我上传的（我用的是7587）

![1583543931644](assets/1583543931644.png)

你可以两个都用，自己做对比，都留着也可以，就是占一些资源

JMX

![1583544009027](assets/1583544009027.png)

这个里面还什么都没有

![1583544017606](assets/1583544017606.png)



#### 把Dubbo微服务数据弄到Grafana

dubbo-service

![1583544062372](assets/1583544062372.png)

~~~
# Edit a Daemon Set，添加以下内容，注意给上一行加逗号
  "prometheus_io_scrape": "true",
  "prometheus_io_port": "12346",
  "prometheus_io_path": "/"
# 直接加进去update，会自动对齐，
~~~

![1583544144136](assets/1583544144136.png)

dubbo-consumer

![1583544157268](assets/1583544157268.png)

~~~
# Edit a Daemon Set，添加以下内容，注意给上一行加逗号
  "prometheus_io_scrape": "true",
  "prometheus_io_port": "12346",
  "prometheus_io_path": "/"
# 直接加进去update，会自动对齐，
~~~

![1583544192459](assets/1583544192459.png)

刷新JMX（可能有点慢，我等了1分钟才出来service，我机器不行了）

![1583544446817](assets/1583544446817.png)

完成

> 此时你可以感受到，Grafana明显比K8S自带的UI界面更加人性化



### 安装部署alertmanager

> **WHAT**： 从 Prometheus server 端接收到 alerts 后，会进行去除重复数据，分组，并路由到对方的接受方式，发出报警。常见的接收方式有：电子邮件，pagerduty 等。
>
> **WHY**：使得系统的警告随时让我们知道

~~~
# 200机器，准备镜像、资源清单：
~]# mkdir /data/k8s-yaml/alertmanager
~]# cd /data/k8s-yaml/alertmanager
alertmanager]# docker pull docker.io/prom/alertmanager:v0.14.0
# 注意，这里你如果不用14版本可能会报错
alertmanager]# docker images|grep alert
alertmanager]# docker tag 23744b2d645c harbor.od.com/infra/alertmanager:v0.14.0
alertmanager]# docker push harbor.od.com/infra/alertmanager:v0.14.0
# 注意下面记得修改成你自己的邮箱等信息，还有中文注释可以删掉
alertmanager]# vi cm.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: alertmanager-config
  namespace: infra
data:
  config.yml: |-
    global:
      # 在没有报警的情况下声明为已解决的时间
      resolve_timeout: 5m
      # 配置邮件发送信息
      smtp_smarthost: 'smtp.163.com:25'
      smtp_from: 'ben909336740@163.com'
      smtp_auth_username: 'ben909336740@163.com'
      smtp_auth_password: 'xxxxxx'
      smtp_require_tls: false
    # 所有报警信息进入后的根路由，用来设置报警的分发策略
    route:
      # 这里的标签列表是接收到报警信息后的重新分组标签，例如，接收到的报警信息里面有许多具有 cluster=A 和 alertname=LatncyHigh 这样的标签的报警信息将会批量被聚合到一个分组里面
      group_by: ['alertname', 'cluster']
      # 当一个新的报警分组被创建后，需要等待至少group_wait时间来初始化通知，这种方式可以确保您能有足够的时间为同一分组来获取多个警报，然后一起触发这个报警信息。
      group_wait: 30s

      # 当第一个报警发送后，等待'group_interval'时间来发送新的一组报警信息。
      group_interval: 5m

      # 如果一个报警信息已经发送成功了，等待'repeat_interval'时间来重新发送他们
      repeat_interval: 5m

      # 默认的receiver：如果一个报警没有被一个route匹配，则发送给默认的接收器
      receiver: default

    receivers:
    - name: 'default'
      email_configs:
      - to: '909336740@qq.com'
        send_resolved: true

alertmanager]# vi dp.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: alertmanager
  namespace: infra
spec:
  replicas: 1
  selector:
    matchLabels:
      app: alertmanager
  template:
    metadata:
      labels:
        app: alertmanager
    spec:
      containers:
      - name: alertmanager
        image: harbor.od.com/infra/alertmanager:v0.14.0
        args:
          - "--config.file=/etc/alertmanager/config.yml"
          - "--storage.path=/alertmanager"
        ports:
        - name: alertmanager
          containerPort: 9093
        volumeMounts:
        - name: alertmanager-cm
          mountPath: /etc/alertmanager
      volumes:
      - name: alertmanager-cm
        configMap:
          name: alertmanager-config
      imagePullSecrets:
      - name: harbor

alertmanager]# vi svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: alertmanager
  namespace: infra
spec:
  selector: 
    app: alertmanager
  ports:
    - port: 80
      targetPort: 9093
~~~

![1583547933312](assets/1583547933312.png)

~~~
# 22机器，应用清单：
~]# kubectl apply -f http://k8s-yaml.od.com/alertmanager/cm.yaml
~]# kubectl apply -f http://k8s-yaml.od.com/alertmanager/dp.yaml
~]# kubectl apply -f http://k8s-yaml.od.com/alertmanager/svc.yaml
~~~

![1583545326722](assets/1583545326722.png)

![1583545352951](assets/1583545352951.png)

~~~
# 200机器，配置报警规则：
~]# vi /data/nfs-volume/prometheus/etc/rules.yml
groups:
- name: hostStatsAlert
  rules:
  - alert: hostCpuUsageAlert
    expr: sum(avg without (cpu)(irate(node_cpu{mode!='idle'}[5m]))) by (instance) > 0.85
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "{{ $labels.instance }} CPU usage above 85% (current value: {{ $value }}%)"
  - alert: hostMemUsageAlert
    expr: (node_memory_MemTotal - node_memory_MemAvailable)/node_memory_MemTotal > 0.85
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "{{ $labels.instance }} MEM usage above 85% (current value: {{ $value }}%)"
  - alert: OutOfInodes
    expr: node_filesystem_free{fstype="overlay",mountpoint ="/"} / node_filesystem_size{fstype="overlay",mountpoint ="/"} * 100 < 10
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Out of inodes (instance {{ $labels.instance }})"
      description: "Disk is almost running out of available inodes (< 10% left) (current value: {{ $value }})"
  - alert: OutOfDiskSpace
    expr: node_filesystem_free{fstype="overlay",mountpoint ="/rootfs"} / node_filesystem_size{fstype="overlay",mountpoint ="/rootfs"} * 100 < 10
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Out of disk space (instance {{ $labels.instance }})"
      description: "Disk is almost full (< 10% left) (current value: {{ $value }})"
  - alert: UnusualNetworkThroughputIn
    expr: sum by (instance) (irate(node_network_receive_bytes[2m])) / 1024 / 1024 > 100
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Unusual network throughput in (instance {{ $labels.instance }})"
      description: "Host network interfaces are probably receiving too much data (> 100 MB/s) (current value: {{ $value }})"
  - alert: UnusualNetworkThroughputOut
    expr: sum by (instance) (irate(node_network_transmit_bytes[2m])) / 1024 / 1024 > 100
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Unusual network throughput out (instance {{ $labels.instance }})"
      description: "Host network interfaces are probably sending too much data (> 100 MB/s) (current value: {{ $value }})"
  - alert: UnusualDiskReadRate
    expr: sum by (instance) (irate(node_disk_bytes_read[2m])) / 1024 / 1024 > 50
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Unusual disk read rate (instance {{ $labels.instance }})"
      description: "Disk is probably reading too much data (> 50 MB/s) (current value: {{ $value }})"
  - alert: UnusualDiskWriteRate
    expr: sum by (instance) (irate(node_disk_bytes_written[2m])) / 1024 / 1024 > 50
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Unusual disk write rate (instance {{ $labels.instance }})"
      description: "Disk is probably writing too much data (> 50 MB/s) (current value: {{ $value }})"
  - alert: UnusualDiskReadLatency
    expr: rate(node_disk_read_time_ms[1m]) / rate(node_disk_reads_completed[1m]) > 100
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Unusual disk read latency (instance {{ $labels.instance }})"
      description: "Disk latency is growing (read operations > 100ms) (current value: {{ $value }})"
  - alert: UnusualDiskWriteLatency
    expr: rate(node_disk_write_time_ms[1m]) / rate(node_disk_writes_completedl[1m]) > 100
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Unusual disk write latency (instance {{ $labels.instance }})"
      description: "Disk latency is growing (write operations > 100ms) (current value: {{ $value }})"
- name: http_status
  rules:
  - alert: ProbeFailed
    expr: probe_success == 0
    for: 1m
    labels:
      severity: error
    annotations:
      summary: "Probe failed (instance {{ $labels.instance }})"
      description: "Probe failed (current value: {{ $value }})"
  - alert: StatusCode
    expr: probe_http_status_code <= 199 OR probe_http_status_code >= 400
    for: 1m
    labels:
      severity: error
    annotations:
      summary: "Status Code (instance {{ $labels.instance }})"
      description: "HTTP status code is not 200-399 (current value: {{ $value }})"
  - alert: SslCertificateWillExpireSoon
    expr: probe_ssl_earliest_cert_expiry - time() < 86400 * 30
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "SSL certificate will expire soon (instance {{ $labels.instance }})"
      description: "SSL certificate expires in 30 days (current value: {{ $value }})"
  - alert: SslCertificateHasExpired
    expr: probe_ssl_earliest_cert_expiry - time()  <= 0
    for: 5m
    labels:
      severity: error
    annotations:
      summary: "SSL certificate has expired (instance {{ $labels.instance }})"
      description: "SSL certificate has expired already (current value: {{ $value }})"
  - alert: BlackboxSlowPing
    expr: probe_icmp_duration_seconds > 2
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Blackbox slow ping (instance {{ $labels.instance }})"
      description: "Blackbox ping took more than 2s (current value: {{ $value }})"
  - alert: BlackboxSlowRequests
    expr: probe_http_duration_seconds > 2 
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Blackbox slow requests (instance {{ $labels.instance }})"
      description: "Blackbox request took more than 2s (current value: {{ $value }})"
  - alert: PodCpuUsagePercent
    expr: sum(sum(label_replace(irate(container_cpu_usage_seconds_total[1m]),"pod","$1","container_label_io_kubernetes_pod_name", "(.*)"))by(pod) / on(pod) group_right kube_pod_container_resource_limits_cpu_cores *100 )by(container,namespace,node,pod,severity) > 80
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Pod cpu usage percent has exceeded 80% (current value: {{ $value }}%)"

# 在最后面添加如下内容
~]# vi /data/nfs-volume/prometheus/etc/prometheus.yml
alerting:
  alertmanagers:
    - static_configs:
        - targets: ["alertmanager"]
rule_files:
 - "/data/etc/rules.yml"
~~~

> ![1583545590235](assets/1583545590235.png)
>
> **rules.yml文件**：这个文件就是报警规则
>
> 这时候可以重启Prometheus的pod，但生产商因为Prometheus太庞大，删掉容易拖垮集群，所以我们用另外一种方法，平滑加载（Prometheus支持）：

~~~
# 21机器，因为我们起的Prometheus是在21机器，平滑加载:
~]# ps aux|grep prometheus
~]# kill -SIGHUP 1488
~~~

![1583545718137](assets/1583545718137.png)

![1583545762475](assets/1583545762475.png)

> 这时候报警规则就都有了
>



### 测试alertmanager报警功能

先把对应的两个邮箱的stmp都打开

![1583721318338](assets/1583721318338.png)

![1583721408460](assets/1583721408460.png)

我们测试一下，把dubbo-service停了，这样consumer就会报错

把service的scale改成0

![1583545840349](assets/1583545840349.png)

[blackbox.od.com](blackbox.od.com)查看，已经failure了

![1583545937643](assets/1583545937643.png)

[prometheus.od.com.alerts](prometheus.od.com.alerts)查看，两个变红了（一开始是变黄）

![1583548102650](assets/1583548102650.png)

![1583545983131](assets/1583545983131.png)

这时候可以在163邮箱看到已发送的报警

![1583721856162](assets/1583721856162.png)

QQ邮箱收到报警

![1583721899076](assets/1583721899076.png)

完成（service的scale记得改回1）

> 关于rules.yml：报警不能错报也不能漏报，在实际应用中，我们需要不断的修改rules的规则，以来贴近我们公司的实际需求。



#### 资源不足时，可关闭部分非必要资源

~~~
# 22机器，也可以用dashboard操作：
~]# kubectl scale deployment grafana --replicas=0 -n infra
# out : deployment.extensions/grafana scaled
~]# kubectl scale deployment alertmanager --replicas=0 -n infra
# out : deployment.extensions/alertmanager scaled
~]# kubectl scale deployment prometheus --replicas=0 -n infra
# out : deployment.extensions/prometheus scaled
~~~



### 通过K8S部署dubbo微服务接入ELK架构

> **WHAT**：ELK是三个开源软件的缩写，分别是：
>
> - E——ElasticSearch：分布式搜索引擎，提供搜集、分析、存储数据三大功能。
> - L——LogStash：对日志的搜集、分析、过滤日志的工具，支持大量的数据获取方式。
> - K——Kibana：为 Logstash 和 ElasticSearch 提供的日志分析友好的 Web 界面，可以帮助汇总、分析和搜索重要数据日志。
> - 还有新增的FileBeat（流式日志收集器）：轻量级的日志收集处理工具，占用资源少，适合于在各个服务器上搜集日志后传输给Logstash，官方也推荐此工具，用来替代部分原本Logstash的工作。[收集日志的多种方式及原理](https://github.com/ben1234560/k8s_PaaS/blob/master/%E5%8E%9F%E7%90%86%E5%8F%8A%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/Kubernetes%E7%9B%B8%E5%85%B3%E7%94%9F%E6%80%81.md#%E6%97%A5%E5%BF%97%E6%94%B6%E9%9B%86%E4%B8%8E%E7%AE%A1%E7%90%86)
>
> **WHY**： 随着容器编排的进行，业务容器在不断的被创建、摧毁、迁移、扩容缩容等，面对如此海量的数据，又分布在各个不同的地方，我们不可能用传统的方法登录到每台机器看，所以我们需要建立一套集中的方法。我们需要这样一套日志手机、分析的系统：
>
> - 收集——采集多种来源的日志数据（流式日志收集器）
> - 传输——稳定的把日志数据传输到中央系统（消息队列）
> - 存储——将日志以结构化数据的形式存储起来（搜索引擎）
> - 分析——支持方便的分析、检索等，有GUI管理系统（前端）
> - 警告——提供错误报告，监控机制（监控工具）
>
> #### 这就是ELK

#### ELK Stack概述

![1581729929995](assets/1581729929995.png)

> **c1/c2**：container（容器）的缩写
>
> **filebeat**：收集业务容器的日志，把c和filebeat放在一个pod里让他们一起跑，这样耦合就紧了
>
> **kafka**：高吞吐量的[分布式](https://baike.baidu.com/item/%E5%88%86%E5%B8%83%E5%BC%8F/19276232)发布订阅消息系统，它可以处理消费者在网站中的所有动作流数据。filebeat收集数据以Topic形式发布到kafka。
>
> **Topic**：Kafka数据写入操作的基本单元
>
> **logstash**：取kafka里的topic，然后再往ElasticSearch上传（异步过程，即又取又传）
>
> **index-pattern**：把数据按环境分（按prod和test分），并传到kibana
>
> **kibana**：展示数据



### 制作tomcat容器的底包镜像

> 尝试用tomcat的方式，因为很多公司老项目都是用tomcat跑起来，之前我们用的是springboot

[tomcat官网](tomcat.apache.org)

![1583558092305](assets/1583558092305.png)

~~~
# 200 机器：
cd /opt/src/
# 你也可以直接用我上传的，因为版本一直在变，之前的版本你是下载不下来的，如何查看新版本如上图
src]# wget https://archive.apache.org/dist/tomcat/tomcat-8/v8.5.51/bin/apache-tomcat-8.5.51.tar.gz
src]# mkdir /data/dockerfile/tomcat
src]# tar xfv  apache-tomcat-8.5.51.tar.gz -C /data/dockerfile/tomcat
src]# cd /data/dockerfile/tomcat
# 配置tomcat-关闭AJP端口
tomcat]# vi apache-tomcat-8.5.51/conf/server.xml
# 找到AJP，注释掉相应的一行，结果如下图，8.5.51是已经自动注释掉的
~~~

![1583558364369](assets/1583558364369.png)

~~~
# 200机器，删掉不需要的日志：
tomcat]# vi apache-tomcat-8.5.51/conf/logging.properties
# 删掉3manager，4host-manager的handlers，并注释掉相关的，结果如下图
# 日志级别改成INFO
~~~

![1583558487445](assets/1583558487445.png)

![1583558525033](assets/1583558525033.png)

![1583558607700](assets/1583558607700.png)

~~~
# 200机器，准备Dockerfile：
tomcat]# vi Dockerfile
From harbor.od.com/public/jre:8u112
RUN /bin/cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime &&\ 
    echo 'Asia/Shanghai' >/etc/timezone
ENV CATALINA_HOME /opt/tomcat
ENV LANG zh_CN.UTF-8
ADD apache-tomcat-8.5.51/ /opt/tomcat
ADD config.yml /opt/prom/config.yml
ADD jmx_javaagent-0.3.1.jar /opt/prom/jmx_javaagent-0.3.1.jar
WORKDIR /opt/tomcat
ADD entrypoint.sh /entrypoint.sh
CMD ["/entrypoint.sh"]

tomcat]# vi config.yml
---
rules:
  - pattern: '-*'

tomcat]# wget https://repo1.maven.org/maven2/io/prometheus/jmx/jmx_prometheus_javaagent/0.3.1/jmx_prometheus_javaagent-0.3.1.jar -O jmx_javaagent-0.3.1.jar
tomcat]# vi entrypoint.sh
#!/bin/bash
M_OPTS="-Duser.timezone=Asia/Shanghai -javaagent:/opt/prom/jmx_javaagent-0.3.1.jar=$(hostname -i):${M_PORT:-"12346"}:/opt/prom/config.yml"
C_OPTS=${C_OPTS}
MIN_HEAP=${MIN_HEAP:-"128m"}
MAX_HEAP=${MAX_HEAP:-"128m"}
JAVA_OPTS=${JAVA_OPTS:-"-Xmn384m -Xss256k -Duser.timezone=GMT+08  -XX:+DisableExplicitGC -XX:+UseConcMarkSweepGC -XX:+UseParNewGC -XX:+CMSParallelRemarkEnabled -XX:+UseCMSCompactAtFullCollection -XX:CMSFullGCsBeforeCompaction=0 -XX:+CMSClassUnloadingEnabled -XX:LargePageSizeInBytes=128m -XX:+UseFastAccessorMethods -XX:+UseCMSInitiatingOccupancyOnly -XX:CMSInitiatingOccupancyFraction=80 -XX:SoftRefLRUPolicyMSPerMB=0 -XX:+PrintClassHistogram  -Dfile.encoding=UTF8 -Dsun.jnu.encoding=UTF8"}
CATALINA_OPTS="${CATALINA_OPTS}"
JAVA_OPTS="${M_OPTS} ${C_OPTS} -Xms${MIN_HEAP} -Xmx${MAX_HEAP} ${JAVA_OPTS}"
sed -i -e "1a\JAVA_OPTS=\"$JAVA_OPTS\"" -e "1a\CATALINA_OPTS=\"$CATALINA_OPTS\"" /opt/tomcat/bin/catalina.sh

cd /opt/tomcat && /opt/tomcat/bin/catalina.sh run 2>&1 >> /opt/tomcat/logs/stdout.log

tomcat]# chmod u+x entrypoint.sh
tomcat]# ll
tomcat]# docker build . -t harbor.od.com/base/tomcat:v8.5.51
tomcat]# docker push harbor.od.com/base/tomcat:v8.5.51
~~~

> **Dockerfile文件解析**：
>
> - FROM：镜像地址
> - RUN：修改时区
> - ENV：设置环境变量，把tomcat软件放到opt下
> - ENV：设置环境变量，字符集用zh_CN.UTF-8
> - ADD：把apache-tomcat-8.5.50包放到/opt/tomcat下
> - ADD：让prome基于文件的自动发现服务，这个可以不要，因为没在用prome
> - ADD：把jmx_javaagent-0.3.1.jar包放到/opt/...下，用来专门收集jvm的export，能提供一个http的接口
> - WORKDIR：工作目录
> - ADD：移动文件
> - CMD：运行文件

![1583559245639](assets/1583559245639.png)

完成



### 交付tomcat形式的dubbo服务消费者到K8S集群

改造下dubbo-demo-web项目

由于是tomcat，我们需要多建一条Jenkins流水线

![1583559296196](assets/1583559296196.png)

![1583559325853](assets/1583559325853.png)

![1583559348560](assets/1583559348560.png)

1

![1583559819045](assets/1583559819045.png)

2

![1583559830860](assets/1583559830860.png)

3

![1583559838755](assets/1583559838755.png)

4

![1583559908506](assets/1583559908506.png)

5

![1583559948318](assets/1583559948318.png)

6

![1583559958558](assets/1583559958558.png)

7

![1583559972282](assets/1583559972282.png)

8

![1583559985561](assets/1583559985561.png)

9

![1583560000469](assets/1583560000469.png)

10

![1583560009300](assets/1583560009300.png)

11

![1583560038370](assets/1583560038370.png)

~~~shell
# 将如下内容填入pipeline：
pipeline {
  agent any 
    stages {
    stage('pull') { //get project code from repo 
      steps {
        sh "git clone ${params.git_repo} ${params.app_name}/${env.BUILD_NUMBER} && cd ${params.app_name}/${env.BUILD_NUMBER} && git checkout ${params.git_ver}"
        }
    }
    stage('build') { //exec mvn cmd
      steps {
        sh "cd ${params.app_name}/${env.BUILD_NUMBER}  && /var/jenkins_home/maven-${params.maven}/bin/${params.mvn_cmd}"
      }
    }
    stage('unzip') { //unzip  target/*.war -c target/project_dir
      steps {
        sh "cd ${params.app_name}/${env.BUILD_NUMBER} && cd ${params.target_dir} && mkdir project_dir && unzip *.war -d ./project_dir"
      }
    }
    stage('image') { //build image and push to registry
      steps {
        writeFile file: "${params.app_name}/${env.BUILD_NUMBER}/Dockerfile", text: """FROM harbor.od.com/${params.base_image}
ADD ${params.target_dir}/project_dir /opt/tomcat/webapps/${params.root_url}"""
        sh "cd  ${params.app_name}/${env.BUILD_NUMBER} && docker build -t harbor.od.com/${params.image_name}:${params.git_ver}_${params.add_tag} . && docker push harbor.od.com/${params.image_name}:${params.git_ver}_${params.add_tag}"
      }
    }
  }
}
~~~

![1583560082441](assets/1583560082441.png)

save

点击构建

![1583560122560](assets/1583560122560.png)

~~~
# 填入指定参数，我的gittee是有tomcat的版本的，下面我依旧用的是gitlab
app_name:       dubbo-demo-web
image_name:     app/dubbo-demo-web
git_repo:       http://gitlab.od.com:10000/909336740/dubbo-demo-web.git
git_ver:        tomcat
add_tag:        20200214_1300
mvn_dir:        ./
target_dir:     ./dubbo-client/target
mvn_cmd:        mvn clean package -Dmaven.test.skip=true
base_image:     base/tomcat:v8.5.51
maven:          3.6.1-8u232
root_url:       ROOT
# 点击Build进行构建，等待构建完成
~~~

![1583561544404](assets/1583561544404.png)

build成功后



修改版本信息，删掉20880，如下图，然后update

![1583630633310](assets/1583630633310.png)

![1583630769979](assets/1583630769979.png)

浏览器输入[demo-test.od.com/hello?name=tomcat](demo-test.od.com/hello?name=tomcat)

![1583630984480](assets/1583630984480.png)

完成

查看dashboard里的pod里

![1583631035074](assets/1583631035074.png)

![1583631074816](assets/1583631074816.png)

> 这些就是我们要收集的日志，收到ELK



### 二进制安装部署elasticsearch

> 我们这里只部一个es的节点，因为我们主要是了解数据流的方式
>

[官网下载包](https://www.elastic.co/downloads/past-releases/elasticsearch-6-8-6)右键复制链接

![1581667412370](assets/1581667412370.png)

~~~
# 12机器：
~]# cd /opt/src/
src]# wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.8.6.tar.gz
src]# tar xfv elasticsearch-6.8.6.tar.gz -C /opt
src]# ln -s /opt/elasticsearch-6.8.6/ /opt/elasticsearch
src]# cd /opt/elasticsearch
# 配置
elasticsearch]# mkdir -p /data/elasticsearch/{data,logs}
# 修改以下内容
elasticsearch]# vi config/elasticsearch.yml
cluster.name: es.od.com
node.name: hdss7-12.host.com
path.data: /data/elasticsearch/data
path.logs: /data/elasticsearch/logs
bootstrap.memory_lock: true
network.host: 10.4.7.12
http.port: 9200

# 修改以下内容
elasticsearch]# vi config/jvm.options
-Xms512m
-Xmx512m

# 创建普通用户
elasticsearch]# useradd -s /bin/bash -M es
elasticsearch]# chown -R es.es /opt/elasticsearch-6.8.6/
elasticsearch]# chown -R es.es /data/elasticsearch/
# 文件描述符
elasticsearch]# vi /etc/security/limits.d/es.conf
es hard nofile 65536
es soft fsize unlimited
es hard memlock unlimited
es soft memlock unlimited

# 调整内核参数
elasticsearch]# sysctl -w vm.max_map_count=262144
elasticsearch]# echo "vm.max_map_count=262144" >> /etc/sysctl.conf
elasticsearch]# sysctl -p
# 启动
elasticsearch]# su -c "/opt/elasticsearch/bin/elasticsearch -d" es
elasticsearch]# netstat -luntp|grep 9200
# 调整ES日志模板
elasticsearch]# curl -H "Content-Type:application/json" -XPUT http://10.4.7.12:9200/_template/k8s -d '{
  "template" : "k8s*",
  "index_patterns": ["k8s*"],  
  "settings": {
    "number_of_shards": 5,
    "number_of_replicas": 0
  }
}'
~~~

![1583631650876](assets/1583631650876.png)

> 完成，你看我敲这么多遍就知道要等



### 安装部署kafka和kafka-manager

[官网](https://kafka.apache.org)

> 做kafka的时候不建议用超过2.2.0的版本
>

~~~
# 11机器：
cd /opt/src/
src]# wget https://archive.apache.org/dist/kafka/2.2.0/kafka_2.12-2.2.0.tgz
src]# tar xfv kafka_2.12-2.2.0.tgz -C /opt/
src]# ln -s /opt/kafka_2.12-2.2.0/ /opt/kafka
src]# cd /opt/kafka
kafka]# ll
~~~

![1583632474778](assets/1583632474778.png)

~~~
# 11机器，配置：
kafka]# mkdir -pv /data/kafka/logs
# 修改以下配置，其中zk是不变的，最下面两行则新增到尾部
kafka]# vi config/server.properties
log.dirs=/data/kafka/logs
zookeeper.connect=localhost:2181
log.flush.interval.messages=10000
log.flush.interval.ms=1000
delete.topic.enable=true
host.name=hdss7-11.host.com
~~~

![1583632595085](assets/1583632595085.png)

~~~
# 11机器，启动：
kafka]# bin/kafka-server-start.sh -daemon config/server.properties
kafka]# ps aux|grep kafka
kafka]# netstat -luntp|grep 80711
~~~

![1583632747263](assets/1583632747263.png)

![1583632764359](assets/1583632764359.png)

##### 部署kafka-manager

~~~
# 200机器，制作docker：
~]# mkdir /data/dockerfile/kafka-manager
~]# cd /data/dockerfile/kafka-manager
kafka-manager]# vi Dockerfile 
FROM hseeberger/scala-sbt

ENV ZK_HOSTS=10.4.7.11:2181 \
     KM_VERSION=2.0.0.2

RUN mkdir -p /tmp && \
    cd /tmp && \
    wget https://github.com/yahoo/kafka-manager/archive/${KM_VERSION}.tar.gz && \
    tar xxf ${KM_VERSION}.tar.gz && \
    cd /tmp/kafka-manager-${KM_VERSION} && \
    sbt clean dist && \
    unzip  -d / ./target/universal/kafka-manager-${KM_VERSION}.zip && \
    rm -fr /tmp/${KM_VERSION} /tmp/kafka-manager-${KM_VERSION}

WORKDIR /kafka-manager-${KM_VERSION}

EXPOSE 9000
ENTRYPOINT ["./bin/kafka-manager","-Dconfig.file=conf/application.conf"]

# 因为大，build过程比较慢，也比较容易失败，20分钟左右，
kafka-manager]# docker build . -t harbor.od.com/infra/kafka-manager:v2.0.0.2
# build一直失败就用我做好的，不跟你的机器也得是10.4.7.11等，因为dockerfile里面已经写死了
# kafka-manager]# docker pull 909336740/kafka-manager:v2.0.0.2
# kafka-manager]# docker tag 29badab5ea08 harbor.od.com/infra/kafka-manager:v2.0.0.2
kafka-manager]# docker images|grep kafka
kafka-manager]# docker push harbor.od.com/infra/kafka-manager:v2.0.0.2
~~~

![1583635072663](assets/1583635072663.png)

~~~
# 200机器，配置资源清单：
mkdir /data/k8s-yaml/kafka-manager
cd /data/k8s-yaml/kafka-manager
kafka-manager]# vi dp.yaml
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  name: kafka-manager
  namespace: infra
  labels: 
    name: kafka-manager
spec:
  replicas: 1
  selector:
    matchLabels: 
      app: kafka-manager
  strategy:
    type: RollingUpdate
    rollingUpdate: 
      maxUnavailable: 1
      maxSurge: 1
  revisionHistoryLimit: 7
  progressDeadlineSeconds: 600
  template:
    metadata:
      labels: 
        app: kafka-manager
    spec:
      containers:
      - name: kafka-manager
        image: harbor.od.com/infra/kafka-manager:v2.0.0.2
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 9000
          protocol: TCP
        env:
        - name: ZK_HOSTS
          value: zk1.od.com:2181
        - name: APPLICATION_SECRET
          value: letmein
      imagePullSecrets:
      - name: harbor
      terminationGracePeriodSeconds: 30
      securityContext: 
        runAsUser: 0

kafka-manager]# vi svc.yaml
kind: Service
apiVersion: v1
metadata: 
  name: kafka-manager
  namespace: infra
spec:
  ports:
  - protocol: TCP
    port: 9000
    targetPort: 9000
  selector: 
    app: kafka-manager

kafka-manager]# vi ingress.yaml
kind: Ingress
apiVersion: extensions/v1beta1
metadata: 
  name: kafka-manager
  namespace: infra
spec:
  rules:
  - host: km.od.com
    http:
      paths:
      - path: /
        backend: 
          serviceName: kafka-manager
          servicePort: 9000
~~~

![1583635130773](assets/1583635130773.png)

~~~
# 11机器，解析域名：
~]# vi /var/named/od.com.zone
serial 前滚一位

km                 A    10.4.7.10
~]# systemctl restart named
~]# dig -t A km.od.com @10.4.7.11 +short
# out:10.4.7.10
~~~

![1583635171353](assets/1583635171353.png)

~~~
# 22机器，应用资源：
~]# kubectl apply -f http://k8s-yaml.od.com/kafka-manager/dp.yaml
~]# kubectl apply -f http://k8s-yaml.od.com/kafka-manager/svc.yaml
~]# kubectl apply -f http://k8s-yaml.od.com/kafka-manager/ingress.yaml
~~~

![1583635645699](assets/1583635645699.png)

文件大可能起不来，需要多拉几次（当然你的资源配置高应该是没问题的）

![1581673449530](assets/1581673449530.png)

启动成功后浏览器输入[km.od.com](km.od.com)

![1583635710991](assets/1583635710991.png)

![1583635777584](assets/1583635777584.png)

填完上面三个值后就可以下拉save了

点击

![1583635809826](assets/1583635809826.png)

![1583635828160](assets/1583635828160.png)

![1583635897650](assets/1583635897650.png)

完成



### 制作filebeat底包并接入dubbo服务消费者

[Filebeat官网](https://www.elastic.co/downloads/beats/filebeat)

下载指纹

![1583636131189](assets/1583636131189.png)

打开后复制，后面的不需要复制

![1583636252938](assets/1583636252938.png)

开始前，请确保你的这些服务都是起来的

![1583636282215](assets/1583636282215.png)

~~~
# 200机器，准备镜像，资源配置清单：
mkdir /data/dockerfile/filebeat
~]# cd /data/dockerfile/filebeat
# 刚刚复制的指纹替代到下面的FILEBEAT_SHA1来，你用的是什么版本FILEBEAT_VERSION就用什么版本，更新的很快，我之前用的是5.1现在已经是6.1了
filebeat]# vi Dockerfile
FROM debian:jessie

ENV FILEBEAT_VERSION=7.6.1 \
    FILEBEAT_SHA1=887edb2ab255084ef96dbc4c7c047bfa92dad16f263e23c0fcc80120ea5aca90a3a7a44d4783ba37b135dac76618971272a591ab4a24997d8ad40c7bc23ffabf

RUN set -x && \
  apt-get update && \
  apt-get install -y wget && \
  wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-${FILEBEAT_VERSION}-linux-x86_64.tar.gz -O /opt/filebeat.tar.gz && \
  cd /opt && \
  echo "${FILEBEAT_SHA1}  filebeat.tar.gz" | sha512sum -c - && \
  tar xzvf filebeat.tar.gz && \
  cd filebeat-* && \
  cp filebeat /bin && \
  cd /opt && \
  rm -rf filebeat* && \
  apt-get purge -y wget && \
  apt-get autoremove -y && \
  apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

COPY docker-entrypoint.sh /
ENTRYPOINT ["/docker-entrypoint.sh"]

filebeat]# vi docker-entrypoint.sh
#!/bin/bash

ENV=${ENV:-"test"}
PROJ_NAME=${PROJ_NAME:-"no-define"}
MULTILINE=${MULTILINE:-"^\d{2}"}

cat > /etc/filebeat.yaml << EOF
filebeat.inputs:
- type: log
  fields_under_root: true
  fields:
    topic: logm-${PROJ_NAME}
  paths:
    - /logm/*.log
    - /logm/*/*.log
    - /logm/*/*/*.log
    - /logm/*/*/*/*.log
    - /logm/*/*/*/*/*.log
  scan_frequency: 120s
  max_bytes: 10485760
  multiline.pattern: '$MULTILINE'
  multiline.negate: true
  multiline.match: after
  multiline.max_lines: 100
- type: log
  fields_under_root: true
  fields:
    topic: logu-${PROJ_NAME}
  paths:
    - /logu/*.log
    - /logu/*/*.log
    - /logu/*/*/*.log
    - /logu/*/*/*/*.log
    - /logu/*/*/*/*/*.log
    - /logu/*/*/*/*/*/*.log
output.kafka:
  hosts: ["10.4.7.11:9092"]
  topic: k8s-fb-$ENV-%{[topic]}
  version: 2.0.0
  required_acks: 0
  max_message_bytes: 10485760
EOF

set -xe

# If user don't provide any command
# Run filebeat
if [[ "$1" == "" ]]; then
     exec filebeat  -c /etc/filebeat.yaml 
else
    # Else allow the user to run arbitrarily commands like bash
    exec "$@"
fi

filebeat]# chmod u+x docker-entrypoint.sh
filebeat]# docker build . -t harbor.od.com/infra/filebeat:v7.6.1
# build可能会失败很多次，我最长的是7次，下面有相关报错
filebeat]# docker images|grep filebeat
filebeat]# docker push harbor.od.com/infra/filebeat:v7.6.1
# 删掉原来的内容全部用新的，使用的两个镜像对应上你自己的镜像
filebeat]# vi /data/k8s-yaml/test/dubbo-demo-consumer/dp.yaml
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  name: dubbo-demo-consumer
  namespace: test
  labels: 
    name: dubbo-demo-consumer
spec:
  replicas: 1
  selector:
    matchLabels: 
      name: dubbo-demo-consumer
  template:
    metadata:
      labels: 
        app: dubbo-demo-consumer
        name: dubbo-demo-consumer
    spec:
      containers:
      - name: dubbo-demo-consumer
        image: harbor.od.com/app/dubbo-demo-web:tomcat_200307_1410
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8080
          protocol: TCP
        env:
        - name: C_OPTS
          value: -Denv=fat -Dapollo.meta=http://apollo-configservice:8080
        volumeMounts:
        - mountPath: /opt/tomcat/logs
          name: logm
      - name: filebeat
        image: harbor.od.com/infra/filebeat:v7.6.1
        imagePullPolicy: IfNotPresent
        env:
        - name: ENV
          value: test
        - name: PROJ_NAME
          value: dubbo-demo-web
        volumeMounts:
        - mountPath: /logm
          name: logm
      volumes:
      - emptyDir: {}
        name: logm
      imagePullSecrets:
      - name: harbor
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      securityContext: 
        runAsUser: 0
      schedulerName: default-scheduler
  strategy:
    type: RollingUpdate
    rollingUpdate: 
      maxUnavailable: 1
      maxSurge: 1
  revisionHistoryLimit: 7
  progressDeadlineSeconds: 600
~~~

> 相关报错（其它问题基本都是网络不稳定的问题）：![1583639164435](assets/1583639164435.png)
>
> 因为你用的指纹不是自己的，或者版本没写对。
>
> **dp.yaml文件解析**：    spec-containers下有两个name，对应的两个容器，这就是边车模式（sidecar）。

~~~
# 22机器，应用资源清单：
~]# kubectl apply -f http://k8s-yaml.od.com/test/dubbo-demo-consumer/dp.yaml
#out: deployment.extensions/dubbo-demo-consumer configured
~]# kubectl get pods -n test
~~~

![1583640124995](assets/1583640124995.png)

机器在21机器

![1583640217181](assets/1583640217181.png)

~~~
# 查看filebeat日志，21机器：
~]# docker ps -a|grep consumer
~]# docker exec -ti a6adcd6e83b3 bash
:/# cd /logm
:/#/logm# ls
:/#/logm# cd ..
# 这个log，是你每一次刷新demo页面都会有数据，你把它夯在这里
:/# tail -fn 200 /logm/stdout.log
# 日志就都在这里了
~~~

![1583640393605](assets/1583640393605.png)

~~~
# 浏览器输入：demo-test.com/hello?name=tomcat
~~~

![1583640365512](assets/1583640365512.png)

刷新上面的页面，去21机器看log

![1583640316947](assets/1583640316947.png)

[刷新km.od.com/clusters/kafka-od/topics](km.od.com/clusters/kafka-od/topics)

![1583640444719](assets/1583640444719.png)

完成



### 部署logstash镜像

~~~
# 200机器，准备镜像、资源清单：
# logstash的版本需要和es的版本一样，11机器cd /opt/目录下即可查看到
~]# docker pull logstash:6.8.6
~]# docker images|grep logstash
~]# docker tag d0a2dac51fcb harbor.od.com/infra/logstash:v6.8.6
~]# docker push harbor.od.com/infra/logstash:v6.8.6
~]# mkdir /etc/logstash
~]# vi /etc/logstash/logstash-test.conf
input {
  kafka {
    bootstrap_servers => "10.4.7.11:9092"
    client_id => "10.4.7.200"
    consumer_threads => 4
    group_id => "k8s_test"
    topics_pattern => "k8s-fb-test-.*"
  }
}

filter {
  json {
    source => "message"
  }
}

output {
  elasticsearch {
    hosts => ["10.4.7.12:9200"]
    index => "k8s-test-%{+YYYY.MM.DD}"
  }
}

~]# vi /etc/logstash/logstash-prod.conf
input {
  kafka {
    bootstrap_servers => "10.4.7.11:9092"
    client_id => "10.4.7.200"
    consumer_threads => 4
    group_id => "k8s_prod"
    topics_pattern => "k8s-fb-prod-.*"
  }
}

filter {
  json {
    source => "message"
  }
}

output {
  elasticsearch {
    hosts => ["10.4.7.12:9200"]
    index => "k8s-prod-%{+YYYY.MM.DD}"
  }
}

# 启动
~]# docker run -d --name logstash-test -v /etc/logstash:/etc/logstash harbor.od.com/infra/logstash:v6.8.6 -f /etc/logstash/logstash-test.conf
~]# docker ps -a|grep logstash
~~~

[^:%s/test/prod/g]: 在vi命令行输入左边内容，即可将文本里面得test全部换成prod

![1583651857160](assets/1583651857160.png)

我们刷新demo页面让kafka里面更新些日志

![1583651874243](assets/1583651874243.png)

有日志了

![1583651931546](assets/1583651931546.png)

~~~
# 200机器，验证ES索引（可能比较慢）：
~]# curl http://10.4.7.12:9200/_cat/indices?v
~~~

![1583652168302](assets/1583652168302.png)

> 这个反应有点慢，我等了快三分钟

完成



### 交付kibana到K8S集群

> 为什么用kibana：当然运维可以直接在dashboard里exec进去，然后命令行看情况，但是开发或者测试不行，那是机密的，我们得要一个页面供他们使用，使用需要kibana。

~~~
~]# docker pull kibana:6.8.6
~]# docker images|grep kibana
~]# docker tag adfab5632ef4 harbor.od.com/infra/kibana:v6.8.6
~]# docker push harbor.od.com/infra/kibana:v6.8.6
~]# mkdir /data/k8s-yaml/kibana
~]# cd /data/k8s-yaml/kibana/
kibana]# vi dp.yaml
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  name: kibana
  namespace: infra
  labels: 
    name: kibana
spec:
  replicas: 1
  selector:
    matchLabels: 
      name: kibana
  template:
    metadata:
      labels: 
        app: kibana
        name: kibana
    spec:
      containers:
      - name: kibana
        image: harbor.od.com/infra/kibana:v6.8.6
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 5601
          protocol: TCP
        env:
        - name: ELASTICSEARCH_URL
          value: http://10.4.7.12:9200
      imagePullSecrets:
      - name: harbor
      securityContext: 
        runAsUser: 0
  strategy:
    type: RollingUpdate
    rollingUpdate: 
      maxUnavailable: 1
      maxSurge: 1
  revisionHistoryLimit: 7
  progressDeadlineSeconds: 600

kibana]# vi svc.yaml
kind: Service
apiVersion: v1
metadata: 
  name: kibana
  namespace: infra
spec:
  ports:
  - protocol: TCP
    port: 5601
    targetPort: 5601
  selector: 
    app: kibana

kibana]# vi ingress.yaml
kind: Ingress
apiVersion: extensions/v1beta1
metadata: 
  name: kibana
  namespace: infra
spec:
  rules:
  - host: kibana.od.com
    http:
      paths:
      - path: /
        backend: 
          serviceName: kibana
          servicePort: 5601
~~~

![1583652777163](assets/1583652777163.png)

~~~
# 11机器，解析域名：
~]# vi /var/named/od.com.zone
serial 前滚一位
kibana             A    10.4.7.10

~]# systemctl restart named
~]# dig -t A kibana.od.com @10.4.7.11 +short
~~~

![1583652835822](assets/1583652835822.png)

~~~
# 22机器（21机器还夯着log），应用资源清单：
~]# kubectl apply -f http://k8s-yaml.od.com/kibana/dp.yaml
~]# kubectl apply -f http://k8s-yaml.od.com/kibana/svc.yaml
~]# kubectl apply -f http://k8s-yaml.od.com/kibana/ingress.yaml
~]# kubectl get pods -n infra
~~~

![1583653095099](assets/1583653095099.png)

[kibana.od.com](kibana.od.com)

> 我用的低配8C32G，机器快跑不动了，还会显示service not yet

![1581735655839](assets/1581735655839.png)

![1583653403755](assets/1583653403755.png)

> 点完可能会转圈转很久

![1583654044637](assets/1583654044637.png)

![1583654055784](assets/1583654055784.png)

去创建

![1583653710404](assets/1583653710404.png)

![1583653734639](assets/1583653734639.png)

![1583653767919](assets/1583653767919.png)

创建后，你就能看到日志

![1583654148342](assets/1583654148342.png)

把prod里的configservice和admin依次起来

![1583654217058](assets/1583654217058.png)

~~~
# 200机器：
cd /data/k8s-yaml/prod/dubbo-demo-consumer/
dubbo-demo-consumer]# cp ../../test/dubbo-demo-consumer/dp.yaml .
# y
# 修改namespace为prod，fat改成pro，http地址也改了
dubbo-demo-consumer]# vi dp.yaml
~~~

![1583654303335](assets/1583654303335.png)

![1583654391353](assets/1583654391353.png)

[config-prod.od.com](config-prod.od.com)

![1583654579273](assets/1583654579273.png)

完成



### 详解Kibana生产实践方法

查看环境情况

确认Eureka有config和admin

![1583654579273](assets/1583654579273.png)

确认Apollo里有两个环境

![1583654655665](assets/1583654655665.png)

确认完后，我们先把service起来

![1583654706752](assets/1583654706752.png)

然后到consumer，consumer需要接日志

~~~~
# 22机器：
~]# kubectl apply -f http://k8s-yaml.od.com/prod/dubbo-demo-consumer/dp.yaml
~]# kubectl get pods -n prod
~~~~

![1583654764653](assets/1583654764653.png)

~~~
# 200机器：
# 启动
~]# docker run -d --name logstash-prod -v /etc/logstash:/etc/logstash harbor.od.com/infra/logstash:v6.8.6 -f /etc/logstash/logstash-prod.conf
~]# docker ps -a|grep logstash
# curl一下，这时候还只有test
~]# curl http://10.4.7.12:9200/_cat/indices?v
~~~

![1583654862812](assets/1583654862812.png)

~~~
# 访问浏览器demo-prod.od.com/hello?name=prod
~~~

![1583654914493](assets/1583654914493.png)

我们看一下调度到哪个节点了

![1583654962325](assets/1583654962325.png)

~~~
# 调度到21节点，我们去21节点看一下：
~]# docker ps -a|grep consumer
~]# docker exec -ti 094e68c795b0 bash
:/# cd /logm
:/logm# ls
:/logm# tail -fn 200 stdout.log
~~~

![1583655033176](assets/1583655033176.png)

夯住

![1583655055686](assets/1583655055686.png)

去[http://km.od.com](http://km.od.com/)

![1583655088618](assets/1583655088618.png)

![1583655114357](assets/1583655114357.png)

![1583655130583](assets/1583655130583.png)

已经有prod了

~~~
# 200机器，curl的时候可能要等一下才有（可以去多刷一下网页产生日志）：
~]# curl http://10.4.7.12:9200/_cat/indices?v
~~~

![1583655264326](assets/1583655264326.png)

去kibana配一下

![1583655369240](assets/1583655369240.png)

![1583655383868](assets/1583655383868.png)

##### 如何使用kibana

时间选择

![1583655477179](assets/1583655477179.png)

![1583655509551](assets/1583655509551.png)

> test没用数据的点下这个就有了，平常用的最多的也是today，后面突然没数据了你就可以刷新或者点时间，特别是配置差的同学
>
> ![1583655706547](assets/1583655706547.png)

环境选择器

![1583655530032](assets/1583655530032.png)

关键字选择器

先把message顶上来，还有log.file.path、hostname

![1583655759010](assets/1583655759010.png)

我们先制造一些错误，把service scale成0

![1583655838132](assets/1583655838132.png)

然后刷新一下页面，让它报错，记得是test环境

![1583655858792](assets/1583655858792.png)

搜exception关键字，并可展开

![1583655981692](assets/1583655981692.png)

![1583656009350](assets/1583656009350.png)

现在consumer日志已经完成了，记得把service的pod还原，并删掉consumer的pod让它重启



#### 课外作业（不是一定要完成，但是你做了我做的这些）

consumer日志已经完成，还可以做service日志

~~~
# 200机器：
# 修改一下内容
~]# cd  /data/dockerfile/jre8/
# 修改以下内容
jre8]# vi entrypoint.sh
exec java -jar ${M_OPTS} ${C_OPTS} ${JAR_BALL} 2>&1 >> /opt/logs/stdout.log

jre8]# docker build . -t harbor.od.com/base/jre8:8u112_with_logs
jre8]# docker push harbor.od.com/base/jre8:8u112_with_logs
~~~

![1584067987917](assets/1584067987917.png)

![1581752521087](assets/1581752521087.png)

去修改一下Jenkins加一个底包

![1584237876688](assets/1584237876688.png)

![1584237900572](assets/1584237900572.png)

![1584237947516](assets/1584237947516.png)

下面就要你接着做了

