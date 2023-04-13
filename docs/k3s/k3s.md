

#  k3s 安装相关
> 官方文档  https://docs.k3s.io/zh/quick-start

## master安装
```shell
 curl -sfL https://rancher-mirror.rancher.cn/k3s/k3s-install.sh | INSTALL_K3S_VERSION=v1.25.6+k3s1 INSTALL_K3S_MIRROR=cn  INSTALL_K3S_EXEC="--docker" sh -s - server --docker --service-node-port-range=1-33000
```

## work 节点安装
```shell
curl -sfL https://rancher-mirror.rancher.cn/k3s/k3s-install.sh | INSTALL_K3S_MIRROR=cn K3S_URL=https://myserver:6443 K3S_TOKEN=mynodetoken sh -
```
K3S_URL 参数会导致安装程序将 K3s 配置为 Agent 而不是 Server。K3s Agent 将注册到在 URL 上监听的 K3s Server。K3S_TOKEN 使用的值存储在 Server 节点上的 **/var/lib/rancher/k3s/server/node-token **中。
> ps：每台主机必须具有唯一的主机名。如果你的计算机没有唯一的主机名，请传递 K3S_NODE_NAME 环境变量，并为每个节点提供一个有效且唯一的主机名。



## 卸载
```shell
/usr/local/bin/k3s-uninstall.sh
```

# k8s 命令
```shell
kubectl get pods --all-namespaces
kubectl exec -ti kuboard-agent-1a9ayvj-2-5dcdccb57-zn5sh  -n kuboard  -- /bin/sh
kubectl logs coredns-597584b69b-cwfd8 -n kube-system
kubectl describe pod kuboard-agent-1a9ayvj-2-5dcdccb57-w6fv4 -n kuboard
kubectl delete deployment kuboard-agent-1a9ayvj-2-5dcdccb57-nc664 -n kuboard
```


# k3s管理UI - kuboard

```shell
 sudo docker run -d \
   --restart=unless-stopped \
   --name=kuboard \
   -p 9980:80/tcp \
   -p 10081:10081/tcp \
   -e KUBOARD_ENDPOINT="http://121.41.197.138:9980" \
   -e KUBOARD_AGENT_SERVER_TCP_PORT="10081" \
   -v /root/kuboard-data:/data \
   swr.cn-east-2.myhuaweicloud.com/kuboard/kuboard:v3
   # 也可以使用镜像 swr.cn-east-2.myhuaweicloud.com/kuboard/kuboard:v3 ，可以更快地完成镜像下载。
   # 请不要使用 127.0.0.1 或者 localhost 作为内网 IP 
   # Kuboard 不需要和 K8S 在同一个网段，Kuboard Agent 甚至可以通过代理访问 Kuboard Server \
```