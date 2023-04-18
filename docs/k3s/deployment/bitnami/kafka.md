# bitnami-kafka

## helm

可在dockerhub的bitnami/kafka中查看详细

```shell
helm repo add my-repo https://charts.bitnami.com/bitnami
helm install my-release my-repo/kafka
```

## 自定义
### 先创建用户
```shell
kubectl create serviceaccount bitnami-kafka
```

### 工作负载ymal
```yaml
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  annotations:
    meta.helm.sh/release-name: bitnami-kafka
    meta.helm.sh/release-namespace: default
  labels:
    app.kubernetes.io/component: kafka
    app.kubernetes.io/instance: bitnami-kafka
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: kafka
    helm.sh/chart: kafka-21.4.4
    k8s.kuboard.cn/name: bitnami-kafka
  name: bitnami-kafka
  namespace: default
  resourceVersion: '1173337'
spec:
  podManagementPolicy: Parallel
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app.kubernetes.io/component: kafka
      app.kubernetes.io/instance: bitnami-kafka
      app.kubernetes.io/name: kafka
  serviceName: bitnami-kafka-headless
  template:
    metadata:
      creationTimestamp: null
      labels:
        app.kubernetes.io/component: kafka
        app.kubernetes.io/instance: bitnami-kafka
        app.kubernetes.io/managed-by: Helm
        app.kubernetes.io/name: kafka
        helm.sh/chart: kafka-21.4.4
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - podAffinityTerm:
                labelSelector:
                  matchLabels:
                    app.kubernetes.io/component: kafka
                    app.kubernetes.io/instance: bitnami-kafka
                    app.kubernetes.io/name: kafka
                topologyKey: kubernetes.io/hostname
              weight: 1
      containers:
        - command:
            - /scripts/setup.sh
          env:
            - name: BITNAMI_DEBUG
              value: 'false'
            - name: MY_POD_IP
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: status.podIP
            - name: MY_POD_NAME
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: metadata.name
            - name: KAFKA_CFG_ZOOKEEPER_CONNECT
              value: >-
                bitnami-zookeeper-0.bitnami-zookeeper-headless.default.svc.cluster.local:2181,bitnami-zookeeper-1.bitnami-zookeeper-headless.default.svc.cluster.local:2181,bitnami-zookeeper-2.bitnami-zookeeper-headless.default.svc.cluster.local:2181
            - name: KAFKA_INTER_BROKER_LISTENER_NAME
              value: INTERNAL
            - name: KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP
              value: 'INTERNAL:PLAINTEXT,CLIENT:PLAINTEXT'
            - name: KAFKA_CFG_LISTENERS
              value: 'INTERNAL://:9093,CLIENT://:9092'
            - name: KAFKA_CFG_ADVERTISED_LISTENERS
              value: >-
                INTERNAL://$(MY_POD_NAME).bitnami-kafka-headless.default.svc.cluster.local:9093,CLIENT://$(MY_POD_NAME).bitnami-kafka-headless.default.svc.cluster.local:9092
            - name: ALLOW_PLAINTEXT_LISTENER
              value: 'yes'
            - name: KAFKA_ZOOKEEPER_PROTOCOL
              value: PLAINTEXT
            - name: KAFKA_VOLUME_DIR
              value: /bitnami/kafka
            - name: KAFKA_LOG_DIR
              value: /opt/bitnami/kafka/logs
            - name: KAFKA_CFG_DELETE_TOPIC_ENABLE
              value: 'false'
            - name: KAFKA_CFG_AUTO_CREATE_TOPICS_ENABLE
              value: 'true'
            - name: KAFKA_HEAP_OPTS
              value: '-Xmx1024m -Xms1024m'
            - name: KAFKA_CFG_LOG_FLUSH_INTERVAL_MESSAGES
              value: '10000'
            - name: KAFKA_CFG_LOG_FLUSH_INTERVAL_MS
              value: '1000'
            - name: KAFKA_CFG_LOG_RETENTION_BYTES
              value: '1073741824'
            - name: KAFKA_CFG_LOG_RETENTION_CHECK_INTERVAL_MS
              value: '300000'
            - name: KAFKA_CFG_LOG_RETENTION_HOURS
              value: '168'
            - name: KAFKA_CFG_MESSAGE_MAX_BYTES
              value: '1000012'
            - name: KAFKA_CFG_LOG_SEGMENT_BYTES
              value: '1073741824'
            - name: KAFKA_CFG_LOG_DIRS
              value: /bitnami/kafka/data
            - name: KAFKA_CFG_DEFAULT_REPLICATION_FACTOR
              value: '1'
            - name: KAFKA_CFG_OFFSETS_TOPIC_REPLICATION_FACTOR
              value: '1'
            - name: KAFKA_CFG_TRANSACTION_STATE_LOG_REPLICATION_FACTOR
              value: '1'
            - name: KAFKA_CFG_TRANSACTION_STATE_LOG_MIN_ISR
              value: '1'
            - name: KAFKA_CFG_NUM_IO_THREADS
              value: '8'
            - name: KAFKA_CFG_NUM_NETWORK_THREADS
              value: '3'
            - name: KAFKA_CFG_NUM_PARTITIONS
              value: '1'
            - name: KAFKA_CFG_NUM_RECOVERY_THREADS_PER_DATA_DIR
              value: '1'
            - name: KAFKA_CFG_SOCKET_RECEIVE_BUFFER_BYTES
              value: '102400'
            - name: KAFKA_CFG_SOCKET_REQUEST_MAX_BYTES
              value: '104857600'
            - name: KAFKA_CFG_SOCKET_SEND_BUFFER_BYTES
              value: '102400'
            - name: KAFKA_CFG_ZOOKEEPER_CONNECTION_TIMEOUT_MS
              value: '6000'
            - name: KAFKA_CFG_AUTHORIZER_CLASS_NAME
            - name: KAFKA_CFG_ALLOW_EVERYONE_IF_NO_ACL_FOUND
              value: 'true'
            - name: KAFKA_CFG_SUPER_USERS
              value: 'User:admin'
          image: 'registry.cn-hangzhou.aliyuncs.com/karleraess/kafka:bitnami-3.4.0'
          imagePullPolicy: IfNotPresent
          livenessProbe:
            failureThreshold: 3
            initialDelaySeconds: 10
            periodSeconds: 10
            successThreshold: 1
            tcpSocket:
              port: kafka-client
            timeoutSeconds: 5
          name: kafka
          ports:
            - containerPort: 9092
              name: kafka-client
              protocol: TCP
            - containerPort: 9093
              name: kafka-internal
              protocol: TCP
          readinessProbe:
            failureThreshold: 6
            initialDelaySeconds: 5
            periodSeconds: 10
            successThreshold: 1
            tcpSocket:
              port: kafka-client
            timeoutSeconds: 5
          resources: {}
          securityContext:
            allowPrivilegeEscalation: false
            runAsNonRoot: true
            runAsUser: 1001
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          volumeMounts:
            - mountPath: /bitnami/kafka
              name: data
            - mountPath: /opt/bitnami/kafka/logs
              name: logs
            - mountPath: /scripts/setup.sh
              name: scripts
              subPath: setup.sh
      dnsPolicy: ClusterFirst
      imagePullSecrets:
        - name: ali
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext:
        fsGroup: 1001
      serviceAccount: bitnami-kafka
      serviceAccountName: bitnami-kafka
      terminationGracePeriodSeconds: 30
      volumes:
        - configMap:
            defaultMode: 493
            name: bitnami-kafka-scripts
          name: scripts
        - emptyDir: {}
          name: logs
  updateStrategy:
    rollingUpdate:
      partition: 0
    type: RollingUpdate
  volumeClaimTemplates:
    - apiVersion: v1
      kind: PersistentVolumeClaim
      metadata:
        creationTimestamp: null
        name: data
      spec:
        accessModes:
          - ReadWriteOnce
        resources:
          requests:
            storage: 5Gi
        storageClassName: local-path
        volumeMode: Filesystem
      status:
        phase: Pending
status:
  availableReplicas: 0
  collisionCount: 0
  currentRevision: bitnami-kafka-78b76f7b57
  observedGeneration: 15
  replicas: 0
  updateRevision: bitnami-kafka-78b76f7b57

---
apiVersion: v1
kind: Service
metadata:
  annotations:
    meta.helm.sh/release-name: bitnami-kafka
    meta.helm.sh/release-namespace: default
  labels:
    app.kubernetes.io/component: kafka
    app.kubernetes.io/instance: bitnami-kafka
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: kafka
    helm.sh/chart: kafka-21.4.4
  name: bitnami-kafka-headless
  namespace: default
  resourceVersion: '1171297'
spec:
  clusterIP: None
  clusterIPs:
    - None
  internalTrafficPolicy: Cluster
  ipFamilies:
    - IPv4
  ipFamilyPolicy: SingleStack
  ports:
    - name: tcp-client
      port: 9092
      protocol: TCP
      targetPort: kafka-client
    - name: tcp-internal
      port: 9093
      protocol: TCP
      targetPort: kafka-internal
  selector:
    app.kubernetes.io/component: kafka
    app.kubernetes.io/instance: bitnami-kafka
    app.kubernetes.io/name: kafka
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}

---
apiVersion: v1
kind: Service
metadata:
  annotations:
    meta.helm.sh/release-name: bitnami-kafka
    meta.helm.sh/release-namespace: default
  labels:
    app.kubernetes.io/component: kafka
    app.kubernetes.io/instance: bitnami-kafka
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: kafka
    helm.sh/chart: kafka-21.4.4
  name: bitnami-kafka
  namespace: default
  resourceVersion: '1171302'
spec:
  clusterIP: 10.43.159.150
  clusterIPs:
    - 10.43.159.150
  internalTrafficPolicy: Cluster
  ipFamilies:
    - IPv4
  ipFamilyPolicy: SingleStack
  ports:
    - name: tcp-client
      port: 9092
      protocol: TCP
      targetPort: kafka-client
  selector:
    app.kubernetes.io/component: kafka
    app.kubernetes.io/instance: bitnami-kafka
    app.kubernetes.io/name: kafka
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}

---
apiVersion: v1
data:
  setup.sh: |-
    #!/bin/bash

    ID="${MY_POD_NAME#"bitnami-kafka-"}"
    # If process.roles is not set at all, it is assumed to be in ZooKeeper mode.
    # https://kafka.apache.org/documentation/#kraft_role

    if [[ -f "/bitnami/kafka/data/meta.properties" ]]; then
        if [[ $KAFKA_CFG_PROCESS_ROLES == "" ]]; then
            export KAFKA_CFG_BROKER_ID="$(grep "broker.id" "/bitnami/kafka/data/meta.properties" | awk -F '=' '{print $2}')"
        else
            export KAFKA_CFG_BROKER_ID="$(grep "node.id" "/bitnami/kafka/data/meta.properties" | awk -F '=' '{print $2}')"
        fi
    else
        export KAFKA_CFG_BROKER_ID="$((ID + 0))"
    fi

    if [[ $KAFKA_CFG_PROCESS_ROLES == *"controller"* ]]; then
        node_id=0
        pod_id=0
        while :
        do 
            VOTERS="${VOTERS}$node_id@bitnami-kafka-$pod_id.bitnami-kafka-headless.default.svc.cluster.local:9095"
            node_id=$(( $node_id + 1 ))
            pod_id=$(( $pod_id + 1 ))
            if [[ $pod_id -ge 1 ]]; then
                break
            else
                VOTERS="$VOTERS,"
            fi
        done
        export KAFKA_CFG_CONTROLLER_QUORUM_VOTERS=$VOTERS
    fi

    # Configure zookeeper client

    exec /entrypoint.sh /run.sh
kind: ConfigMap
metadata:
  annotations:
    meta.helm.sh/release-name: bitnami-kafka
    meta.helm.sh/release-namespace: default
  labels:
    app.kubernetes.io/instance: bitnami-kafka
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: kafka
    helm.sh/chart: kafka-21.4.4
  name: bitnami-kafka-scripts
  namespace: default
  resourceVersion: '1173532'


```


## kafka 相关命令
### 查看
```shell
./opt/kafka/bin/kafka-topics.sh --zookeeper zk-1.zk-hs.default.svc.cluster.local:2181 --describe --topic csm-screen-biz-message
```

### 创建topic
```shell
./opt/kafka/bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic mytest
```

### 下面是通过命令指定分区数和副本数
```shell
./opt/kafka/bin/kafka-topics.sh  --zookeeper zok-0.zk-hs.default.svc.cluster.local:2181 --create   --topic csm-screen-biz-message --partitions 2  --replication-factor 1
PS: 创建topic时，分区数为num.partitions（默认1），副本因子为default.replication.factor
```
### 更新topic
```shell
./opt/kafka/bin/kafka-topics.sh --alter --zookeeper zk-2.zk-hs.default.svc.cluster.local:2181 --topic csm-screen-biz-message --partitions 2
```