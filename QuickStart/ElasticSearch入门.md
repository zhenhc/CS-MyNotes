# ElasticSearch入门

#### ElasticSearch简介

- 一个分布式的，Restful风格的搜索引擎
- 支持对各种类型的数据的检索
- 搜索速度快，可以提供实时的搜索服务。
- 便于水平扩展，每秒可以处理PB级海量数据。

#### ElasticSearch术语

- 索引，类型，文档，字段。
- 集群，节点，分片，副本。

- ElasticSearch的使用

#### Elasticsearch的配置

- 在config目录下elasticsearch.yml中修改配置文件

```yaml
# Use a descriptive name for your cluster:
#
cluster.name: nowcoder
# Path to directory where to store the data (separate multiple locations by comma):
#
path.data: E:\work\data\elasticsearch-6.4.3\data
#
# Path to log files:
#
path.logs: E:\work\data\elasticsearch-6.4.3\logs
```

#### 在cmd窗口中使用elasticsearch

```sh
C:\WINDOWS\system32>curl -X GET "localhost:9200/_cat/health?v"
epoch      timestamp cluster  status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1617120694 00:11:34  nowcoder yellow          1         1     10  10    0    0       10             0                  -                 50.0%

C:\WINDOWS\system32>curl -X GET "localhost:9200/_cat/nodes?v"
ip        heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
127.0.0.1           26          93  19                          mdi       *      KOAnGyg

C:\WINDOWS\system32>curl -X GET "localhost:9200/_cat/indices?v"
health status index       uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   discusspost aS6GDnZ4QLm2Y4AhTeBnpw   5   1          1            0      7.6kb          7.6kb
yellow open   test        DrZwHxexR6afpOqGdn8QPQ   5   1          3            0     13.7kb         13.7kb

C:\WINDOWS\system32>curl -X PUT "localhost:9200/test1"
{"acknowledged":true,"shards_acknowledged":true,"index":"test1"}
C:\WINDOWS\system32>curl -X GET "localhost:9200/_cat/indices?v"
health status index       uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   discusspost aS6GDnZ4QLm2Y4AhTeBnpw   5   1          1            0      7.6kb          7.6kb
yellow open   test        DrZwHxexR6afpOqGdn8QPQ   5   1          3            0     13.7kb         13.7kb
yellow open   test1       k0rdZvlRRpyYjIiOY2wjEg   5   1          0            0      1.1kb          1.1kb

C:\WINDOWS\system32>curl -X DELETE "localhost:9200/test1"
{"acknowledged":true}
C:\WINDOWS\system32>curl -X GET "localhost:9200/_cat/indices?v"
health status index       uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   discusspost aS6GDnZ4QLm2Y4AhTeBnpw   5   1          1            0      7.6kb          7.6kb
yellow open   test        DrZwHxexR6afpOqGdn8QPQ   5   1          3            0     13.7kb         13.7kb
```

#### Spring整合Elasticsearch

1. 引入依赖

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
        </dependency>
```

2. 配置集群的名字和集群的节点

```properties
#ElasticsearchProperties
spring.data.elasticsearch.cluster-name=nowcoder
spring.data.elasticsearch.cluster-nodes=127.0.0.1:9300
```

elasticsearch有两个端口，9200是http访问的端口，9300是TCP访问的端口

3. 在DiscussPost实体类里使用elasticsearch的注解标记

```java
package com.nowcoder.community.entity;

import org.springframework.data.annotation.Id;
import org.springframework.data.elasticsearch.annotations.Document;
import org.springframework.data.elasticsearch.annotations.Field;
import org.springframework.data.elasticsearch.annotations.FieldType;

import java.util.Date;

@Document(indexName = "discusspost",type = "_doc",shards = 6,replicas = 3)
public class DiscussPost {

    @Id
    private int id;

    @Field(type = FieldType.Integer)
    private int userId;

    // 互联网校招
    @Field(type = FieldType.Text, analyzer = "ik_max_word", searchAnalyzer = "ik_smart")
    private String title;

    @Field(type = FieldType.Text, analyzer = "ik_max_word", searchAnalyzer = "ik_smart")
    private String content;

    @Field(type = FieldType.Integer)
    private int type;

    @Field(type = FieldType.Integer)
    private int status;

    @Field(type = FieldType.Date)
    private Date createTime;

    @Field(type = FieldType.Integer)
    private int commentCount;

    @Field(type = FieldType.Double)
    private double score;
    ....省略getter,setter和toString()方法
}
```

4. 声明一个接口继承ElasticsearchRepository,不用声明任何方法和属性

```java
package com.nowcoder.community.dao.elasticsearch;

import com.nowcoder.community.entity.DiscussPost;
import org.springframework.data.elasticsearch.repository.ElasticsearchRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface DiscussPostRepository extends ElasticsearchRepository<DiscussPost,Integer> {
}
```

