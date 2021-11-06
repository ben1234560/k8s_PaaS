## 第八章——spinnaker部署与应用

> 从本章节开始，很多事情我不再截图而是一笔带过，以便你更好的动手操作，不过，一笔带过的地方我一定不会留大坑，这个你可以完全放心

#### 关于IaaS、PaaS、SaaS

![1581819941099](assets/1581819941099.png)

> K8S不是传统意义上的Paas平台，而很多互联网公司都需要的是Paas平台，而不是单纯的K8S，K8S及其周边生态（如logstash、Prometheus等）才是Paas平台，才是公司需要的

#### 获得PaaS能力的几个必要条件：

- 统一应有的运行时环境（docker）
- 有IaaS能力（K8S）
- 有可靠的中间件集群、数据库集群（DBA的主要工作）
- 有分布式存储集群（存储工程师的主要工作）
- 有适配的监控、日志系统（Prometheus、ELK）
- 有完善的CI、CD系统（Jenkins、Spinnaker）

> 阿里云、腾讯云等厂商都提供了K8S为底的服务，即你买了集群就给你配备了K8S，但我们不能完全依赖于厂商，而被钳制，同时我们也需要不断的学习以备更好的理解和使用，公司越大时越需要自己创建而不是依赖于厂商。

#### spinnaker简介

> **WHAT**：通过灵活和可配置 Pipelines，实现可重复的自动化部署；提供所有环境的全局视图，可随时查看应用程序在其部署 Pipeline 的状态；易于配置、维护和扩展；等等；

主要功能

- 集群管理：主要用于管理云资源，即主要是IaaS的资源
- 部落管理：负责将Jenkins流水线创建的镜像，部署到K8S集群中去，让服务真正运行起来。
- 架构：[官网地址](https://www.spinnaker.io/reference/architecture/)

![1581822996520](assets/1581822996520.png)

> Deck：点点点页面
>
> Gate：网关
>
> Igor：用来和Jenkins通信
>
> Echo：信息通讯组件
>
> Orca：任务编排引擎
>
> Clouddriver：云计算基础设施
>
> Front50：管理持久化数据
>
> 其中我们用到redis、minio
>
> 部署顺序：Minio-->Redis-->Clouddriver-->Front50-->Orca-->Echo-->Igor-->Gate-->Deck-->Nginx(是静态页面所以需要)

### 部署Spinnaker的Amory发行版

~~~
# 200机器，准备镜像、在资源清单：
~]# docker pull minio/minio:latest
~]# docker images|grep minio
~]# docker tag 7ea4a619ecfc harbor.od.com/armory/minio:latest
# 在此之前你应该创建一个私有armory
# 否则报错：denied: requested access to the resource is denied
~]# docker push harbor.od.com/armory/minio:latest
~]# mkdir -p /data/k8s-yaml/armory/minio
~]# cd /data/k8s-yaml/armory/minio/
minio]# vi dp.yaml
kind: Deployment
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    name: minio
  name: minio
  namespace: armory
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 7
  selector:
    matchLabels:
      name: minio
  template:
    metadata:
      labels:
        app: minio
        name: minio
    spec:
      containers:
      - name: minio
        image: harbor.od.com/armory/minio:latest
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 9000
          protocol: TCP
        args:
        - server
        - /data
        env:
        - name: MINIO_ACCESS_KEY
          value: admin
        - name: MINIO_SECRET_KEY
          value: admin123
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /minio/health/ready
            port: 9000
            scheme: HTTP
          initialDelaySeconds: 10
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 5
        volumeMounts:
        - mountPath: /data
          name: data
      imagePullSecrets:
      - name: harbor
      volumes:
      - nfs:
          server: hdss7-200
          path: /data/nfs-volume/minio
        name: data

minio]# vi svc.yaml 
apiVersion: v1
kind: Service
metadata:
  name: minio
  namespace: armory
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 9000
  selector:
    app: minio

minio]# vi ingress.yaml 
kind: Ingress
apiVersion: extensions/v1beta1
metadata: 
  name: minio
  namespace: armory
spec:
  rules:
  - host: minio.od.com
    http:
      paths:
      - path: /
        backend: 
          serviceName: minio
          servicePort: 80

# 创建对应的存储
minio]# mkdir /data/nfs-volume/minio
~~~



~~~
# 11机器，解析域名：
vi /var/named/od.com.zone
serial 前滚一位
minio              A    10.4.7.10

systemctl restart named
dig -t A minio.od.com +short
~~~

> 为什么每次每次都不用些od.com，是因为第一行有$ORIGIN od.com. 的宏指令，会自动补

~~~
# 22机器：
# 创建名称空间
~]# kubectl create ns armory
# 因为armory仓库是私有的，要创建secret，不然拉不了
~]# kubectl create secret docker-registry harbor --docker-server=harbor.od.com --docker-username=admin --docker-password=Harbor12345 -n armory
~~~

在amory名称空间里面就能看到

![1583994197390](assets/1583994197390.png)

~~~
# 22机器，应用清单:
~]# kubectl apply -f http://k8s-yaml.od.com/armory/minio/dp.yaml
~]# kubectl apply -f http://k8s-yaml.od.com/armory/minio/svc.yaml
~]# kubectl apply -f http://k8s-yaml.od.com/armory/minio/ingress.yaml
~~~

![1583994262819](assets/1583994262819.png)

[minio.od.com](minio.od.com)

账户：admin

密码：admin123

![1583994292679](assets/1583994292679.png)

完成



### 安装部署redis

~~~
# 200机器，准备镜像、资源清单：
~]# docker pull redis:4.0.14
~]# docker images|grep redis
~]# docker tag 6e221e67453d harbor.od.com/armory/redis:v4.0.14
~]# docker push !$
~]# mkdir /data/k8s-yaml/armory/redis
~]# cd /data/k8s-yaml/armory/redis
redis]# vi dp.yaml
kind: Deployment
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    name: redis
  name: redis
  namespace: armory
spec:
  replicas: 1
  revisionHistoryLimit: 7
  selector:
    matchLabels:
      name: redis
  template:
    metadata:
      labels:
        app: redis
        name: redis
    spec:
      containers:
      - name: redis
        image: harbor.od.com/armory/redis:v4.0.14
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 6379
          protocol: TCP
      imagePullSecrets:
      - name: harbor

redis]# vi svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: redis
  namespace: armory
spec:
  ports:
  - port: 6379
    protocol: TCP
    targetPort: 6379
  selector:
    app: redis

