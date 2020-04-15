## Kubernetes调度机制

### Kubernetes的资源模型与资源管理

所有跟调度和资源管理相关的属性都是属于 Pod 对象的字段，其中最重要的部分，就是 Pod 的 CPU 和内存配置，如下所示：

~~~
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: db
    image: mysql
    env:
    - name: MYSQL_ROOT_PASSWORD
      value: "password"
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
  - name: wp
    image: wordpress
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
~~~

> 在Kubernetes 中：
>
> - CPU称为 “可压缩资源” ，可压缩资源不足时，Pod只会“饥饿”，不会退出
> - 内存称为“不可压缩资源”，不可压缩资源不足时，Pod会因为OOM（Out-Of-Memory）被内核杀掉
> -  Pod 可以由多个 Container 组成，所以 CPU 和内存资源的限额，是要配置在每个 Container 的定义上的，而Pod整体资源配置，就由这些 Container 的配置值累加得到

**limits 和 requests** 的区别很简单：在调度的时候，kube-scheduler 只会按照 requests 的值进行计算。而在真正设置 Cgroups 限制的时候，kubelet 则会按照 limits 的值来进行设置。

requests+limits 的做法，其实是Borg的思想，即，实际场景中，大多数作业使用到的资源其实远小宇它所请求的资源限额，基于这种假设，用户可以声明一个相对小的requests值供调度器使用，而Kubernetes 真正设置给容器 Cgroups 的，则是相对较大的 limits 值。

#### QoS 模型：

> 三个级别，衔接上面

- **Guaranteed：**Pod 里的每一个 Container 都同时设置了 requests 和 limits，并且 requests 和 limits 值相等的时候，这个 Pod 就属于 Guaranteed 类别

~~~
apiVersion: v1
kind: Pod
metadata:
  name: qos-demo
  namespace: qos-example
spec:
  containers:
  - name: qos-demo-ctr
    image: nginx
    resources:
      limits:
        memory: "200Mi"
        cpu: "700m"
      requests:
        memory: "200Mi"
        cpu: "700m"
~~~

- **Burstable：**当 Pod 不满足 Guaranteed 的条件，但至少有一个 Container 设置了 requests。那么这个 Pod 就会被划分到 Burstable 类别

~~~
apiVersion: v1
kind: Pod
metadata:
  name: qos-demo-2
  namespace: qos-example
spec:
  containers:
  - name: qos-demo-2-ctr
    image: nginx
    resources:
      limits
        memory: "200Mi"
      requests:
        memory: "100Mi"
~~~

- **BestEffort：**如果 Pod 既没有设置 requests，也没有设置 limits，那么它的 QoS 类别就是 BestEffort

~~~
apiVersion: v1
kind: Pod
metadata:
  name: qos-demo-3
  namespace: qos-example
spec:
  containers:
  - name: qos-demo-3-ctr
    image: nginx
~~~

**QoS 划分的主要应用场景，是当宿主机资源紧张的时候，kubelet 对 Pod 进行 Eviction（即资源回收）时需要用到的。**当Eviction发生时，kubelet 具体会挑 Pod 进行删除操作，按如下级别：

**BestEffort < Burstable < Guaranteed**

并且，Kubernetes 会保证只有当 Guaranteed 类别的 Pod 的资源使用量超过了其 limits 的限制，或者宿主机本身正处于 Memory Pressure 状态时，Guaranteed 的 Pod 才可能被选中进行 Eviction 操作。

#### cpuset 的设置

> 一个实际生产中非常有用的特性，衔接上面
>
> 在使用容器时，可以通过设置 cpuset 把容器绑定到某个 CPU 的核上，而不是像 cpushare 那样共享 CPU 的计算能力，这样CPU之间进行上下文切换的次数大大减少，容器里应用的性能会得到大幅提升

实现方法：

-  Pod 必须是 Guaranteed 的 QoS 类型
- 将 Pod 的 CPU 资源的 requests 和 limits 设置为同一个相等的整数值

如下例子：

~~~
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      limits:
        memory: "200Mi"
        cpu: "2"
      requests:
        memory: "200Mi"
        cpu: "2"
~~~

这样，该Pod就会绑定在2个独占的CPU核上，具体是哪两个CPU，由kubelet 分配

基于上面的情况，建议将DaemonSet（亦或者类似的）的 Pod 都设置为 Guaranteed 的 QoS 类型，否则一旦被资源紧张被回收，又立即会在宿主机上重建出来，这样资源回收的动作就没有意义了