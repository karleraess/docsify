# helm 安装

```shell
hostnamectl set-hostname master
yum install wget -y
wget https://get.helm.sh/helm-v3.3.1-linux-amd64.tar.gz
tar -zxvf helm-v3.3.1-linux-amd64.tar.gz
cp linux-amd64/helm /usr/local/bin/
yum install vim -y
vim /etc/profile
# 末尾写入内容
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
# esc :wq 退出保存 执行
source /etc/profile
# 查看版本
helm version
```