~~~



~~~
# 22机器，应用资源清单：
~]# kubectl apply -f http://k8s-yaml.od.com/armory/redis/dp.yaml
~]# kubectl apply -f http://k8s-yaml.od.com/armory/redis/svc.yaml
~~~

> 去看pod已经起来了
>

~~~
# 看pod的ip起在哪里了，我是172.7.22.13，22机器，确认redis的服务起来了没：
~]# docker ps -a|grep redis
~]# telnet 172.7.22.13 6379
~~~

![1583994842832](assets/1583994842832.png)

完成



### 安装部署clouddriver

~~~
# 200机器：
~]# docker pull docker.io/armory/spinnaker-clouddriver-slim:release-1.8.x-14c9664
~]# docker images|grep clouddriver
~]# docker tag edb2507fdb62 harbor.od.com/armory/clouddriver:v1.8.x
~]# docker push harbor.od.com/armory/clouddriver:v1.8.x
~]# mkdir /data/k8s-yaml/armory/clouddriver
~]# cd /data/k8s-yaml/armory/clouddriver/
clouddriver] vi credentials
[default]
aws_access_key_id=admin
aws_secret_access_key=admin123

# 做证书
~]# cd /opt/certs/
~]# cp client-csr.json admin-csr.json
# 修改一下内容
~]# vi admin-csr.json
    "CN": "cluster-admin"

~]# cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=client admin-csr.json | cfssl-json -bare admin
~~~

![1584005779485](assets/1584005779485.png)

~~~
# 21机器：
~]# wget wget http://k8s-yaml.od.com/armory/clouddriver/credentials
~]# kubectl create secret generic credentials --from-file=./credentials -n armory
~]# scp hdss7-200:/opt/certs/ca.pem .
~]# scp hdss7-200:/opt/certs/admin.pem .
~]# scp hdss7-200:/opt/certs/admin-key.pem .

~]# kubectl config set-cluster myk8s --certificate-authority=./ca.pem --embed-certs=true --server=https://10.4.7.10:7443 --kubeconfig=config
~]# kubectl config set-credentials cluster-admin --client-certificate=./admin.pem --client-key=./admin-key.pem --embed-certs=true --kubeconfig=config
~]# kubectl config set-context myk8s-context --cluster=myk8s --user=cluster-admin --kubeconfig=config
~]# kubectl config use-context myk8s-context --kubeconfig=config
kubectl create clusterrolebinding myk8s-admin --clusterrole=cluster-admin --user=cluster-admin
~]# cd /root/.kube/
.kube]# cp /root/config .
.kube]# kubectl config view
.kube]# kubectl get pods -n armory
~~~

![1584005992328](assets/1584005992328.png)

~~~
# 200机器：
~]# mkdir /root/.kube
~]# cd /root/.kube
.kube]# scp hdss7-21:/root/config .
.kube]# cd 
~]# scp hdss7-21:/opt/kubernetes/server/bin/kubectl .
~]# mv kubectl /usr/bin/
~]# kubectl config view
~]# kubectl get pods -n infra
~~~

![1584006160054](assets/1584006160054.png)

~~~
# 21机器，给权限：
.kube]# mv config default-kubeconfig
.kube]# kubectl create configmap default-kubeconfig --from-file=./default-kubeconfig -n armory
~~~

![1584014708599](assets/1584014708599.png)

~~~shell
# 200机器：
~]# cd /data/k8s-yaml/armory/clouddriver/
clouddriver]# vi init-env.yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: init-env
  namespace: armory
data:
  API_HOST: http://spinnaker.od.com/api
  ARMORY_ID: c02f0781-92f5-4e80-86db-0ba8fe7b8544
  ARMORYSPINNAKER_CONF_STORE_BUCKET: armory-platform
  ARMORYSPINNAKER_CONF_STORE_PREFIX: front50
  ARMORYSPINNAKER_GCS_ENABLED: "false"
  ARMORYSPINNAKER_S3_ENABLED: "true"
  AUTH_ENABLED: "false"
  AWS_REGION: us-east-1
  BASE_IP: 127.0.0.1
  CLOUDDRIVER_OPTS: -Dspring.profiles.active=armory,configurator,local
  CONFIGURATOR_ENABLED: "false"
  DECK_HOST: http://spinnaker.od.com
  ECHO_OPTS: -Dspring.profiles.active=armory,configurator,local
  GATE_OPTS: -Dspring.profiles.active=armory,configurator,local
  IGOR_OPTS: -Dspring.profiles.active=armory,configurator,local
  PLATFORM_ARCHITECTURE: k8s
  REDIS_HOST: redis://redis:6379
  SERVER_ADDRESS: 0.0.0.0
  SPINNAKER_AWS_DEFAULT_REGION: us-east-1
  SPINNAKER_AWS_ENABLED: "false"
  SPINNAKER_CONFIG_DIR: /home/spinnaker/config
  SPINNAKER_GOOGLE_PROJECT_CREDENTIALS_PATH: ""
  SPINNAKER_HOME: /home/spinnaker
  SPRING_PROFILES_ACTIVE: armory,configurator,local

clouddriver]# vi default-config.yaml
# 这里的内容在另外放的default-config.yaml里，因为实在太大，所以没办法复制进来

clouddriver]# vi custom-config.yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: custom-config
  namespace: armory
data:
  clouddriver-local.yml: |
    kubernetes:
      enabled: true
      accounts:
        - name: cluster-admin
          serviceAccount: false
          dockerRegistries:
            - accountName: harbor
              namespace: []
          namespaces:
            - test
            - prod
          kubeconfigFile: /opt/spinnaker/credentials/custom/default-kubeconfig
      primaryAccount: cluster-admin
    dockerRegistry:
      enabled: true
      accounts:
        - name: harbor
          requiredGroupMembership: []
          providerVersion: V1
          insecureRegistry: true
          address: http://harbor.od.com
          username: admin
          password: Harbor12345
      primaryAccount: harbor
    artifacts:
      s3:
        enabled: true
        accounts:
        - name: armory-config-s3-account
          apiEndpoint: http://minio
          apiRegion: us-east-1
      gcs:
        enabled: false
        accounts:
        - name: armory-config-gcs-account
  custom-config.json: ""
  echo-configurator.yml: |
    diagnostics:
      enabled: true
  front50-local.yml: |
    spinnaker:
      s3:
        endpoint: http://minio
  igor-local.yml: |
    jenkins:
      enabled: true
      masters:
        - name: jenkins-admin
          address: http://jenkins.od.com
          username: admin
          password: admin123
      primaryAccount: jenkins-admin
  nginx.conf: |
    gzip on;
    gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript application/vnd.ms-fontobject application/x-font-ttf font/opentype image/svg+xml image/x-icon;

    server {
           listen 80;

           location / {
                proxy_pass http://armory-deck/;
           }

           location /api/ {
                proxy_pass http://armory-gate:8084/;
           }

           rewrite ^/login(.*)$ /api/login$1 last;
           rewrite ^/auth(.*)$ /api/auth$1 last;
    }
  spinnaker-local.yml: |
    services:
      igor:
        enabled: true

