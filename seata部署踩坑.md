# Seata 部署踩坑

## 下载 & 运行

seata 初始化配置文件极其难用，非常推荐通过映射文件的方式直接修改 seata 启动用的 yaml 配置

1. 

```shell
docker run -d --restart=unless-stopped -e SEATA_IP=192.168.1.11必须填本机的地址，很重要 -p 7091:7091 -p 8091:8091 -v "电脑自己的本地路径:/seata-server/resources" --name=seata-server --net=你的docker虚拟网段名 seataio/seata-server:latest
```

2. 修改映射文件夹内的 application.yml

   ```yaml
   seata:
     config:
       # support: nacos, consul, apollo, zk, etcd3
       type: file
     registry:
       # support: nacos, eureka, redis, zk, consul, etcd3, sofa
       type: nacos
       nacos:
         application: seata-server
         server-addr: smart-nacos:8848 # 必须和 nacos 在同一个虚拟网段才能这么写，并且不能用默认的 bridge，不然只能写 ip
         group: "DEFAULT_GROUP"
         namespace: ""
         username: ""
         password: ""
   ```

3. 重启容器

4. 微服务模块添加依赖

   ```xml
   <dependency>
       <groupId>com.alibaba.cloud</groupId>
       <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
   </dependency>
   ```

5. 在微服务模块内的配置文件加上 seata 相关配置

   ```yaml
   seata:
     registry:
       type: nacos
       nacos:
         server-addr: smart-nacos:8848 #这里我设置了 host 映射 smart-nacos 到了 127.0.0.1 ，懒得映射的话直接 127.0.0.1 就行
         namespace: ""
         group: "DEFAULT_GROUP"
         application: seata-server
         username: ""
         password: ""
     tx-service-group: default_tx_group
     service:
       vgroup-mapping:
         default_tx_group: default
   ```

6. 重启微服务和 seata ，观察 seata 内日志是否出现 success ，例如 `2024-07-28 18:52:45 18:52:45.124  INFO --- [ttyServerNIOWorker_1_3_32] [rocessor.server.RegTmProcessor] [      onRegTmMessage]  [] : TM register success,message:RegisterTMRequest{version='2.0.0', applicationId='user-service', transactionServiceGroup='default_tx_group', extraData='ak=null`





