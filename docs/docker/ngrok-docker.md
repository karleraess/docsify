# 1.centos 7.9.2009 快速开始
## 服务端启动docker
```shell
sudo docker run -d \
	--name=ngrok \
	-p 80:80 \
	-p 443:443 \
	-p 4443:4443 \
	-v /root/ngrok:/ngrok \
	registry.cn-hangzhou.aliyuncs.com/karleraess/ngrok:xxx \
	/ngrok/bin/ngrokd \
	-domain="ngrok.karleraess.top" \
	-httpAddr=":80" \
	-httpsAddr=":443"
	# 自行修改挂载点，客户端在/ngrok/bin下
	# domain的域名要和镜像中编译的域名一致
	# httpAddr的端口要和-p中http的三个端口一致，例如-p 18080:18080 -httpAddr=":18080"，httpsAddr同理
```

## 客户端启动
### 下载客户端
从/root/ngrok/bin下载对应系统的客户端ngrok

### 配置ngrok.cfg
在ngrok客户端同目录下创建ngrok.cfg
```shell
server_addr: ngrok.karleraess.top:4443
trust_host_root_certs: false
tunnels:
 http:
  subdomain: web
  proto:
   http: 80
```

### 启动命令
进入 ngrok 目录
```shell
./ngrok -config ngrok.cfg start-all

# 后台运行
nohup ./ngrok -config ngrok.cfg start-all >./nohup.out 2>&1&
```
### centos 7.9.2009 开机自动启动
1.vi /etc/rc.d/rc.local

2.末尾添加命令
```shell
# 后台运行
nohup 绝对路径/ngrok -config 绝对路径/ngrok.cfg start-all >绝对路径/nohup.out 2>&1&
```


# 2.自行制作
## 创建go基础镜像，包含ngrok源码

### Dockerfile
```shell
FROM golang:1.14.15-alpine
RUN apk add --no-cache git make openssl tzdata && \
	cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    git clone https://gitee.com/OtherCopy/ngrok.git --depth=1 /ngrok && \
	git clone -- https://gitee.com/mirrors/log4go.git /ngrok/src/github.com/alecthomas/log4go && \
	git clone -- https://gitee.com/ngrok-install/websocket.git /ngrok/src/github.com/gorilla/websocket && \
	git clone -- https://gitee.com/ngrok-install/go-vhost.git /ngrok/src/github.com/inconshreveable/go-vhost && \
	git clone -- https://gitee.com/ngrok-install/mousetrap.git /ngrok/src/github.com/inconshreveable/mousetrap && \
	git clone -- https://gitee.com/ngrok-install/go-bindata.git /ngrok/src/github.com/jteeuwen/go-bindata && \
	git clone -- https://gitee.com/mirrors_addons/osext.git /ngrok/src/github.com/kardianos/osext && \
	git clone -- https://gitee.com/ngrok-install/binarydist.git /ngrok/src/github.com/kr/binarydist && \
	git clone -- https://gitee.com/GoLibs/go-runewidth.git /ngrok/src/github.com/mattn/go-runewidth && \
	git clone -- https://gitee.com/ngrok-install/termbox-go.git /ngrok/src/github.com/nsf/termbox-go && \
	git clone -- https://gitee.com/mirrors/go-metrics.git /ngrok/src/github.com/rcrowley/go-metrics && \
	git clone -b v0 https://gitee.com/gityongjie/go-update.git /usr/local/ngrok/src/gopkg.in/inconshreveable/go-update.v0 && \
	git clone -b v0 https://gitee.com/gityongjie/go-update.git /usr/local/go/src/gopkg.in/inconshreveable/go-update.v0 && \
	git clone -b v1 https://gitee.com/thorbo/yaml /usr/local/ngrok/src/gopkg.in/yaml.v1 && \
	git clone -b v1 https://gitee.com/thorbo/yaml /usr/local/go/src/gopkg.in/yaml.v1
```
> 已制作好的镜像
>> registry.cn-hangzhou.aliyuncs.com/karleraess/ngrok-base:latest

### 镜像编译命令
```shell
# 镜像名称和版本号自行修改
docker build -t registry.cn-hangzhou.aliyuncs.com/karleraess/ngrok-base:xxxx .
```