clouddriver]# vi dp.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    app: armory-clouddriver
  name: armory-clouddriver
  namespace: armory
spec:
  replicas: 1
  revisionHistoryLimit: 7
  selector:
    matchLabels:
      app: armory-clouddriver
  template:
    metadata:
      annotations:
        artifact.spinnaker.io/location: '"armory"'
        artifact.spinnaker.io/name: '"armory-clouddriver"'
        artifact.spinnaker.io/type: '"kubernetes/deployment"'
        moniker.spinnaker.io/application: '"armory"'
        moniker.spinnaker.io/cluster: '"clouddriver"'
      labels:
        app: armory-clouddriver
    spec:
      containers:
      - name: armory-clouddriver
        image: harbor.od.com/armory/clouddriver:v1.8.x
        imagePullPolicy: IfNotPresent
        command:
        - bash
        - -c
        args:
        - bash /opt/spinnaker/config/default/fetch.sh && cd /home/spinnaker/config
          && /opt/clouddriver/bin/clouddriver
        ports:
        - containerPort: 7002
          protocol: TCP
        env:
        - name: JAVA_OPTS
          value: -Xmx512M
        envFrom:
        - configMapRef:
            name: init-env
        livenessProbe:
          failureThreshold: 5
          httpGet:
            path: /health
            port: 7002
            scheme: HTTP
          initialDelaySeconds: 600
          periodSeconds: 3
          successThreshold: 1
          timeoutSeconds: 1
        readinessProbe:
          failureThreshold: 5
          httpGet:
            path: /health
            port: 7002
            scheme: HTTP
          initialDelaySeconds: 180
          periodSeconds: 3
          successThreshold: 5
          timeoutSeconds: 1
        securityContext: 
          runAsUser: 0
        volumeMounts:
        - mountPath: /etc/podinfo
          name: podinfo
        - mountPath: /home/spinnaker/.aws
          name: credentials
        - mountPath: /opt/spinnaker/credentials/custom
          name: default-kubeconfig
        - mountPath: /opt/spinnaker/config/default
          name: default-config
        - mountPath: /opt/spinnaker/config/custom
          name: custom-config
      imagePullSecrets:
      - name: harbor
      volumes:
      - configMap:
          defaultMode: 420
          name: default-kubeconfig
        name: default-kubeconfig
      - configMap:
          defaultMode: 420
          name: custom-config
        name: custom-config
      - configMap:
          defaultMode: 420
          name: default-config
        name: default-config
      - name: credentials
        secret:
          defaultMode: 420
          secretName: credentials
      - downwardAPI:
          defaultMode: 420
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.labels
            path: labels
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.annotations
            path: annotations
        name: podinfo

clouddriver]# vi svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: armory-clouddriver
  namespace: armory
spec:
  ports:
  - port: 7002
    protocol: TCP
    targetPort: 7002
  selector:
    app: armory-clouddriver

clouddriver]# kubectl apply -f ./init-env.yaml
clouddriver]# kubectl apply -f ./default-config.yaml
clouddriver]# kubectl apply -f ./custom-config.yaml
clouddriver]# kubectl apply -f ./dp.yaml
clouddriver]# kubectl apply -f ./svc.yaml
~~~

![1584014649148](assets/1584014649148.png)

![1584015398060](assets/1584015398060.png)

~~~
# 21机器(因为我的minio在21机器)：
.kube]# docker ps -a|grep minio
.kube]# docker exec -it 9f5592fb2950 /bin/sh
/ # curl armory-clouddriver:7002/health
~~~

![1584015549913](assets/1584015549913.png)

完成



### 安装部署spinnaker其余组件

部署数据持久化组件——Front50

~~~
# 200机器：
~]# docker pull docker.io/armory/spinnaker-front50-slim:release-1.8.x-93febf2
~]# docker images|grep front50
~]# docker tag 0d353788f4f2 harbor.od.com/armory/front50:v1.8.x
~]# docker push harbor.od.com/armory/front50:v1.8.x
~]# mkdir /data/k8s-yaml/armory/front50
~]# cd /data/k8s-yaml/armory/front50/
front50]# vi dp.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    app: armory-front50
  name: armory-front50
  namespace: armory
spec:
  replicas: 1
  revisionHistoryLimit: 7
  selector:
    matchLabels:
      app: armory-front50
  template:
    metadata:
      annotations:
        artifact.spinnaker.io/location: '"armory"'
        artifact.spinnaker.io/name: '"armory-front50"'
        artifact.spinnaker.io/type: '"kubernetes/deployment"'
        moniker.spinnaker.io/application: '"armory"'
        moniker.spinnaker.io/cluster: '"front50"'
      labels:
        app: armory-front50
    spec:
      containers:
      - name: armory-front50
        image: harbor.od.com/armory/front50:v1.8.x
        imagePullPolicy: IfNotPresent
        command:
        - bash
        - -c
        args:
        - bash /opt/spinnaker/config/default/fetch.sh && cd /home/spinnaker/config
          && /opt/front50/bin/front50
        ports:
        - containerPort: 8080
          protocol: TCP
        env:
        - name: JAVA_OPTS
          value: -javaagent:/opt/front50/lib/jamm-0.2.5.jar -Xmx1000M
        envFrom:
        - configMapRef:
            name: init-env
        livenessProbe:
          failureThreshold: 3
          httpGet:
            path: /health
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 600
          periodSeconds: 3
          successThreshold: 1
          timeoutSeconds: 1
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /health
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 180
          periodSeconds: 5
          successThreshold: 8
          timeoutSeconds: 1
        volumeMounts:
        - mountPath: /etc/podinfo
          name: podinfo
        - mountPath: /home/spinnaker/.aws
          name: credentials
        - mountPath: /opt/spinnaker/config/default
          name: default-config
        - mountPath: /opt/spinnaker/config/custom
          name: custom-config
      imagePullSecrets:
      - name: harbor
      volumes:
      - configMap:
          defaultMode: 420
          name: custom-config
        name: custom-config
      - configMap:
          defaultMode: 420
          name: default-config
        name: default-config
      - name: credentials
        secret:
          defaultMode: 420
          secretName: credentials
      - downwardAPI:
          defaultMode: 420
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.labels
            path: labels
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.annotations
            path: annotations
        name: podinfo

