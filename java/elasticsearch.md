# lucene 

lucene是一套信息检索工具包，jar包，并不包含搜索引擎系统。

包含：索引结构，读写索引的工具，排序，搜索规则，工具类。

Lucene 和 ElasticSearch关系：

ElasticSearch 是基于 Lucene 做了一些封装和增强。

# ElasticSearch

ElasticSearch，简称es，是一个开源的**高扩展**的**分布式全文搜索引擎**，它可以近乎实时的存储，检索数据；本身扩展性很好，可以扩展到上百台服务器，处PB级别的数据。es也是用Java开发并使用Lucene作为其核心来实现所有索引和搜索功能，但是他的目的是通过简单地RESTful API来隐藏Lucene的复杂性，从而让全文搜索变得更加简单。可以用于全文搜索、结构化搜索、分析以及将这三者混用。

# Solr

Solr是Apache下的一个顶级开源项目，采用Java开发，它是基于Lucene的全文搜索服务器。Solr提供了比Lucene更为丰富的查询语言，同时实现了可配置，可扩展，并对索引、搜索性能进行了优化。

Solr可以独立运行，运行在Jetty、Tomcat等等Servlet同期中，Solr索引的实现方法非常简单，用POST方法向Solr服务器发送一个描述Fleid及内容的XML文档，Solr根据XML文档添加、删除、更新索引。Solr搜索只需要发送HTTP GET请求，然后对Solr返回的XML、json等格式的查询结果解析，展示。Solr不提供构建UI的功能，提供了一个管理界面，通过管理界面可以查询Solr的配置和运行情况。

Solr是基于Lucene开发的企业级搜索服务器，实际上就是封装了Lucene。

Solr是一个独立的企业级搜索应用服务器，它对外提供类似于Web-service的API接口。用户可以通过HTTP 请求，面向搜索引擎服务器提交一定格式的文件，生成索引；也可以通过提出查找请求，并得到返回结果。

# ElasticSearch与Solr比较

单纯的对已有的数据进行操作时，Solr更快。

当实时建立索引时，Solr会产生io阻塞，性能较差，es具有明显优势。

随着数据量增加。Solr的搜索效率会变得更低，es几乎没变化。

总之：

- es开箱即用，非常简单。Solr安装略微复杂。
- Solr利用Zookeeper进行分布式管理，es自身带有分布式协调管理功能。
- Solr支持更多格式的数据，如CSV，JSON，XML，而es仅支持JSON。
- Solr官方提供的功能更多，而es本身注重核心功能，高级功能需要第三方插件提供，例如途径化界面需要kibana支撑。
- Solr查询快，但更新索引时慢，用于电商等等查询多的应用。
- es建立索引块，实时查询快，用于fackbook，微博等等搜索。
- Solr是传统搜索应用的有力解决方案，但es更适用于新兴的实时搜索应用。
- Solr比较成熟，有更大的，更成熟的用户、开发和贡献者社区，而es相对开发维护者较少，更新太快，学习使用成本较高。

# ES使用

## 目录

>bin 启动文件
>
>config 配置文件
>
>- log4j2 日志配置文件
>- jvm.options java虚拟机配置文件，这里是它自带的java虚拟机的，不是本机的
>- ElasticSearch.yml  ElasticSearch的配置文件！默认端口：9200
>
>lib jar包，依赖
>
>modules 功能模块
>
>plugins 插件
>
>logs 日志

## 启动

直接在es安装目录下的bin目录下，执行elasticsearch.bat就可以了，访问localhost:9200就可以了。

## 安装可视化插件

地址：https://github.com/mobz/elasticsearch-head

是一个前端项目。需要有node的环境，下载之后执行npm install，然后执行npm run start，访问localhost:9100就可以看到界面了。这个项目的端口在9100，如果要连接9200端口的es，会出现跨域的问题，不能连接。

解决：关闭es，配置conf/elasticsearch.yml，在末尾加上

```yml
http.cors.enabled: true
http.cors.allow-origin: "*"
```

第一行配置开启跨域请求，第二行配置可以访问的设备，这里是所有设备。然后重启就可以了。

创建索引（当做数据库）：可视化界面中新建索引。

创建文档（当做数据）

这个head项目，仅仅作为数据展示的工具，具体的数据查询用Kibana做。

## 了解ELK

ELK是ElasticSearch、Logstash、Kibana三大开源框架首字母简称。市面上也成为Elastic Stack。其中，Logstash是ELK的中央数据流引擎，用于从不同目标（文件、数据存储、MQ）手机不同格式的数据，经过过滤之后支持输出到不同目的地（文件、MQ、redis、ElasticSearch、kafka等）。Kibana可以将ElasticSearch的数据通过有好的页面展示出来，提供实时分析的功能。

市面上多开发只要提到ELK能够一直说出它是一个日志分析架构技术栈的总称，但是ELK实际上不仅仅适用于日志分析，还可以支持其他任何数据分析和收集的场景，日志分析和收集只是更具有代表性。并非唯一性。

![工作流示意图](http://oos.sfzzz.top/2020_08_01_11_48_53_1.png)

Kibana是一个针对Elasticsearch的开源分析及可视化平台，用来搜索、查看交互存储在Elasticsearch索引中的数据。使用Kibana，可以通过各种图表进行高级数据分析及展示。Kibana让海量数据更容易理解。它操作简单，基于浏览器的用户界面可以快速创建仪表板（dashboard）实时显示Elasticsearch查询动态。设置Kibana非常简单。无需编码或者额外的基础架构，几分钟内就可以完成Kibana安装并启动Elasticsearch索引监测。

下载解压Kibana，进入目录bin，执行kibana.bat即可

汉化：修改配置文件Kibana.yml，最后加上

```yml
i18n.locale: "zh-CN"
```

重启即可。

# ES核心概念

集群、节点、索引、类型、文档、分片、映射是什么？

ElasticSearch是面向文档、关系型数据库 