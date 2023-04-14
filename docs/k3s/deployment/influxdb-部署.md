# 一、InfluxDB介绍

InfluxDB是一款用Go语言编写的开源分布式时序、事件和指标数据库。

官方提供1.X和2.X两个版本，版本区别较大。改动最为明显的是语法的变化，1.X版本使用的是SQL语句，2.X版本使用的JavaScript。

两个版本官方均在提供技术支持。其中，1.X目前最新版本为1.8.4，2.X目前最新版本为2.0.4。

本文档使用的InfluxDB版本为1.8.4。

# 二、InfluxDB部署

访问hub.docker.com查看InfluxDB官方镜像文档说明，根据官方文档介绍，导出默认配置文件。

```shell
docker pull influxdb:1.8.4-alpine       #拉取最新版本镜像
docker run --rm influxdb:1.8.4-alpine influxd config > influxdb.conf   #导出默认配置文件

```
根据导出的配置文件内容创建对应的K8S ConfigMap，以便日后对部署在K8S内的InfluDB进行配置管理

```shell
kubectl create namespace influxdb
kubectl create configmap influxdb-config --from-file influxdb.conf -n influxdb
kubectl get cm influxdb-config -n influxdb
NAME              DATA   AGE
influxdb-config   1      26s
kubectl get cm influxdb-config -n influxdb -oyaml   #查看CM内容，内容省略
```


通过默认配置文件内容可以知道，InfluxDB数据保存在/var/lib/influxdb目录中，为了实现持久化，需要为InfluxDB创建storageclass和pvc，其中storageclass使用的是K8S已部署好的后端ceph存储。

```shell
cat influxdb-sc.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: influxdb-sc
provisioner: ceph.com/cephfs
parameters:
  adminId: admin
  adminSecretName: ceph-secret
  adminSecretNamespace: kube-system
  claimRoot: /volumes/kubernetes/influxdb-sc
  monitors: 192.168.208.27:6789,192.168.208.28:6789,192.168.208.29:6789
cat influxdb-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: influxdb-pvc
  namespace: influxdb
  annotations:
    volume.beta.kubernetes.io/storage-class: "influxdb-sc"
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
    storage: 20Gi
```


配置InfluxDB deployment和service的yaml文件

```shell
cat influxdb-dp.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: influxdb-dp
  namespace: influxdb
  labels:
spec:
  replicas: 1
  selector:
    matchLabels:
      app: influxdb
  template:
    metadata:
      labels:
        app: influxdb
    spec:
      containers:
      - name: influxdb
        image: influxdb:1.8.4-alpine
        imagePullPolicy: IfNotPresent
        ports:
        - name: influxdb
          containerPort: 8086
          protocol: TCP
        volumeMounts:
        - name: influxdb-data
          mountPath: /var/lib/influxdb
          subPath: influxdb
        - name: influxdb-config
          mountPath: /etc/influxdb
      volumes:
      - name: influxdb-data
        persistentVolumeClaim:
          claimName: influxdb-pvc
      - name: influxdb-config
        configMap:
          name: influxdb-config
		  
		  
cat influxdb-svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: influxdb-svc
  namespace: influxdb
spec:
  type: NodePort
  ports:
    - port: 8086
      targetPort: 8086
      name: influxdb
  selector:
    app: influxdb
```

创建InfluxDB对应资源，查看pod状态，svc状态，endpoint状态是否正常

```shell
kubectl apply -f influxdb-*.yaml
kubectl get pod -n influxdb
NAME                           READY   STATUS    RESTARTS   AGE
influxdb-dp-6c8756cfcd-pc8gf   1/1     Running   0          6m28s
kubectl get svc -n influxdb
NAME           TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
influxdb-svc   NodePort   10.108.71.107   <none>        8086:31945/TCP   6h11m
kubectl get ep influxdb-svc -n influxdb
NAME           ENDPOINTS           AGE
influxdb-svc   172.17.5.186:8086   6h11m
```


# 三、Influxdb开启认证

首先进入pod，登录influxdb服务器创建用户

```shell
kubectl -n influxdb exec influxdb-dp-6c8756cfcd-pc8gf -it -- /bin/bash
bash-5.0# influx
Connected to http://localhost:8086 version 1.8.4
InfluxDB shell version: 1.8.4
> CREATE USER root WITH PASSWORD '123456' WITH ALL PRIVILEGES
> show users
user admin
---- -----
root true
> exit

```

修改configmap内容，开启InfluxDB登录认证功能，此步骤能顺便测试configmap是否挂载正常

```shell
kubectl edit cm influxdb-config  -n influxdb
#....其他内容省略....
    [http]
      enabled = true
      bind-address = ":8086"
      auth-enabled = true        #将此项false改为true
#....其他内容省略....
```


重启pod以重新加载configmap配置文件，此步骤能顺便测试pvc持久卷是否挂载正常

```shell
kubectl -n influxdb delete pod influxdb-dp-6c8756cfcd-pc8gf
kubectl -n influxdb get pod
NAME                           READY   STATUS    RESTARTS   AGE
influxdb-dp-6c8756cfcd-fhvws   1/1     Running   0          30s

```

进入pod，测试认证功能是否开启

```shell
kubectl -n influxdb exec influxdb-dp-6c8756cfcd-fhvws -it -- /bin/bash
bash-5.0# influx
Connected to http://localhost:8086 version 1.8.4
InfluxDB shell version: 1.8.4
> show database;
ERR: unable to parse authentication credentials
Warning: It is possible this error is due to not setting a database.        #因为没有登录，所以报错了
Please set a database with the command "use <database>".
> auth                                                                      #输入auth进行认证，填写创建好的账号密码
username: root
password:
> show databases                                                            #认证后可以正常列出databases
name: databases
name
----
_internal
> exit
```

至此，InfluxDB部署完成，也实现了数据持久化。