front50]# vi svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: armory-front50
  namespace: armory
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: armory-front50

front50]# kubectl apply -f ./dp.yaml
front50]# kubectl apply -f ./svc.yaml
~~~

![1584023207016](assets/1584023207016.png)

[minio.od.com](minio.od.com)

![1584023230603](assets/1584023230603.png)

#### 部署任务编排组件——Orca

~~~
# 200机器：
~]# docker pull docker.io/armory/spinnaker-orca-slim:release-1.8.x-de4ab55
~]# docker images|grep orca
~]# docker tag 5103b1f73e04 harbor.od.com/armory/orca:v1.8.x
~]# docker push harbor.od.com/armory/orca:v1.8.x
~]# mkdir /data/k8s-yaml/orca
~]# cd /data/k8s-yaml/orca/
orca]# vi dp.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    app: armory-orca
  name: armory-orca
  namespace: armory
spec:
  replicas: 1
  revisionHistoryLimit: 7
  selector:
    matchLabels:
      app: armory-orca
  template:
    metadata:
      annotations:
        artifact.spinnaker.io/location: '"armory"'
        artifact.spinnaker.io/name: '"armory-orca"'
        artifact.spinnaker.io/type: '"kubernetes/deployment"'
        moniker.spinnaker.io/application: '"armory"'
        moniker.spinnaker.io/cluster: '"orca"'
      labels:
        app: armory-orca
    spec:
      containers:
      - name: armory-orca
        image: harbor.od.com/armory/orca:v1.8.x
        imagePullPolicy: IfNotPresent
        command:
        - bash
        - -c
        args:
        - bash /opt/spinnaker/config/default/fetch.sh && cd /home/spinnaker/config
          && /opt/orca/bin/orca
        ports:
        - containerPort: 8083
          protocol: TCP
        env:
        - name: JAVA_OPTS
          value: -Xmx1000M
        envFrom:
        - configMapRef:
            name: init-env
        livenessProbe:
          failureThreshold: 5
          httpGet:
            path: /health
            port: 8083
            scheme: HTTP
          initialDelaySeconds: 600
          periodSeconds: 5
          successThreshold: 1
          timeoutSeconds: 1
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /health
            port: 8083
            scheme: HTTP
          initialDelaySeconds: 180
          periodSeconds: 3
          successThreshold: 5
          timeoutSeconds: 1
        volumeMounts:
        - mountPath: /etc/podinfo
          name: podinfo
        - mountPath: /opt/spinnaker/config/default
          name: default-config
        - mountPath: /opt/spinnaker/config/custom
          name: custom-config
      imagePullSecrets:
      - name: harbor
      volumes:
      - configMap:
          defaultMode: 420
          name: custom-config
        name: custom-config
      - configMap:
          defaultMode: 420
          name: default-config
        name: default-config
      - downwardAPI:
          defaultMode: 420
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.labels
            path: labels
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.annotations
            path: annotations
        name: podinfo

orca]# vi svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: armory-orca
  namespace: armory
spec:
  ports:
  - port: 8083
    protocol: TCP
    targetPort: 8083
  selector:
    app: armory-orca

orca]# kubectl apply -f ./dp.yaml
orca]# kubectl apply -f ./svc.yaml
~~~



#### 部署消息总线组件——Echo

~~~
# 200机器：
~]# docker pull docker.io/armory/echo-armory:c36d576-release-1.8.x-617c567
~]# docker images|grep echo
~]# docker tag 415efd46f474 harbor.od.com/armory/echo:v1.8.x
~]# docker push harbor.od.com/armory/echo:v1.8.x
~]# mkdir /data/k8s-yaml/echo
~]# cd /data/k8s-yaml/echo/
echo]# vi dp.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    app: armory-echo
  name: armory-echo
  namespace: armory
spec:
  replicas: 1
  revisionHistoryLimit: 7
  selector:
    matchLabels:
      app: armory-echo
  template:
    metadata:
      annotations:
        artifact.spinnaker.io/location: '"armory"'
        artifact.spinnaker.io/name: '"armory-echo"'
        artifact.spinnaker.io/type: '"kubernetes/deployment"'
        moniker.spinnaker.io/application: '"armory"'
        moniker.spinnaker.io/cluster: '"echo"'
      labels:
        app: armory-echo
    spec:
      containers:
      - name: armory-echo
        image: harbor.od.com/armory/echo:v1.8.x
        imagePullPolicy: IfNotPresent
        command:
        - bash
        - -c
        args:
        - bash /opt/spinnaker/config/default/fetch.sh && cd /home/spinnaker/config
          && /opt/echo/bin/echo
        ports:
        - containerPort: 8089
          protocol: TCP
        env:
        - name: JAVA_OPTS
          value: -javaagent:/opt/echo/lib/jamm-0.2.5.jar -Xmx1000M
        envFrom:
        - configMapRef:
            name: init-env
        livenessProbe:
          failureThreshold: 3
          httpGet:
            path: /health
            port: 8089
            scheme: HTTP
          initialDelaySeconds: 600
          periodSeconds: 3
          successThreshold: 1
          timeoutSeconds: 1
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /health
            port: 8089
            scheme: HTTP
          initialDelaySeconds: 180
          periodSeconds: 3
          successThreshold: 5
          timeoutSeconds: 1
        volumeMounts:
        - mountPath: /etc/podinfo
          name: podinfo
        - mountPath: /opt/spinnaker/config/default
          name: default-config
        - mountPath: /opt/spinnaker/config/custom
          name: custom-config
      imagePullSecrets:
      - name: harbor
      volumes:
      - configMap:
          defaultMode: 420
          name: custom-config
        name: custom-config
      - configMap:
          defaultMode: 420
          name: default-config
        name: default-config
      - downwardAPI:
          defaultMode: 420
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.labels
            path: labels
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.annotations
            path: annotations
        name: podinfo

