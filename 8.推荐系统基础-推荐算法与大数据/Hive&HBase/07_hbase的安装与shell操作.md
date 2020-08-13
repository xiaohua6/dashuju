## 5.3 HBase 的安装与Shell操作

### 1 HBase的安装

- 下载安装包 http://archive.cloudera.com/cdh5/cdh/5/hbase-1.2.0-cdh5.7.0.tar.gz

- 配置伪分布式环境

  - 环境变量配置

    ```shell
    export HBASE_HOME=/root/bigdata/hbase
    export PATH=$HBASE_HOME/bin:$PATH
    ```

  - 配置hbase-env.sh

    ```shell
    export JAVA_HOME=/root/bigdata/jdk
    export HBASE_MANAGES_ZK=false  --如果你是使用hbase自带的zk就是true，如果使用自己的zk就是false
    ```

  - 配置hbase-site.xml

    ```xml
    <configuration>
            <property>
                    <name>hbase.rootdir</name>
                    <value>hdfs://hadoop-master:9000/hbase</value>
            </property>
            <property>
                    <name>hbase.cluster.distributed</name>
                    <value>true</value>
            </property>
            <property>
                    <name>hbase.master</name>
                    <value>hadoop-master:60000</value>
            </property>
            <property>
                    <name>hbase.zookeeper.quorum</name>
                    <value>hadoop-master:2181</value>
            </property>
            <property>
                    <name>hbase.zookeeper.property.clientPort</name>
                    <value>2181</value>
            </property>
            <property>
                    <name>hbase.zookeeper.property.dataDir</name>
                    <value>/root/bigdata/zookeeper-3.4.14/dataDir</value>
            </property>
            <property>
                    <name>hbase.unsafe.stream.capability.enforce</name>
                <value>false</value>
            </property>
    </configuration>      
    ```

  - 启动hbase（启动的hbase的时候要保证hadoop集群已经启动）

    ```shell
    /hbase/bin/start-hbase.sh
    ```

  - 输入hbase shell（进入shell命令行）

### 2 HBase shell

- HBase DDL 和 DML 命令

<table>
  <tr>
    <th>名称</th>
    <th>命令表达式</th>
  </tr>
  <tr>
    <td> 创建表 </td>
   <td> create '表名', '列族名1','列族名2','列族名n' </td>
  </tr>
  <tr>
    <td> 添加记录 </td>
    <td> put '表名','行名','列名:','值 </td>
  </tr>
    <tr>
    <td> 查看记录 </td>
    <td> get '表名','行名' </td>
  </tr>
  <tr>
    <td> 查看表中的记录总数 </td>
    <td> count '表名' </td>
  </tr>
    <tr>
    <td> 删除记录 </td>
    <td> delete '表名', '行名','列名' </td>
  </tr>
  <tr>
    <td> 删除一张表 </td>
    <td> 第一步 disable '表名' 第二步 drop '表名' </td>
  </tr>
  <tr>
    <td> 查看所有记录 </td>
    <td> scan "表名称" </td>
  </tr>
  <tr>
    <td> 查看指定表指定列所有数据 </td>
    <td> scan '表名' ,{COLUMNS=>'列族名:列名'} </td>
  </tr>
   <tr>
    <td> 更新记录 </td>
    <td> 重写覆盖 </td>
  </tr>
</table>

- 连接集群

```
hbase shell
```

- 创建表

```sql
create 'user','base_info'
```

- 删除表

```sql
disable 'user'
drop 'user'
```

- 创建名称空间

```sql
create_namespace 'test'
```

- 展示现有名称空间

```sql
list_namespace
```

- 创建表的时候添加namespace

```sql
create 'test:user','base_info'
```

- 显示某个名称空间下有哪些表

```
list_namespace_tables 'test'
```

- 插入数据

  put  ‘表名’，‘rowkey的值’，’列族：列标识符‘，’值‘

```python
put 'user','rowkey_10','base_info:username','Tom'
put 'user','rowkey_10','base_info:birthday','2014-07-10'
put 'user','rowkey_10','base_info:sex','1'
put 'user','rowkey_10','base_info:address','Tokyo'

put 'user','rowkey_16','base_info:username','Mike'
put 'user','rowkey_16','base_info:birthday','2014-07-10'
put 'user','rowkey_16','base_info:sex','1'
put 'user','rowkey_16','base_info:address','beijing'

put 'user','rowkey_22','base_info:username','Jerry'
put 'user','rowkey_22','base_info:birthday','2014-07-10'
put 'user','rowkey_22','base_info:sex','1'
put 'user','rowkey_22','base_info:address','Newyork'

put 'user','rowkey_24','base_info:username','Nico'
put 'user','rowkey_24','base_info:birthday','2014-07-10'
put 'user','rowkey_24','base_info:sex','1'
put 'user','rowkey_24','base_info:address','shanghai'

put 'user','rowkey_25','base_info:username','Rose'
put 'user','rowkey_25','base_info:birthday','2014-07-10'
put 'user','rowkey_25','base_info:sex','1'
put 'user','rowkey_25','base_info:address','Soul'
```

- 查询表中的所有数据

```shell
scan 'user'
#HBase中一般存储数据量都很大 很少使用全表查询 scan会加上一些条件限制
```

- Scan查询中添加限制条件

