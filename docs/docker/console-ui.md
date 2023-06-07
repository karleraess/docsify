# 各种控制台UI工具

## grafana
都支持，可以插件安装各个中间件的ui管理，牛

## kafka-ui，仅限kafka
```shell
docker run -it -p 8080:8080 -e DYNAMIC_CONFIG_ENABLED=true provectuslabs/kafka-ui 
```

## kafka-ui-lite，包含kafka、zookeeper、redis
```shell
docker run -d -p 8889:8889 registry.cn-hangzhou.aliyuncs.com/karleraess/kafka-ui-lite:latest
```
从 https://gitee.com/karleraess/kafka-ui-lite/tree/dev/src/main/resources  拷贝两个文件
数据在 data.db

可用/kafka-ui-lite-1.2.11/conf  挂在出来，其文件夹下就是上述的两个文件