echo]# vi svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: armory-echo
  namespace: armory
spec:
  ports:
  - port: 8089
    protocol: TCP
    targetPort: 8089
  selector:
    app: armory-echo

echo]# kubectl apply -f ./dp.yaml
echo]# kubectl apply -f ./svc.yaml
~~~

#### 部署流水线交互组件——Igor

~~~~
# 200机器：
~]# docker pull docker.io/armory/spinnaker-igor-slim:release-1.8-x-new-install-healthy-ae2b329
~]# docker images|grep igor
~]# docker tag 23984f5b43f6 harbor.od.com/armory/igor:v1.8.x
~]# docker push harbor.od.com/armory/igor:v1.8.x
~]# mkdir /data/k8s-yaml/igor
~]# cd /data/k8s-yaml/igor/
igor]# vi dp.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    app: armory-igor
  name: armory-igor
  namespace: armory
spec:
  replicas: 1
  revisionHistoryLimit: 7
  selector:
    matchLabels:
      app: armory-igor
  template:
    metadata:
      annotations:
        artifact.spinnaker.io/location: '"armory"'
        artifact.spinnaker.io/name: '"armory-igor"'
        artifact.spinnaker.io/type: '"kubernetes/deployment"'
        moniker.spinnaker.io/application: '"armory"'
        moniker.spinnaker.io/cluster: '"igor"'
      labels:
        app: armory-igor
    spec:
      containers:
      - name: armory-igor
        image: harbor.od.com/armory/igor:v1.8.x
        imagePullPolicy: IfNotPresent
        command:
        - bash
        - -c
        args:
        - bash /opt/spinnaker/config/default/fetch.sh && cd /home/spinnaker/config
          && /opt/igor/bin/igor
        ports:
        - containerPort: 8088
          protocol: TCP
        env:
        - name: IGOR_PORT_MAPPING
          value: -8088:8088
        - name: JAVA_OPTS
          value: -Xmx1000M
        envFrom:
        - configMapRef:
            name: init-env
        livenessProbe:
          failureThreshold: 3
          httpGet:
            path: /health
            port: 8088
            scheme: HTTP
          initialDelaySeconds: 600
          periodSeconds: 3
          successThreshold: 1
          timeoutSeconds: 1
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /health
            port: 8088
            scheme: HTTP
          initialDelaySeconds: 180
          periodSeconds: 5
          successThreshold: 5
          timeoutSeconds: 1
        volumeMounts:
        - mountPath: /etc/podinfo
          name: podinfo
        - mountPath: /opt/spinnaker/config/default
          name: default-config
        - mountPath: /opt/spinnaker/config/custom
          name: custom-config
      imagePullSecrets:
      - name: harbor
      securityContext:
        runAsUser: 0
      volumes:
      - configMap:
          defaultMode: 420
          name: custom-config
        name: custom-config
      - configMap:
          defaultMode: 420
          name: default-config
        name: default-config
      - downwardAPI:
          defaultMode: 420
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.labels
            path: labels
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.annotations
            path: annotations
        name: podinfo

igor]# vi svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: armory-igor
  namespace: armory
spec:
  ports:
  - port: 8088
    protocol: TCP
    targetPort: 8088
  selector:
    app: armory-igor

igor]# kubectl apply -f ./dp.yaml
igor]# kubectl apply -f ./svc.yaml
~~~~



#### 部署API提供组件——Gate

~~~
# 200机器：
~]# docker pull docker.io/armory/gate-armory:dfafe73-release-1.8.x-5d505ca
~]# docker images|grep gate
~]# docker tag b092d4665301 harbor.od.com/armory/gate:v1.8.x
~]# docker push harbor.od.com/armory/gate:v1.8.x
~]# mkdir /data/k8s-yaml/gate
~]# cd /data/k8s-yaml/gate/
gate]# vi dp.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    app: armory-gate
  name: armory-gate
  namespace: armory
spec:
  replicas: 1
  revisionHistoryLimit: 7
  selector:
    matchLabels:
      app: armory-gate
  template:
    metadata:
      annotations:
        artifact.spinnaker.io/location: '"armory"'
        artifact.spinnaker.io/name: '"armory-gate"'
        artifact.spinnaker.io/type: '"kubernetes/deployment"'
        moniker.spinnaker.io/application: '"armory"'
        moniker.spinnaker.io/cluster: '"gate"'
      labels:
        app: armory-gate
    spec:
      containers:
      - name: armory-gate
        image: harbor.od.com/armory/gate:v1.8.x
        imagePullPolicy: IfNotPresent
        command:
        - bash
        - -c
        args:
        - bash /opt/spinnaker/config/default/fetch.sh gate && cd /home/spinnaker/config
          && /opt/gate/bin/gate
        ports:
        - containerPort: 8084
          name: gate-port
          protocol: TCP
        - containerPort: 8085
          name: gate-api-port
          protocol: TCP
        env:
        - name: GATE_PORT_MAPPING
          value: -8084:8084
        - name: GATE_API_PORT_MAPPING
          value: -8085:8085
        - name: JAVA_OPTS
          value: -Xmx1000M
        envFrom:
        - configMapRef:
            name: init-env
        livenessProbe:
          exec:
            command:
            - /bin/bash
            - -c
            - wget -O - http://localhost:8084/health || wget -O - https://localhost:8084/health
          failureThreshold: 5
          initialDelaySeconds: 600
          periodSeconds: 5
          successThreshold: 1
          timeoutSeconds: 1
        readinessProbe:
          exec:
            command:
            - /bin/bash
            - -c
            - wget -O - http://localhost:8084/health?checkDownstreamServices=true&downstreamServices=true
              || wget -O - https://localhost:8084/health?checkDownstreamServices=true&downstreamServices=true
          failureThreshold: 3
          initialDelaySeconds: 180
          periodSeconds: 5
          successThreshold: 10
          timeoutSeconds: 1
        volumeMounts:
        - mountPath: /etc/podinfo
          name: podinfo
        - mountPath: /opt/spinnaker/config/default
          name: default-config
        - mountPath: /opt/spinnaker/config/custom
          name: custom-config
      imagePullSecrets:
      - name: harbor
      securityContext:
        runAsUser: 0
      volumes:
      - configMap:
          defaultMode: 420
          name: custom-config
        name: custom-config
      - configMap:
          defaultMode: 420
          name: default-config
        name: default-config
      - downwardAPI:
          defaultMode: 420
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.labels
            path: labels
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.annotations
            path: annotations
        name: podinfo

