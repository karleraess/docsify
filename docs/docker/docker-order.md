# docker 相关命令

## 运行容器
~~~shell
sudo docker run \
    # 后台运行
    -d \
    --name=ngrok \
    -p 80:80 \
    
    -v /root/ngrok:/ngrok \

~~~

## 清理
~~~shell
# 清除没有被容器使用的镜像文件
docker image prune -af

# 清除多余的数据，包括停止的容器、多余的镜像、未被使用的volume等等
docker system prune -f
~~~

ps：参考清理脚本
~~~shell
#! /bin/bash 

need_clean() {
    used=`df -h /var/lib/docker | awk -F"[ %]+" '/dev/{print $5}'`
    if [[ $used -ge 80 ]]; then
        return 0
    fi
    
    return 1

}

if need_clean; then
    docker image prune -af
    if need_clean; then
        docker system prune -f
    fi
fi
~~~

这里通过awk获取df -h的结果中的占用比例那一栏。-F"[ %]+"的意思是指定若干个空格或%为分隔符，从而将百分数中的%剔除。  
只有当占用比例大于80%时才执行清除命令。如果执行第一条命令后降到了80%以下，则不用再执行下一条命令。
