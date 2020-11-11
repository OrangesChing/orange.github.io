---
title: Canal的原理及使用
date: 2020-09-10 18:24:59
tags:
  - cannal
categories:
  - 数据库
  - 关系型数据库
  - 性能分析与优化
TODO: cannal的原理,api
---

# 简介

![img](\Canal的原理及使用\架构.png)

canal，译意为水道/管道/沟渠，主要用途是基于 MySQL 数据库增量日志解析，提供增量数据订阅和消费



# 原理

- canal 模拟 MySQL slave 的交互协议，伪装自己为 MySQL slave ，向 MySQL master 发送 dump 协议
- MySQL master 收到 dump 请求，开始推送 binary log 给 slave (即 canal )
- canal 解析 binary log 对象(原始为 byte 流)

> **主从复制的流程**
>
> ![img](\Canal的原理及使用\原理.png)
>
> 复制遵循三步过程：
>
> 1. 主服务器将更改记录到binlog中（这些记录称为binlog事件，可以通过来查看`show binary events`）
> 2. 从服务器将主服务器的二进制日志事件复制到其中继日志。
> 3. 中继日志中的从服务器重做事件随后将更新其旧数据。

# 初步使用

## Server的搭建

### MYSQL开启Binlog功能

- MySQL开启 Binlog 写入功能，配置 binlog-format 为 ROW 模式，my.ini(Windows)/my.cof(Linux) 中配置如下

  ```ini
  [mysqld]
  log-bin=mysql-bin # 开启 binlog
  binlog-format=ROW # 选择 ROW 模式
  server_id=1 # 配置 MySQL replaction 需要定义，不要和 canal 的 slaveId 重复
  ```

- 授权 canal 链接 MySQL 账号具有作为 MySQL slave 的权限, 如果已有账户可直接 grant

  ```SQL
  CREATE USER canal IDENTIFIED BY 'canal';  
  GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'canal'@'%';
  -- GRANT ALL PRIVILEGES ON *.* TO 'canal'@'%' ;
  FLUSH PRIVILEGES;
  ```

### 启动

- 下载 canal, 访问 [release 页面](https://github.com/alibaba/canal/releases) , 选择需要的包下载

- Linux解压缩 / Windows直接解压

  ```
  mkdir /tmp/canal
  tar zxvf canal.deployer-$version.tar.gz  -C /tmp/canal
  ```

- 解压完成后，进入canal目录，可以看到如下结构

    ```
    drwxr-xr-x 2 jianghang jianghang  136 2013-02-05 21:51 bin
    drwxr-xr-x 4 jianghang jianghang  160 2013-02-05 21:51 conf
    drwxr-xr-x 2 jianghang jianghang 1.3K 2013-02-05 21:51 lib
    drwxr-xr-x 2 jianghang jianghang   48 2013-02-05 21:29 logs
    ```

- 修改配置

  配置文件路径`conf/example/instance.properties`

  ```
  ## mysql serverId
  canal.instance.mysql.slaveId = 1234
  #position info，需要改成自己的数据库信息
  canal.instance.master.address = 127.0.0.1:3306 
  canal.instance.master.journal.name = 
  canal.instance.master.position = 
  canal.instance.master.timestamp = 
  #canal.instance.standby.address = 
  #canal.instance.standby.journal.name =
  #canal.instance.standby.position = 
  #canal.instance.standby.timestamp = 
  #username/password，需要改成自己的数据库信息
  canal.instance.dbUsername = canal  
  canal.instance.dbPassword = canal
  canal.instance.defaultDatabaseName =
  canal.instance.connectionCharset = UTF-8  # 数据库的编码方式对应到 java 中的编码类型
  #table regex
  canal.instance.filter.regex = .\*\\\\..\*
  ```

- 启动

  Linux

  ```
  sh bin/startup.sh
  ```

  Windows

  直接打开bin/starup.bat

  ![image-20200911091233595](\Canal的原理及使用\canal服务器启动.png)

- 查看 server 日志

  日志路径`logs/canal/canal.log`
  
  ![image-20200911091324959](\Canal的原理及使用\canal服务器启动后日志.png)

- 查看 instance 的日志

  日志路径`logs/example/example.log`
  
  ![image-20200911091559490](\Canal的原理及使用\example日志.png)

- 关闭

  linux
  
  ```
  sh bin/stop.sh
  ```

## 使用Client开发

### Jar包

```text
<dependency>
 <groupId>com.alibaba.otter</groupId>
 <artifactId>canal.client</artifactId>
 <version>1.1.3</version>
</dependency>
```

### 监听并处理消息

```java
@Component
public class CanalClient implements ApplicationRunner {


    public void handle(Message msg){
        CanalEntry.RowChange rowChange = null;
        for (CanalEntry.Entry entry : msg.getEntries()) {
            try {
                rowChange = CanalEntry.RowChange.parseFrom(entry.getStoreValue());
                List<CanalEntry.RowData> rows = rowChange.getRowDatasList();
                List<FzTaskLog> changePoolItemSubjectsLogs = new ArrayList<>();
                switch (rowChange.getEventType()){
                    case DELETE:
                        for (CanalEntry.RowData row : rows) {
                            List<CanalEntry.Column> afterColumnsList = row.getAfterColumnsList();
                            List<CanalEntry.Column> beforeColumnsList = row.getBeforeColumnsList();
                            Map<String, Object> afterColumnsDataMap = transforListToMap(afterColumnsList);
                            Map<String, Object> beforeColumnsDataMap = transforListToMap(beforeColumnsList);
         				   System.out.println(afterColumnsDataMap.toString())
                            System.out.println(beforeColumnsDataMap.toString())  
                        }
                        break;
                	case UPDATE:
                	case INSERT
                }

            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

    @Override
    public void run(ApplicationArguments args) throws Exception {
        //连接canal
        CanalConnector connector = CanalConnectors.newSingleConnector(new InetSocketAddress(AddressUtils.getHostIp(), 11111), "example", "", "");
        connector.connect();
        //订阅 监控的 数据库.表
        connector.subscribe("orange_learning.item_subject");
        while (true) {
            //一次取5条
            Message msg = connector.getWithoutAck(5);

            long batchId = msg.getId();
            int size = msg.getEntries().size();
            if (batchId < 0 || size == 0) {
                System.out.println("没有消息，休眠5秒");
                try {
                    Thread.sleep(5000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            } else {
                handle(msg);
                //确认消息
                connector.ack(batchId);
            }

        }
    }

    public static Map<String, Object> transforListToMap(List<CanalEntry.Column> afterColumnsList) {
        Map map = new HashMap();
        if (afterColumnsList != null && afterColumnsList.size() > 0) {
            for (CanalEntry.Column column : afterColumnsList) {
                map.put(column.getName(), column.getValue());
            }
        }
        return map;
    }


}
```

# Client-API