gate]# vi svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: armory-gate
  namespace: armory
spec:
  ports:
  - name: gate-port
    port: 8084
    protocol: TCP
    targetPort: 8084
  - name: gate-api-port
    port: 8085
    protocol: TCP
    targetPort: 8085
  selector:
    app: armory-gate

gate]# kubectl apply -f ./dp.yaml
gate]# kubectl apply -f ./svc.yaml
~~~

![1584023952250](assets/1584023952250.png)

5个

#### 部署前端网页项目——Deck

~~~
# 200机器：
~]# docker pull docker.io/armory/deck-armory:d4bf0cf-release-1.8.x-0a33f94
~]# docker images|grep deck
~]# docker tag 9a87ba3b319f harbor.od.com/armory/deck:v1.8.x
~]# docker push harbor.od.com/armory/deck:v1.8.x
~]# mkdir /data/k8s-yaml/deck
~]# cd /data/k8s-yaml/deck/
deck]# vi dp.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    app: armory-deck
  name: armory-deck
  namespace: armory
spec:
  replicas: 1
  revisionHistoryLimit: 7
  selector:
    matchLabels:
      app: armory-deck
  template:
    metadata:
      annotations:
        artifact.spinnaker.io/location: '"armory"'
        artifact.spinnaker.io/name: '"armory-deck"'
        artifact.spinnaker.io/type: '"kubernetes/deployment"'
        moniker.spinnaker.io/application: '"armory"'
        moniker.spinnaker.io/cluster: '"deck"'
      labels:
        app: armory-deck
    spec:
      containers:
      - name: armory-deck
        image: harbor.od.com/armory/deck:v1.8.x
        imagePullPolicy: IfNotPresent
        command:
        - bash
        - -c
        args:
        - bash /opt/spinnaker/config/default/fetch.sh && /entrypoint.sh
        ports:
        - containerPort: 9000
          protocol: TCP
        envFrom:
        - configMapRef:
            name: init-env
        livenessProbe:
          failureThreshold: 3
          httpGet:
            path: /
            port: 9000
            scheme: HTTP
          initialDelaySeconds: 180
          periodSeconds: 3
          successThreshold: 1
          timeoutSeconds: 1
        readinessProbe:
          failureThreshold: 5
          httpGet:
            path: /
            port: 9000
            scheme: HTTP
          initialDelaySeconds: 30
          periodSeconds: 3
          successThreshold: 5
          timeoutSeconds: 1
        volumeMounts:
        - mountPath: /etc/podinfo
          name: podinfo
        - mountPath: /opt/spinnaker/config/default
          name: default-config
        - mountPath: /opt/spinnaker/config/custom
          name: custom-config
      imagePullSecrets:
      - name: harbor
      volumes:
      - configMap:
          defaultMode: 420
          name: custom-config
        name: custom-config
      - configMap:
          defaultMode: 420
          name: default-config
        name: default-config
      - downwardAPI:
          defaultMode: 420
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.labels
            path: labels
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.annotations
            path: annotations
        name: podinfo

deck]# vi svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: armory-deck
  namespace: armory
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 9000
  selector:
    app: armory-deck

deck]# kubectl apply -f ./dp.yaml
deck]# kubectl apply -f ./svc.yaml
~~~

#### 部署前端代理——Nginx

~~~
# 200机器：
~]# docker pull nginx:1.12.2
~]# docker images|grep nginx
~]# docker tag 4037a5562b03 harbor.od.com/armory/nginx:v1.12.2
~]# docker push harbor.od.com/armory/nginx:v1.12.2
~]# mkdir /data/k8s-yaml/nginx
~]# cd /data/k8s-yaml/nginx/
nginx]# vi dp.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    app: armory-nginx
  name: armory-nginx
  namespace: armory
spec:
  replicas: 1
  revisionHistoryLimit: 7
  selector:
    matchLabels:
      app: armory-nginx
  template:
    metadata:
      annotations:
        artifact.spinnaker.io/location: '"armory"'
        artifact.spinnaker.io/name: '"armory-nginx"'
        artifact.spinnaker.io/type: '"kubernetes/deployment"'
        moniker.spinnaker.io/application: '"armory"'
        moniker.spinnaker.io/cluster: '"nginx"'
      labels:
        app: armory-nginx
    spec:
      containers:
      - name: armory-nginx
        image: harbor.od.com/armory/nginx:v1.12.2
        imagePullPolicy: Always
        command:
        - bash
        - -c
        args:
        - bash /opt/spinnaker/config/default/fetch.sh nginx && nginx -g 'daemon off;'
        ports:
        - containerPort: 80
          name: http
          protocol: TCP
        - containerPort: 443
          name: https
          protocol: TCP
        - containerPort: 8085
          name: api
          protocol: TCP
        livenessProbe:
          failureThreshold: 3
          httpGet:
            path: /
            port: 80
            scheme: HTTP
          initialDelaySeconds: 180
          periodSeconds: 3
          successThreshold: 1
          timeoutSeconds: 1
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /
            port: 80
            scheme: HTTP
          initialDelaySeconds: 30
          periodSeconds: 3
          successThreshold: 5
          timeoutSeconds: 1
        volumeMounts:
        - mountPath: /opt/spinnaker/config/default
          name: default-config
        - mountPath: /etc/nginx/conf.d
          name: custom-config
      imagePullSecrets:
      - name: harbor
      volumes:
      - configMap:
          defaultMode: 420
          name: custom-config
        name: custom-config
      - configMap:
          defaultMode: 420
          name: default-config
        name: default-config

nginx]# vi svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: armory-nginx
  namespace: armory
spec:
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 80
  - name: https
    port: 443
    protocol: TCP
    targetPort: 443
  - name: api
    port: 8085
    protocol: TCP
    targetPort: 8085
  selector:
    app: armory-nginx

nginx]# vi ingress.yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  labels:
    app: spinnaker
    web: spinnaker.od.com
  name: armory-nginx
  namespace: armory
spec:
  rules:
  - host: spinnaker.od.com
    http:
      paths:
      - backend:
          serviceName: armory-nginx
          servicePort: 80
~~~



~~~
# 11机器，解析域名：
~]# vi /var/named/od.com.zone
serial 前滚一位
spinnaker          A    10.4.7.10

~]# systemctl restart named
~]# dig -t A spinnaker.od.com +short
#out: 10.4.7.10 
~~~

