## 第二章——kubernetes基础

### 初识Pod

> **WHAT：**它只是一个逻辑概念、是一种编排思想、k8s中最小编排单位，k8s处理的还是宿主机上Linux的Namespace和Cgrous
>
> **WHY：**
>
> - 一些容器更适合放在一起紧密协作
> - 容器的日志收集

**Pod 里的所有容器，共享的是同一个 Network Namespace，并且可以声明共享同一个 Volume。**

对与上面的容器的日志收集，举例：有一个应用，需要不断地把日志文件输出到容器的 /var/log 目录，这时我们把一个 Pod 里的 Volume 挂载到应用容器的 /var/log 目录上。然后在这个Pod里运行一个 sidecar 容器，也声明挂载同一个 Volume 到自己的 /var/log 目录上。sidecar 容器就只需要做一件事儿，就是不断地从自己的 /var/log 目录里读取日志文件，转发到 MongoDB 或者 Elasticsearch 中存储起来。一个最基本的日志收集工作就完成了。

**实际工作中：**当你需要把一个运行在虚拟机里的应用迁移到 Docker 容器中时，一定要仔细分析到底有哪些进程（组件）运行在这个虚拟机里。

然后，你就可以把整个虚拟机想象成为一个 Pod，把这些进程分别做成容器镜像，把有顺序关系的容器，定义为 Init Container。这才是更加合理的、松耦合的容器编排诀窍，也是从传统应用架构，到“微服务架构”最自然的过渡方式。



### Pod中几个重要字段的含义和用法

**凡是调度、网络、存储，以及安全相关的属性，基本上是 Pod 级别的。**

**HostAliases：**定义了 Pod 的 hosts 文件（比如 /etc/hosts）里的内容，用法如下：

~~~
apiVersion: v1
kind: Pod
...
spec:
  hostAliases:
  - ip: "10.1.2.3"
    hostnames:
    - "foo.remote"
    - "bar.remote"
...
~~~

**shareProcessNamespace=true：**在这个 Pod 里的容器共享 PID Namespace

~~~
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  shareProcessNamespace: true
  containers:
  - name: nginx
    image: nginx
  - name: shell
    image: busybox
    stdin: true
    tty: tru
~~~

上面的YAML文件中，还定义了两个容器：一个是 nginx 容器，一个是开启了 tty 和 stdin 的 shell 容器。在 Pod 的 YAML 文件里声明开启它们俩，其实等同于设置了 docker run 里的 -it（-i 即 stdin，-t 即 tty）参数。

> **tty：**Linux 给用户提供的一个常驻小程序，用于接收用户的标准输入，返回操作系统的标准输出
>
> **stdin：**为了能够在 tty 中输入信息，还需要同时开启 stdin（标准输入流）。

这个 Pod 被创建后，你就可以使用 shell 容器的 tty 跟这个容器进行交互了。

**容器要共享宿主机的 Namespace，也一定是 Pod 级别的定义**

~~~
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  hostNetwork: true
  hostIPC: true
  hostPID: true
  containers:
  - name: nginx
    image: nginx
  - name: shell
    image: busybox
    stdin: true
    tty: true
~~~

在这个 Pod 中，定义了共享宿主机的 Network、IPC 和 PID Namespace。这就意味着，这个 Pod 里的所有容器，会直接使用宿主机的网络、直接与宿主机进行 IPC 通信、看到宿主机里正在运行的所有进程。

#### Container是Pod中最重要的字段

- **ImagePullPolicy：**定义了镜像拉取的策略

  - 默认是 Always，即每次创建 Pod 都重新拉取一次镜像。
  - 可以定义为 Never 或者 IfNotPresent，则意味着 Pod 永远不会主动拉取这个镜像，或者只在宿主机上不存在这个镜像时才拉取。

- **Lifecycle：**定义的是 Container Lifecycle Hooks。在容器状态发生变化时触发一系列“钩子”。如下例子：

  ~~~
  apiVersion: v1
  kind: Pod
  metadata:
    name: lifecycle-demo
  spec:
    containers:
    - name: lifecycle-demo-container
      image: nginx
      lifecycle:
        postStart:
          exec:
            command: ["/bin/sh", "-c", "echo Hello from the postStart handler > /usr/share/message"]
        preStop:
          exec:
            command: ["/usr/sbin/nginx","-s","quit"]
  ~~~

  > **postStart ：**在容器启动后，立刻执行一个指定的操作。
  >
  > - 该操作虽然是在 Docker 容器 ENTRYPOINT 执行之后，但它并不严格保证顺序。也就是说，在 postStart 启动时，ENTRYPOINT 有可能还没有结束。
  > - 执行超时或者错误，Kubernetes 会在该 Pod 的 Events 中报出该容器启动失败的错误信息，导致 Pod 也处于失败的状态。
  >
  > **preStop：**preStop 发生的时机，则是容器被杀死之前（比如，收到了 SIGKILL 信号）。preStop 操作的执行，是**同步**的，它会阻塞当前的容器杀死流程，直到这个 Hook 定义操作完成之后，才允许容器被杀死 



### Pod的几种状态

1. **Pending：**这个状态意味着，Pod 的 YAML 文件已经提交给了 Kubernetes，API 对象已经被创建并保存在 Etcd 当中。但是，这个 Pod 里有些容器因为某种原因而不能被顺利创建。比如，调度不成功。
2. **Running：**这个状态下，Pod 已经调度成功，跟一个具体的节点绑定。它包含的容器都已经创建成功，并且至少有一个正在运行中。
3. **Succeeded：**这个状态意味着，Pod 里的所有容器都正常运行完毕，并且已经退出了。这种情况在运行一次性任务时最为常见。
4. **Failed：**这个状态下，Pod 里至少有一个容器以不正常的状态（非 0 的返回码）退出。这个状态的出现，意味着你得想办法 Debug 这个容器的应用，比如查看 Pod 的 Events 和日志。
5. **Unknown：**这是一个异常状态，意味着 Pod 的状态不能持续地被 kubelet 汇报给 kube-apiserver，这很有可能是主从节点（Master 和 Kubelet）间的通信出现了问题。

### kubernetes技能图谱

![kubernetes技能图谱](assets/kubernetes技能图谱.jpg)