## 制作ngrok镜像
### Dockerfile
```shell
# 基础镜像自行修改
FROM registry.cn-hangzhou.aliyuncs.com/karleraess/ngrok-base:xxxx .
ADD build.sh /
RUN sh /build.sh
EXPOSE 80
VOLUME [ "/ngrok" ]
CMD [ "/ngrok/bin/ngrokd"]
```

### build.sh
```shell
# NGROK_DOMAIN的域名自行修改
export NGROK_DOMAIN="xxx"
cd /ngrok/

# 为域名生成证书
openssl genrsa -out rootCA.key 2048
openssl req -x509 -new -nodes -key rootCA.key -subj "/CN=$NGROK_DOMAIN" -days 5000 -out rootCA.pem
openssl genrsa -out server.key 2048
openssl req -new -key server.key -subj "/CN=$NGROK_DOMAIN" -out server.csr
openssl x509 -req -in server.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out server.crt -days 5000

# copy生成的证书到指定目录，编译需要
cp rootCA.pem assets/client/tls/ngrokroot.crt
cp server.crt assets/server/tls/snakeoil.crt
cp server.key assets/server/tls/snakeoil.key

#linux server
GOOS=linux GOARCH=amd64 make release-server

#linux client
GOOS=linux GOARCH=arm make release-client
GOOS=linux GOARCH=386 make release-client
GOOS=linux GOARCH=amd64 make release-client
#window client
GOOS=windows GOARCH=386 make release-client
GOOS=windows GOARCH=amd64 make release-client
#mac client
GOOS=darwin GOARCH=amd64 make release-client
GOOS=darwin GOARCH=386 make release-client
GOOS=darwin GOARCH=amd64 make release-client
```
> 已制作好的，域名ngrok.karleraess.top
>> registry.cn-hangzhou.aliyuncs.com/karleraess/ngrok:ngrok.karleraess.top

### Docker 启动命令
```shell
sudo docker run -d \
	--name=ngrok \
	-p 80:80 \
	-p 443:443 \
	-p 4443:4443 \
	-v xxx:/ngrok \
	registry-vpc.cn-hangzhou.aliyuncs.com/karleraess/ngrok:xxx \
	/ngrok/bin/ngrokd \
	-domain="xxx" \
	-httpAddr=":xxx" \
	-httpsAddr=":443"
	# 容器的4443端口不可更改，如-p 14443:4443
	# 自行修改挂载点，客户端在/ngrok/bin下
	# domain的域名要和镜像中编译的域名一致
	# httpAddr的端口要和-p中http的三个端口一致，例如-p 18080:18080 -httpAddr=":18080"，httpsAddr同理
```
##客户端使用
### 下载客户端
从docker容器的挂在点中下载对应系统的客户端ngrok
> 因为go镜像是alpine系统，linux 64的客户端在centos下会报错，可使用32位的客户端

### 配置ngrok.cfg
在ngrok客户端同目录下创建ngrok.cfg
```shell
# server_addr的端口最终需要走到docker容器内的4443端口
server_addr: 域名:端口
trust_host_root_certs: false
tunnels:
 # tcp 的remote_port配置方式，注意端口需要可以映射到容器内端口
 # 访问是 域名:端口，如域名:1122，域名不可加前缀
 ssh:
  remote_port: 1122
  proto:
   tcp: 22
 ftp:
  remote_port: 20
  proto:
   tcp: 20
 ftp2:
  remote_port: 21
  proto:
   tcp: 21
 # http/https 的subdomain配置方式，访问是 前缀.域名:端口，如web.域名:端口，端口最终需要走到docker定义的httpAddr/httpsAddr
 http:
  subdomain: web
  proto:
   http: 80
 http:
  subdomain: webs
  proto:
   https: 443
```

### 客户端启动
#### 启动命令
进入 ngrok 目录
```shell
./ngrok -config ngrok.cfg start-all

# 后台运行
nohup ./ngrok -config ngrok.cfg start-all >./nohup.out 2>&1&
```
#### centos 7.9.2009 开机自动启动
1.vi /etc/rc.d/rc.local

2.末尾添加命令
```shell
# 后台运行
nohup 绝对路径/ngrok -config 绝对路径/ngrok.cfg start-all >绝对路径/nohup.out 2>&1&
```