![1581925138470](assets/1581925138470.png)

~~~
# 200机器，应用资源清单：
nginx]# kubectl apply -f ./dp.yaml
nginx]# kubectl apply -f ./svc.yaml
nginx]# kubectl apply -f ./ingress.yaml
~~~

![1584024475469](assets/1584024475469.png)

[spinnaker.od.com](spinnaker.od.com)

![1584024561816](assets/1584024561816.png)

完成



### 使用spinnaker结合Jenkins构建镜像

把test和prod里的deployment的dubbo删掉，结果如图

![1584024722628](assets/1584024722628.png)

![1584024764124](assets/1584024764124.png)

删完



这时候spinnaker已经没了dubbo了，我们创建

![1584024817000](assets/1584024817000.png)

![1584057829023](assets/1584057829023.png)

刷新spinnaker，点进去，如果页面抖动的可以调成125比例

查看minio

![1584058052650](assets/1584058052650.png)

制作流水线

![1584058085990](assets/1584058085990.png)

![1584058106075](assets/1584058106075.png)

> 先制作service

添加4个参数

![1584058190590](assets/1584058190590.png)

![1584058342950](assets/1584058342950.png)

> 版本号、分支、commitID

![1585354175145](assets/1585354175145.png)

> 标签等

![1584058243942](assets/1584058243942.png)

> 名字

![1584058287708](assets/1584058287708.png)

> 仓库名

![1584058477180](assets/1584058477180.png)

> 保存后增加一个流水线的阶段

![1584058493370](assets/1584058493370.png)

![1584058537386](assets/1584058537386.png)

> 可以看到Jenkins已经被加载进来了，下面的参数化构建的选项也被加载进来了

~~~
# 填入对应参数，下面不能复制粘贴
~~~

![1584058979741](assets/1584058979741.png)

> 建好了，运行
>

![1584059021370](assets/1584059021370.png)

![1584059050225](assets/1584059050225.png)

> 点击Details，然后下拉可以看到#9，然后点进去
>

![1584063826998](assets/1584063826998.png)

> 运行中显示蓝条、成功是绿条、失败是红条

![1584063848587](assets/1584063848587.png)

> 跳转到Jenkins了
>

![1584063864144](assets/1584063864144.png)

> 完成后去看harbor
>

![1584064090529](assets/1584064090529.png)

在公司，新上项目投产只需要这样点点点即可。



### 使用spinnaker配置dubbo服务提供者发布至K8S

![1584064188189](assets/1584064188189.png)

> 开始制作deployment，之前我们是用yaml文本，现在我们只要点点点

![1584064212691](assets/1584064212691.png)

![1584064339967](assets/1584064339967.png)

> Accont：账户（选择集群管理员）
>
> Namespace：所发布的名称空间
>
> Detail：项目名字，要和git的保持一致
>
> Containers：要用那个镜像，从harbor里选
>
> 对准问号会有注释

![1584064378399](assets/1584064378399.png)

> Strategy：发布策略，选择滚动升级，还有个就是重新构建

![1584064490029](assets/1584064490029.png)

> 挂载日志

![1584065127079](assets/1584065127079.png)

> 要连接到Prometheus的设置

![1584065253695](assets/1584065253695.png)

200机器： cat /data/k8s-yaml/test/dubbo-demo-service/dp.yaml，复制指定Value内容填入Value

![1584065443152](assets/1584065443152.png)

![1584065496089](assets/1584065496089.png)

> 传入环境变量：一个是JAR、一个是Apollo

![1584065532833](assets/1584065532833.png)

> 控制资源大小

![1584065559311](assets/1584065559311.png)

> 挂载日志

![1584065607642](assets/1584065607642.png)

> 就绪性探针：
>
> ​	测试端口：20880

![1584065630673](assets/1584065630673.png)



> 拿什么用户去执行

填写第二个container，配filebeat

![1584065690268](assets/1584065690268.png)

![1584066057307](assets/1584066057307.png)

![1584066134675](assets/1584066134675.png)

修改底包

![1584068096613](assets/1584068096613.png)

![1584068118456](assets/1584068118456.png)

更改版本，点这里

![1584066764236](assets/1584066764236.png)

然后再这里修改一些内容

~~~
"imageId": "harbor.od.com/${parameters.image_name}:${parameters.git_ver}_${parameters.add_tag}",
"registry": "harbor.od.com",
"repository": "${parameters.image_name}",
"tag": "${parameters.git_ver}_${parameters.add_tag}"
~~~

![1584066809117](assets/1584066809117.png)

![1584066841074](assets/1584066841074.png)

![1584066869312](assets/1584066869312.png)

然后update并save

![1584066908437](assets/1584066908437.png)

![1584066931500](assets/1584066931500.png)

![1584067032466](assets/1584067032466.png)

![1584067042410](assets/1584067042410.png)

第一阶段完成后就开始第二阶段

![1584067152277](assets/1584067152277.png)

![1584067170255](assets/1584067170255.png)

全部完成后，查看K8S

![1584067199269](assets/1584067199269.png)

![1584067285901](assets/1584067285901.png)

~~~
# 22机器
~]# docker ps -a|grep dubbo-demo-service
~]# docker exec -ti 0f46cfc60e82 bash
#/ cd /logm
#/ ls
#/ cat stdout.log
~~~

![1584068666418](assets/1584068666418.png)

有日志了



查看kibana的日志

![1584068537962](assets/1584068537962.png)

![1584068699067](assets/1584068699067.png)

完成（PS：log日志去到es里也需要时间，我等了大约两分钟）



### 使用spinnaker配置dubbo服务消费者到K8S

创建流水线

![1584078407553](assets/1584078407553.png)

![1584078439769](assets/1584078439769.png)

![1584078491859](assets/1584078491859.png)

![1584078517624](assets/1584078517624.png)

![1584078573644](assets/1584078573644.png)

![1584078612786](assets/1584078612786.png)

![1584078672019](assets/1584078672019.png)

![1584083114651](assets/1584083114651.png)

先制作镜像，看有没有问题

![1584079045516](assets/1584079045516.png)

![1584079077761](assets/1584079077761.png)

去看一下Jenkins能不能打包

![1584079104250](assets/1584079104250.png)

![1584079120220](assets/1584079120220.png)

success后

现在相当于已经配了dp.yaml，这是web项目，所以还要配svc和ingress

![1584079151477](assets/1584079151477.png)

