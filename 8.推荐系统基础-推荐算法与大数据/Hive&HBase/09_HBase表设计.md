## 5.5 HBase表设计

### 1 HBase表设计特点及需要考虑的问题

- 设计HBase表时需要注意的特点
  - HBase中表的索引是通过rowkey实现的
  - 在表中是通过Row key的字典顺序来对数据进行排序的, 表中Region的划分通过起始Rowkey和结束Rowkey来决定的
  - 所有存储在HBase中的数据都是二进制字节, 没有数据类型
  - 原子性只在行内保证, HBase表中没有多行事务
  - 列族(Column Family)在表创建之前就要定义好
  - 列族中的列标识(Column Qualifier)可以在表创建后动态插入数据的时候添加
  - 不同的column family保存在不同的文件中。
- 设计HBase表时需要考虑的问题
  - Row key的结构该如何设置, Row key中又该包含什么样的信息
  - 表中应该有多少的列族
  - 列族中应该存储什么样的数据
  - 每个列族中存储多少列数据
  - 列的名字分别是什么
  - cell中应该存储什么样的信息
  - 每个cell中存储多少个版本信息
- DDI  目的是为了克服HBase架构上的缺陷(join繁琐 只有row key索引等)
  - Denormalization (反规范化, 解决join麻烦的问题)
  - Duplication (数据冗余)
  - Intelligent keys(通过row key设计实现 索引 排序对读写优化) 

### 2 HBase表设计案例: 社交应用互粉信息表

- 设计表保存应用中用户互粉的信息

  - 读场景:
    - 某用户都关注了哪些用户
    - 用户A有没有关注用户B
    - 谁关注了用户A
  - 写场景
    - 用户关注了某个用户
    - 用户取消关注了某个用户

- 设计1:

  - colunm qulifier(列名)  1:  2:

  ![](C:/Users/beibei/Desktop/%E6%96%87%E6%A1%A3%E4%BC%98%E5%8C%96/hbase/img/table1.png)

- 设计2

  - 添加了一个 count 记录当前的最后一个记录的列名

  ![](C:/Users/beibei/Desktop/%E6%96%87%E6%A1%A3%E4%BC%98%E5%8C%96/hbase/img/table2.png)

- 设计3

  - 列名 user_id

  ![](C:/Users/beibei/Desktop/%E6%96%87%E6%A1%A3%E4%BC%98%E5%8C%96/hbase/img/table3.png)

- 最终设计(DDI)

  - 解决谁关注了用户A问题
    - ① 设计一张新表, 里面保存某个用户和他的粉丝
    - ② 在同一张表中同时记录粉丝列表的和用户关注的列表, 并通过Rowkey来区分
      - 01_userid: 用户关注列表
      - 02_userid: 粉丝列表
    - 上两种设计方案的问题(事务)

**案例总结**

- Rowkey是HBase表结构设计中很重要的环节, 直接影响到HBase的效率和性能
- HBase的表结构比传统关系型数据库更灵活, 能存储任何二进制数据,无需考虑数据类型
- 利用列标识(Column Qualifier)来存储数据
- 衡量设计好坏的简单标准 是否会全表查询 