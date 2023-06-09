# v2ray 客户端使用

## 下载zip

进入此链接下载linux的zip包，并解压
```shell
https://github.com/v2ray/v2ray-core/releases
```

## 简单使用

### 运行命令
```shell
/.../v2ray –config=/.../config.json
```

### config.json
zi包有默认的config.json，但是没有outbounds  
可以从windows等环境的v2ray ui工具中的某个节点copy一份outbound的json。  
对默认的json补充outbound即可，以防万一，文末贴了一份默认的config.json  

对编写完整的config.json，使用如下命令测试
```shell
v2ray -test -config config.json
```
配置文件有效会显示如下
```shell
V2Ray 4.44.0 (V2Fly, a community-driven edition of V2Ray.) Custom (go1.17.3 linux/amd64)
A unified platform for anti-censorship.
2022/02/23 22:23:52 [Info] main/jsonem: Reading config: config.json
Configuration OK.
```

运行v2ray后可执行以下命令测试
```shell
curl –-socks5 127.0.0.1:1080 https://www.google.com
```

# 注册 systemd
参考
```shell
installed: /usr/local/bin/v2ray
installed: /usr/local/share/v2ray/geoip.dat
installed: /usr/local/share/v2ray/geosite.dat
installed: /usr/local/etc/v2ray/config.json
installed: /var/log/v2ray/
installed: /var/log/v2ray/access.log
installed: /var/log/v2ray/error.log
installed: /etc/systemd/system/v2ray.service
installed: /etc/systemd/system/v2ray@.service
removed: /tmp/tmp.coUwPtRYSL
info: V2Ray v5.4.1 is installed.
You may need to execute a command to remove dependent software: yum remove curl unzip
Please execute the command: systemctl enable v2ray; systemctl start v2ray
```

创建 /etc/systemd/system/v2ray.service，填入以下
```shell
[Unit]
Description=V2Ray Service
Documentation=https://www.v2fly.org/
After=network.target nss-lookup.target

[Service]
User=root
CapabilityBoundingSet=CAP_NET_ADMIN CAP_NET_BIND_SERVICE
AmbientCapabilities=CAP_NET_ADMIN CAP_NET_BIND_SERVICE
NoNewPrivileges=true
ExecStart=/usr/local/bin/v2ray -config /usr/local/etc/v2ray/config.json
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
systemctl 命令
```shell
systemctl enable v2ray
systemctl start v2ray
systemctl stop v2ray
```

# 配置代理
方法一
修改文件“/etc/profile”，在文件结束位置增加如下内容：
```shell
# 设置http代理
export http_proxy=socks5://127.0.0.1:10808

# 设置https代理
export https_proxy=socks5://127.0.0.1:10808

# 设置ftp代理
export ftp_proxy=socks5://127.0.0.1:10808

# 17.16.x.x为我们自己的云服务器的内网IP 配置为no_proxy代表内网传输不走代理
export no_proxy="172.16.x.x"
```

执行命令生效
```shell
source /etc/profile
```

方法二（针对单用户生效）
编辑文件 vim ~/.bashrc 添加以下内容
```shell
# set proxy
function setproxy() {
    export http_proxy=socks5://127.0.0.1:10808
    export https_proxy=socks5://127.0.0.1:10808
    export ftp_proxy=socks5://127.0.0.1:10808
    export no_proxy="172.16.x.x"
}

# unset proxy
function unsetproxy() {
    unset http_proxy https_proxy ftp_proxy no_proxy
}
```

执行 source ~/.bashrc ,使得配置立即生效;  
或是关闭当前终端，重新打开，使得配置立即生效;  
在终端执行 setproxy 使代理生效  
在终端执行 unsetproxy 使代理生效  



# config.json 默认示例
```json
{
  "dns": {
    "hosts": {
      "domain:googleapis.cn": "googleapis.com"
    },
    "servers": [
      "1.1.1.1"
    ]
  },
  "inbounds": [
    {
      "listen": "127.0.0.1",
      "port": 10808,
      "protocol": "socks",
      "settings": {
        "auth": "noauth",
        "udp": true,
        "userLevel": 8
      },
      "sniffing": {
        "destOverride": [],
        "enabled": false
      },
      "tag": "socks"
    },
    {
      "listen": "127.0.0.1",
      "port": 10809,
      "protocol": "http",
      "settings": {
        "userLevel": 8
      },
      "tag": "http"
    }
  ],
  "log": {
    "loglevel": "warning"
  },
  "outbounds": [
    {
      "mux": {
        "concurrency": 8,
        "enabled": false
      },
      "protocol": "vmess",
      "settings": {
        "vnext": [
          {
            "address": "补充",
            "port": 补充,
            "users": [
              {
                "alterId": 0,
                "encryption": "",
                "flow": "",
                "id": "补充",
                "level": 8,
                "security": "auto"
              }
            ]
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "security": "",
        "wsSettings": {
          "headers": {
            "Host": "v.qq.com"
          },
          "path": "/images"
        }
      },
      "tag": "proxy"
    },
    {
      "protocol": "freedom",
      "settings": {},
      "tag": "direct"
    },
    {
      "protocol": "blackhole",
      "settings": {
        "response": {
          "type": "http"
        }
      },
      "tag": "block"
    }
  ],
  "policy": {
    "levels": {
      "8": {
        "connIdle": 300,
        "downlinkOnly": 1,
        "handshake": 4,
        "uplinkOnly": 1
      }
    },
    "system": {
      "statsOutboundUplink": true,
      "statsOutboundDownlink": true
    }
  },
  "routing": {
    "domainMatcher": "mph",
    "domainStrategy": "IPOnDemand",
    "rules": [
      {
        "ip": [
          "1.1.1.1"
        ],
        "outboundTag": "proxy",
        "port": "53",
        "type": "field"
      }
    ]
  },
  "stats": {}
}
```