![1584079186625](assets/1584079186625.png)

![1584079823806](assets/1584079823806.png)

![1584079256815](assets/1584079256815.png)

![1584079884058](assets/1584079884058.png)

这时候已经K8S里已经有svc（我把原来的删掉了）

![1584079910637](assets/1584079910637.png)

以前的ingress我也顺便删了

![1584079645933](assets/1584079645933.png)

做完svc还有ingress

![1584079330070](assets/1584079330070.png)

![1584079347814](assets/1584079347814.png)

![1584080007523](assets/1584080007523.png)

去K8S里看

![1584080082818](assets/1584080082818.png)

然后继续

![1584080109009](assets/1584080109009.png)

![1584080129596](assets/1584080129596.png)

![1584080169008](assets/1584080169008.png)

![1584083845728](assets/1584083845728.png)

> 编号可能不同，因为我重新做了一版tomcat，但是无所谓，后面我们都会用变量弄

![1584080263076](assets/1584080263076.png)

![1584083914202](assets/1584083914202.png)

![1584080330316](assets/1584080330316.png)

![1584080535614](assets/1584080535614.png)

第二个container

同样200机器去拿那段

![1584081101719](assets/1584081101719.png)

![1584081175808](assets/1584081175808.png)

第三个container

![1584081304018](assets/1584081304018.png)

![1584081338810](assets/1584081338810.png)

![1584081381128](assets/1584081381128.png)

![1584081394888](assets/1584081394888.png)

~~~
"imageId": "harbor.od.com/${parameters.image_name}:${parameters.git_ver}_${parameters.add_tag}",
"registry": "harbor.od.com",
"repository": "${parameters.image_name}",
"tag": "${parameters.git_ver}_${parameters.add_tag}"
~~~

![1584081434175](assets/1584081434175.png)

update

![1584081462543](assets/1584081462543.png)

首先，网址是没有内容的

![1584081529356](assets/1584081529356.png)

![1584081486194](assets/1584081486194.png)

继续构建

![1584081573772](assets/1584081573772.png)

![1584081698091](assets/1584081698091.png)

> 可以看到只花了两分钟不到

查看K8S里面起来了没有

![1584081739090](assets/1584081739090.png)

然后再刷新页面

![1584084336732](assets/1584084336732.png)

测试页面正常，去kibana（等数据过来又是漫长的过程）

![1584084381158](assets/1584084381158.png)

![1584084728950](assets/1584084728950.png)

完成



### 模拟生产上代码迭代

这里我就直接用gitlab，你用自己的git即可，tomcat分支（`dubbo-client/src/main/java/com/od/dubbotest/action/HelloAction.java`）

![1584085399503](assets/1584085399503.png)

复制id

![1584085534421](assets/1584085534421.png)

![1584085463078](assets/1584085463078.png)

![1584085573303](assets/1584085573303.png)

![1584085726806](assets/1584085726806.png)

完成后刷新页面

![1584085742862](assets/1584085742862.png)

> 版本更新你最需要点这么几下就可以了，全程用不到1分钟

确认一致，可以上线了

![1584085802694](assets/1584085802694.png)

![1584085844657](assets/1584085844657.png)

![1584085871848](assets/1584085871848.png)

![1584085899129](assets/1584085899129.png)

![1584085940886](assets/1584085940886.png)

![1584085965431](assets/1584085965431.png)

![1584086000580](assets/1584086000580.png)

![1584086046821](assets/1584086046821.png)

![1584086075157](assets/1584086075157.png)

![1584086138413](assets/1584086138413.png)

![1584086157826](assets/1584086157826.png)

![1584086175983](assets/1584086175983.png)

![1584086277429](assets/1584086277429.png)

第二个container

同样的方法去拿value：cat /data/k8s-yaml/prod/dubbo-demo-service/dp.yaml

![1584086436864](assets/1584086436864.png)

![1584086463792](assets/1584086463792.png)

第三个container

![1584086522123](assets/1584086522123.png)

![1584086550684](assets/1584086550684.png)



![1584086871253](assets/1584086871253.png)

~~~
# 添加以下内容
"imageId": "harbor.od.com/${parameters.image_name}:${parameters.git_ver}_${parameters.add_tag}",
"registry": "harbor.od.com",
"repository": "${parameters.image_name}",
"tag": "${parameters.git_ver}_${parameters.add_tag}"
~~~

![1584086852397](assets/1584086852397.png)

![1584086900158](assets/1584086900158.png)

看一下哪个版本通过测试了

![1584086932887](assets/1584086932887.png)

复制

![1584086963769](assets/1584086963769.png)

![1584087001949](assets/1584087001949.png)

![1584087023343](assets/1584087023343.png)

![1584087116902](assets/1584087116902.png)

起来了，我们把prod环境里面的service和ingress的consumer干掉

![1584087185801](assets/1584087185801.png)

![1584087200129](assets/1584087200129.png)

开始制作services

![1584087239935](assets/1584087239935.png)

![1584087255667](assets/1584087255667.png)

![1584087279759](assets/1584087279759.png)

再去做ingress

![1584087808345](assets/1584087808345.png)

![1584087822173](assets/1584087822173.png)

ingress也好了

跑起来（其实就跟test一样）

![1584088360303](assets/1584088360303.png)

![1584088366782](assets/1584088366782.png)

![1584088379944](assets/1584088379944.png)![1584088384747](assets/1584088384747.png)

![1584088390258](assets/1584088390258.png)

![1584088394971](assets/1584088394971.png)

![1584088400134](assets/1584088400134.png)

![1584088403965](assets/1584088403965.png)

第二个container

![1584089282070](assets/1584089282070.png)

![1584089287727](assets/1584089287727.png)

![1584089291709](assets/1584089291709.png)

![1584089303135](assets/1584089303135.png)

![1584089307606](assets/1584089307606.png)

![1584089311723](assets/1584089311723.png)

这下面的信息是用test的信息，因为test已经测试过没问题了

![1584089319292](assets/1584089319292.png)

![1584089345632](assets/1584089345632.png)

![1584089352491](assets/1584089352491.png)

会提示替代web，不同管，RUN

![1584089265210](assets/1584089265210.png)

成功

当然你可能担心test的web真的被替代了，可以看一下

![1584089405816](assets/1584089405816.png)

也是没问题的，我们再看下kibana有没有数据

![1584089444787](assets/1584089444787.png)

也有

### 恭喜你，顺利毕业！