```python
scan '名称空间:表名', {COLUMNS => ['列族名1', '列族名2'], LIMIT => 10, STARTROW => '起始的rowkey'}  # 通过COLUMNS  LIMIT STARTROW 等条件缩小查询范围

#LIMIT=>2 限制输出两行
scan 'user' ,{COLUMNS =>['base_info'],LIMIT=>2}
## 返回结果
ROW                        COLUMN+CELL
 rowkey_10                 column=base_info:address, timestamp=1558323139732, value=Tokyo
 rowkey_10                 column=base_info:birthday, timestamp=1558323139636, value=2014-07-10
 rowkey_10                 column=base_info:sex, timestamp=1558323139678, value=1
 rowkey_10                 column=base_info:username, timestamp=1558323918953, value=Tom4
 rowkey_16                 column=base_info:address, timestamp=1558323139963, value=beijing
 rowkey_16                 column=base_info:birthday, timestamp=1558323139866, value=2014-07-10
 rowkey_16                 column=base_info:sex, timestamp=1558323139907, value=1
 
#STARTROW 限制起始的Rowkey
scan 'user' ,{COLUMNS =>['base_info'],LIMIT=>2,STARTROW=>'rowkey_16'}
#返回结果：
ROW                        COLUMN+CELL
 rowkey_16                 column=base_info:address, timestamp=1558323139963, value=beijing
 rowkey_16                 column=base_info:birthday, timestamp=1558323139866, value=2014-07-10
 rowkey_16                 column=base_info:sex, timestamp=1558323139907, value=1
 rowkey_22                 column=base_info:address, timestamp=1558323140188, value=Newyork
 rowkey_22                 column=base_info:birthday, timestamp=1558323140107, value=2014-07-10
 rowkey_22                 column=base_info:sex, timestamp=1558323140143, value=1
 rowkey_22                 column=base_info:username, timestamp=1558323140036, value=Jerry

```

- scan查询添加过滤器

  - ROWPREFIXFILTER rowkey 前缀过滤器

  ```shell
  scan 'user', {ROWPREFIXFILTER=>'rowkey_22'}
  #显示结果
  ROW                        COLUMN+CELL
   rowkey_22                 column=base_info:address, timestamp=1558323140188, value=Newyork
   rowkey_22                 column=base_info:birthday, timestamp=1558323140107, value=2014-07-10
   rowkey_22                 column=base_info:sex, timestamp=1558323140143, value=1
   rowkey_22                 column=base_info:username, timestamp=1558323140036, value=Jerry
  1 row(s)
  Took 0.0120 seconds
  ```

- 查询某个rowkey的数据

```
get 'user','rowkey_16'
```

- 查询某个列簇的数据

```shell
get 'user','rowkey_16','base_info'
get 'user','rowkey_16','base_info:username'
get 'user', 'rowkey_16', {COLUMN => ['base_info:username','base_info:sex']}
```

- 删除表中的数据

```
delete 'user', 'rowkey_16', 'base_info:username'
```

- 清空数据

```
truncate 'user'
```

- 操作列簇

```
alter 'user', NAME => 'f2'
alter 'user', 'delete' => 'f2'
```

- HBase 追加型数据库 会保留多个版本数据

  ```sql
  desc 'user'
  Table user is ENABLED
  user
  COLUMN FAMILIES DESCRIPTION
  {NAME => 'base_info', VERSIONS => '1', EVICT_BLOCKS_ON_CLOSE => 'false', NEW_VERSION_B
  HE_DATA_ON_WRITE => 'false', DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', MI
  ER => 'NONE', CACHE_INDEX_ON_WRITE => 'false', IN_MEMORY => 'false', CACHE_BLOOM
  se', COMPRESSION => 'NONE', BLOCKCACHE => 'false', BLOCKSIZE => '65536'}
  ```

  - VERSIONS=>'1'说明最多可以显示一个版本 修改数据

  ```sql
  put 'user','rowkey_10','base_info:username','Tom'
  ```

  - 指定显示多个版本

  ```shell
  get 'user','rowkey_10',{COLUMN=>'base_info:username',VERSIONS=>2}
  ```

  - 修改可以显示的版本数量

  ```shell
  alter 'user',NAME=>'base_info',VERSIONS=>10
  ```

- 通过时间戳查询

  - 通过TIMERANGE 指定时间范围

  ```python
  scan 'user',{COLUMNS => 'base_info', TIMERANGE => [1558323139732, 1558323139866]}
  get 'user','rowkey_10',{COLUMN=>'base_info:username',VERSIONS=>2,TIMERANGE => [1558323904130, 1558323918954]}
  ```

  - 通过时间戳过滤器 指定具体时间戳的值

  ```python
  scan 'user',{FILTER => 'TimestampsFilter (1558323139732, 1558323139866)'}
  get 'user','rowkey_10',{COLUMN=>'base_info:username',VERSIONS=>2,FILTER => 'TimestampsFilter (1558323904130, 1558323918954)'}
  
  ```

  - 获取最近多个版本的数据

  ```python
  get 'user','rowkey_10',{COLUMN=>'base_info:username',VERSIONS=>10}
  
  COLUMN                           CELL
   base_info:username              timestamp=1558323918953, value=Tom4
   base_info:username              timestamp=1558323904133, value=Tom3
   base_info:username              timestamp=1558323758696, value=Tom2
   base_info:username              timestamp=1558323139575, value=Tom
  
  ```

  - 通过指定时间戳获取不同版本的数据

  ```python
  get 'user','rowkey_10',{COLUMN=>'base_info:username',TIMESTAMP=>1558323904133}
  
  #返回结果如下
  COLUMN                           CELL
   base_info:username              timestamp=1558323904133, value=Tom3
  
  get 'user','rowkey_10',{COLUMN=>'base_info:username',TIMESTAMP=>1558323918953}
  #返回结果如下
  COLUMN                           CELL
   base_info:username              timestamp=1558323918953, value=Tom4
  
  ```

  

- 命令表

![](img/2017-12-27_230420.jpg)

