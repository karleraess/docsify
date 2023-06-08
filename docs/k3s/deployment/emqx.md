## emqx
[emqx 文档](https://www.emqx.io/docs/zh/v5.0/)

### 端口说明
tcp：1883  
ssl：8883  
ws：8083  
wss：8084  
管理平台：18083  

### emqx 单节点安装

```shell
docker run -d --name emqx -p 1883:1883 -p 8083:8083 -p 8084:8084 -p 8883:8883 -p 18083:18083 emqx/emqx:5.0.21
```

### emqx 集群compose

```shell
version: '3'

services:
  emqx1:
    image: emqx:5.0.21
    container_name: emqx1
    environment:
    - "EMQX_NODE_NAME=emqx@node1.emqx.io"
    - "EMQX_CLUSTER__DISCOVERY_STRATEGY=static"
    - "EMQX_CLUSTER__STATIC__SEEDS=[emqx@node1.emqx.io,emqx@node2.emqx.io]"
    healthcheck:
      test: ["CMD", "/opt/emqx/bin/emqx_ctl", "status"]
      interval: 5s
      timeout: 25s
      retries: 5
    networks:
      emqx-bridge:
        aliases:
        - node1.emqx.io
    ports:
      - 1883:1883
      - 8083:8083
      - 8084:8084
      - 8883:8883
      - 18083:18083 
    # volumes:
    #   - $PWD/emqx1_data:/opt/emqx/data

  emqx2:
    image: emqx:5.0.21
    container_name: emqx2
    environment:
    - "EMQX_NODE_NAME=emqx@node2.emqx.io"
    - "EMQX_CLUSTER__DISCOVERY_STRATEGY=static"
    - "EMQX_CLUSTER__STATIC__SEEDS=[emqx@node1.emqx.io,emqx@node2.emqx.io]"
    healthcheck:
      test: ["CMD", "/opt/emqx/bin/emqx_ctl", "status"]
      interval: 5s
      timeout: 25s
      retries: 5
    networks:
      emqx-bridge:
        aliases:
        - node2.emqx.io
    # volumes:
    #   - $PWD/emqx2_data:/opt/emqx/data

networks:
  emqx-bridge:
    driver: bridge
```

### k8s 部署

```shell
---
apiVersion: v1
data:
  emqx.conf: >-
    {node:{data_dir:data, etc_dir:etc}, cluster:{discovery_strategy:dns,
    dns:{record_type:srv, name:"emqx-headless.default.svc.cluster.local"}},
    dashboard:{listeners:{http:{bind:18083}}, default_username:admin,
    default_password:public}, listeners:{tcp:{default:{bind:"0.0.0.0:1883",
    max_connections:1024000}}}}
kind: ConfigMap
metadata:
  labels:
    apps.emqx.io/instance: emqx
  name: emqx-bootstrap-config
  namespace: default



---
apiVersion: v1
data:
  node_cookie: >-
    NWxqYnJubGFiZGp2b2JlcWgxMjZ4bWFzZjRqY3B2Z204YnBwcnltYWw3c2U5cWRnbnpxYzF6dG1wajB2a2dseA==
kind: Secret
metadata:
  labels:
    apps.emqx.io/instance: emqx
  name: emqx-node-cookie
  namespace: default
type: Opaque





---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  labels:
    apps.emqx.io/db-role: core
    apps.emqx.io/instance: emqx
    apps.emqx.io/managed-by: emqx-operator
  name: emqx-core
  namespace: default
spec:
  podManagementPolicy: Parallel
  replicas: 0
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      apps.emqx.io/db-role: core
      apps.emqx.io/instance: emqx
      apps.emqx.io/managed-by: emqx-operator
  serviceName: emqx-headless
  template:
    metadata:
      annotations:
        apps.emqx.io/headless-service-name: emqx-headless
        apps.emqx.io/manage-containers: emqx
      creationTimestamp: null
      labels:
        apps.emqx.io/db-role: core
        apps.emqx.io/instance: emqx
        apps.emqx.io/managed-by: emqx-operator
    spec:
      containers:
        - env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: metadata.namespace
            - name: STS_HEADLESS_SERVICE_NAME
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: 'metadata.annotations[''apps.emqx.io/headless-service-name'']'
            - name: EMQX_HOST
              value: >-
                $(POD_NAME).$(STS_HEADLESS_SERVICE_NAME).$(POD_NAMESPACE).svc.cluster.local
            - name: EMQX_NODE__DB_ROLE
              value: core
            - name: EMQX_NODE__COOKIE
              valueFrom:
                secretKeyRef:
                  key: node_cookie
                  name: emqx-node-cookie
          image: 'emqx:5.0'
          imagePullPolicy: IfNotPresent
          livenessProbe:
            failureThreshold: 3
            httpGet:
              path: /status
              port: 18083
              scheme: HTTP
            initialDelaySeconds: 60
            periodSeconds: 30
            successThreshold: 1
            timeoutSeconds: 1
          name: emqx
          readinessProbe:
            failureThreshold: 12
            httpGet:
              path: /status
              port: 18083
              scheme: HTTP
            initialDelaySeconds: 10
            periodSeconds: 5
            successThreshold: 1
            timeoutSeconds: 1
          resources: {}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          volumeMounts:
            - mountPath: /opt/emqx/data
              name: emqx-core-data
            - mountPath: /opt/emqx/etc/emqx.conf
              name: bootstrap-config
              readOnly: true
              subPath: emqx.conf
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
      volumes:
        - emptyDir: {}
          name: emqx-core-data
        - configMap:
            defaultMode: 420
            name: emqx-bootstrap-config
          name: bootstrap-config
  updateStrategy:
    rollingUpdate:
      partition: 0
    type: RollingUpdate

---
apiVersion: v1
kind: Service
metadata:
  name: emqx-headless
  namespace: default
spec:
  clusterIP: None
  clusterIPs:
    - None
  internalTrafficPolicy: Cluster
  ipFamilies:
    - IPv4
  ipFamilyPolicy: SingleStack
  ports:
    - name: ekka
      port: 4370
      protocol: TCP
      targetPort: 4370
  publishNotReadyAddresses: true
  selector:
    apps.emqx.io/db-role: core
    apps.emqx.io/instance: emqx
  sessionAffinity: None
  type: ClusterIP

---
apiVersion: v1
kind: Service
metadata:
  annotations: {}
  labels:
    apps.emqx.io/db-role: core
    apps.emqx.io/instance: emqx
    k8s.kuboard.cn/name: emqx-core
  name: emqx-core
  namespace: default
spec:
  internalTrafficPolicy: Cluster
  ipFamilies:
    - IPv4
  ipFamilyPolicy: SingleStack
  ports:
    - name: tcp
      port: 1883
      protocol: TCP
      targetPort: 1883
    - name: ssl
      port: 8883
      protocol: TCP
      targetPort: 8883
    - name: ws
      port: 8083
      protocol: TCP
      targetPort: 8083
    - name: wss
      port: 8084
      protocol: TCP
      targetPort: 8084
    - name: http
      port: 18083
      protocol: TCP
      targetPort: 18083
  selector:
    apps.emqx.io/db-role: core
    apps.emqx.io/instance: emqx
    apps.emqx.io/managed-by: emqx-operator
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}
```


### 查看集群状态
```shell
$ emqx_ctl cluster status
正常结果
Cluster status: #{running_nodes => ['emqx@node1.emqx.com','emqx@node2.emqx.com'],
stopped_nodes => []}
```
