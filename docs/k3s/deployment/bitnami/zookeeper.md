# bitnami-zookeeper

## helm

可在dockerhub的bitnami/zookeeper中查看详细

```shell
helm repo add my-repo https://charts.bitnami.com/bitnami
helm install my-release my-repo/zookeeper
```

## 自定义
### 工作负载ymal

```yaml
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  annotations:
    meta.helm.sh/release-name: bitnami-zookeeper
    meta.helm.sh/release-namespace: default
  labels:
    app.kubernetes.io/component: zookeeper
    app.kubernetes.io/instance: bitnami-zookeeper
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: zookeeper
    helm.sh/chart: zookeeper-11.1.6
    role: zookeeper
  name: bitnami-zookeeper
  namespace: default
  resourceVersion: '1160272'
spec:
  podManagementPolicy: Parallel
  replicas: 0
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app.kubernetes.io/component: zookeeper
      app.kubernetes.io/instance: bitnami-zookeeper
      app.kubernetes.io/name: zookeeper
  serviceName: bitnami-zookeeper-headless
  template:
    metadata:
      creationTimestamp: null
      labels:
        app.kubernetes.io/component: zookeeper
        app.kubernetes.io/instance: bitnami-zookeeper
        app.kubernetes.io/managed-by: Helm
        app.kubernetes.io/name: zookeeper
        helm.sh/chart: zookeeper-11.1.6
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - podAffinityTerm:
                labelSelector:
                  matchLabels:
                    app.kubernetes.io/component: zookeeper
                    app.kubernetes.io/instance: bitnami-zookeeper
                    app.kubernetes.io/name: zookeeper
                topologyKey: kubernetes.io/hostname
              weight: 1
      containers:
        - command:
            - /scripts/setup.sh
          env:
            - name: BITNAMI_DEBUG
              value: 'false'
            - name: ZOO_DATA_LOG_DIR
            - name: ZOO_PORT_NUMBER
              value: '2181'
            - name: ZOO_TICK_TIME
              value: '2000'
            - name: ZOO_INIT_LIMIT
              value: '10'
            - name: ZOO_SYNC_LIMIT
              value: '5'
            - name: ZOO_PRE_ALLOC_SIZE
              value: '65536'
            - name: ZOO_SNAPCOUNT
              value: '100000'
            - name: ZOO_MAX_CLIENT_CNXNS
              value: '60'
            - name: ZOO_4LW_COMMANDS_WHITELIST
              value: 'srvr, mntr, ruok'
            - name: ZOO_LISTEN_ALLIPS_ENABLED
              value: 'no'
            - name: ZOO_AUTOPURGE_INTERVAL
              value: '0'
            - name: ZOO_AUTOPURGE_RETAIN_COUNT
              value: '3'
            - name: ZOO_MAX_SESSION_TIMEOUT
              value: '40000'
            - name: ZOO_SERVERS
              value: >-
                bitnami-zookeeper-0.bitnami-zookeeper-headless.default.svc.cluster.local:2888:3888::1,bitnami-zookeeper-1.bitnami-zookeeper-headless.default.svc.cluster.local:2888:3888::2,bitnami-zookeeper-2.bitnami-zookeeper-headless.default.svc.cluster.local:2888:3888::3
            - name: ZOO_ENABLE_AUTH
              value: 'no'
            - name: ZOO_ENABLE_QUORUM_AUTH
              value: 'no'
            - name: ZOO_HEAP_SIZE
              value: '1024'
            - name: ZOO_LOG_LEVEL
              value: ERROR
            - name: ALLOW_ANONYMOUS_LOGIN
              value: 'yes'
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: metadata.name
          image: 'registry.cn-hangzhou.aliyuncs.com/karleraess/zookeeper:bitnami-3.8.1'
          imagePullPolicy: IfNotPresent
          livenessProbe:
            exec:
              command:
                - /bin/bash
                - '-c'
                - echo "ruok" | timeout 2 nc -w 2 localhost 2181 | grep imok
            failureThreshold: 6
            initialDelaySeconds: 30
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 5
          name: zookeeper
          ports:
            - containerPort: 2181
              name: client
              protocol: TCP
            - containerPort: 2888
              name: follower
              protocol: TCP
            - containerPort: 3888
              name: election
              protocol: TCP
          readinessProbe:
            exec:
              command:
                - /bin/bash
                - '-c'
                - echo "ruok" | timeout 2 nc -w 2 localhost 2181 | grep imok
            failureThreshold: 6
            initialDelaySeconds: 5
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 5
          resources:
            requests:
              cpu: 250m
              memory: 256Mi
          securityContext:
            allowPrivilegeEscalation: false
            runAsNonRoot: true
            runAsUser: 1001
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          volumeMounts:
            - mountPath: /scripts/setup.sh
              name: scripts
              subPath: setup.sh
            - mountPath: /bitnami/zookeeper
              name: data
      dnsPolicy: ClusterFirst
      imagePullSecrets:
        - name: ali
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext:
        fsGroup: 1001
      serviceAccount: default
      serviceAccountName: default
      terminationGracePeriodSeconds: 30
      volumes:
        - configMap:
            defaultMode: 493
            name: bitnami-zookeeper-scripts
          name: scripts
  updateStrategy:
    rollingUpdate:
      partition: 0
    type: RollingUpdate
  volumeClaimTemplates:
    - apiVersion: v1
      kind: PersistentVolumeClaim
      metadata:
        k8s.kuboard.cn/pvcType: Dynamic
        creationTimestamp: null
        name: data
      spec:
        accessModes:
          - ReadWriteOnce
        resources:
          requests:
            storage: 8Gi
        storageClassName: local-path
        volumeMode: Filesystem
      status:
        phase: Pending

---
apiVersion: v1
kind: Service
metadata:
  annotations:
    meta.helm.sh/release-name: bitnami-zookeeper
    meta.helm.sh/release-namespace: default
  labels:
    app.kubernetes.io/component: zookeeper
    app.kubernetes.io/instance: bitnami-zookeeper
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: zookeeper
    helm.sh/chart: zookeeper-11.1.6
  name: bitnami-zookeeper-headless
  namespace: default
  resourceVersion: '1159766'
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
      port: 2181
      protocol: TCP
      targetPort: client
    - name: tcp-follower
      port: 2888
      protocol: TCP
      targetPort: follower
    - name: tcp-election
      port: 3888
      protocol: TCP
      targetPort: election
  publishNotReadyAddresses: true
  selector:
    app.kubernetes.io/component: zookeeper
    app.kubernetes.io/instance: bitnami-zookeeper
    app.kubernetes.io/name: zookeeper
  sessionAffinity: None
  type: ClusterIP

---
apiVersion: v1
kind: Service
metadata:
  annotations:
    meta.helm.sh/release-name: bitnami-zookeeper
    meta.helm.sh/release-namespace: default
  labels:
    app.kubernetes.io/component: zookeeper
    app.kubernetes.io/instance: bitnami-zookeeper
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: zookeeper
    helm.sh/chart: zookeeper-11.1.6
  name: bitnami-zookeeper
  namespace: default
  resourceVersion: '1159768'
spec:
  clusterIP: 10.43.154.182
  clusterIPs:
    - 10.43.154.182
  internalTrafficPolicy: Cluster
  ipFamilies:
    - IPv4
  ipFamilyPolicy: SingleStack
  ports:
    - name: tcp-client
      port: 2181
      protocol: TCP
      targetPort: client
    - name: tcp-follower
      port: 2888
      protocol: TCP
      targetPort: follower
    - name: tcp-election
      port: 3888
      protocol: TCP
      targetPort: election
  selector:
    app.kubernetes.io/component: zookeeper
    app.kubernetes.io/instance: bitnami-zookeeper
    app.kubernetes.io/name: zookeeper
  sessionAffinity: None
  type: ClusterIP

---
apiVersion: v1
data:
  init-certs.sh: '#!/bin/bash'
  setup.sh: |-
    #!/bin/bash

    # Execute entrypoint as usual after obtaining ZOO_SERVER_ID
    # check ZOO_SERVER_ID in persistent volume via myid
    # if not present, set based on POD hostname
    if [[ -f "/bitnami/zookeeper/data/myid" ]]; then
        export ZOO_SERVER_ID="$(cat /bitnami/zookeeper/data/myid)"
    else
        HOSTNAME="$(hostname -s)"
        if [[ $HOSTNAME =~ (.*)-([0-9]+)$ ]]; then
            ORD=${BASH_REMATCH[2]}
            export ZOO_SERVER_ID="$((ORD + 1 ))"
        else
            echo "Failed to get index from hostname $HOST"
            exit 1
        fi
    fi
    exec /entrypoint.sh /run.sh
kind: ConfigMap
metadata:
  annotations:
    meta.helm.sh/release-name: bitnami-zookeeper
    meta.helm.sh/release-namespace: default
  labels:
    app.kubernetes.io/component: zookeeper
    app.kubernetes.io/instance: bitnami-zookeeper
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: zookeeper
    helm.sh/chart: zookeeper-11.1.6
  name: bitnami-zookeeper-scripts
  namespace: default
  resourceVersion: '1172508'


```

## zk 相关命令
### 查看zk的leader/follower 状态
```shell
/opt/bitnami/zookeeper/bin/zkServer.sh status
```