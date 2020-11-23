# 1 安装

## 1.1 单机安装

[官网](https://www.elastic.co/cn/downloads/elasticsearch)下载之后，解压到本地即可。

目录介绍：

| 目录    | 作用                                 |
| ------- | ------------------------------------ |
| bin     | 可执行文件，启动脚本等等就是在这里面 |
| config  | 配置文件，主要的配置文件就是这个目录 |
| data    | 存放数据的文件夹                     |
| lib     | 外部依赖库                           |
| logs    | 日志文件夹                           |
| modules | 模块文件夹                           |
| plugins | 插件文件夹                           |

## 1.2 HEAD插件安装

HEAD插件是一个用于ElasticSearch可视化的前端项目。可以在浏览器端安装，也可以本地node运行。

### 1.2.1 浏览器插件直接安装

Chrome应用商店直接搜索ElasticSearch Head下载安装即可。

### 1.2.2 下载插件安装

github搜索ElasticSearch Head下载即可。

## 1.3 分布式安装

### 主机

解压之后，打开config/elasticsearch.yaml配置文件做如下配置：

```yaml
# 配置集群名和节点名
cluster.name: sufu
node.name: master

# 开启跨域请求
http.cors.enabled: true
http.cors.allow-origin: "*"

# 设置当前节点为master，设置网络
node.master: true
network.host: 127.0.0.1
```

配置完成之后，重启master。

### 从机

这里配置一主二从的集群，将es的压缩包解压两份，分别命名为salve01，salve02，代表两个从机。然后分别做如下配置：

```yaml
cluster.name: sufu
node.name: slave01
network.host: 127.0.0.1
http.port: 9201
discovery.zen.ping.unicast.hosts: ["127.0.0.1"]
```

```yaml
cluster.name: sufu
node.name: slave02
network.host: 127.0.0.1
http.port: 9202
discovery.zen.ping.unicast.hosts: ["127.0.0.1"] # 配置在哪里寻找主机
```

注意：一个集群的所有节点**集群名**必须保持一致

然后分别启动两个从机，启动之后就可以在head插件上查看集群信息。

![image-20201118230213311](C:\Users\sufu\Desktop\笔记\Java\Typora_Img\image-20201118230213311.png)

## 1.4 Kibana安装

Kibana是一个Elastic公司针对es的分析以及数据可视化平台，可以搜索，查看存放在es中的数据。