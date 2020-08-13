## 4.2 Hive 基本操作

### 1 Hive HQL操作初体验

- 创建数据库

  ```sql
  CREATE DATABASE test;
  ```

- 显示所有数据库

  ```sql
  SHOW DATABASES;
  ```

- 创建表

  ```sql
  CREATE TABLE student(classNo string, stuNo string, score int) row format delimited fields terminated by ',';
  ```

  - row format delimited fields terminated by ','  指定了字段的分隔符为逗号，所以load数据的时候，load的文本也要为逗号，否则加载后为NULL。hive只支持单个字符的分隔符，hive默认的分隔符是\001

- 将数据load到表中

  - 在本地文件系统创建一个如下的文本文件：/home/hadoop/tmp/student.txt

    ```
    C01,N0101,82
    C01,N0102,59
    C01,N0103,65
    C02,N0201,81
    C02,N0202,82
    C02,N0203,79
    C03,N0301,56
    C03,N0302,92
    C03,N0306,72
    ```

  - ```sql
    load data local inpath '/home/hadoop/tmp/student.txt'overwrite into table student;
    ```

  - 这个命令将student.txt文件复制到hive的warehouse目录中，这个目录由hive.metastore.warehouse.dir配置项设置，默认值为/user/hive/warehouse。Overwrite选项将导致Hive事先删除student目录下所有的文件, 并将文件内容映射到表中。
    Hive不会对student.txt做任何格式处理，因为Hive本身并不强调数据的存储格式。

- 查询表中的数据 跟SQL类似

  ```sql
  hive>select * from student;
  ```

- 分组查询group by和统计 count

  ```sql
  hive>select classNo,count(score) from student where score>=60 group by classNo;
  ```

  从执行结果可以看出 hive把查询的结果变成了MapReduce作业通过hadoop执行

### 2 Hive的内部表和外部表

<table>
  <tr>
    <th></th>
    <th>内部表(managed table)</th>
    <th>外部表(external table)</th>
  </tr>
  <tr>
    <td> 概念 </td>
    <td> 创建表时无external修饰 </td>
    <td> 创建表时被external修饰 </td>
  </tr>
  <tr>
    <td> 数据管理 </td>
    <td> 由Hive自身管理 </td>
    <td> 由HDFS管理 </td>
  </tr>
  <tr>
    <td> 数据保存位置 </td>
    <td> hive.metastore.warehouse.dir  （默认：/user/hive/warehouse） </td>
    <td> hdfs中任意位置 </td>
  </tr>
  <tr>
    <td> 删除时影响 </td>
    <td> 直接删除元数据（metadata）及存储数据 </td>
    <td> 仅会删除元数据，HDFS上的文件并不会被删除 </td>
  </tr>
  <tr>
    <td> 表结构修改时影响 </td>
    <td> 修改会将修改直接同步给元数据  </td>
    <td> 表结构和分区进行修改，则需要修复（MSCK REPAIR TABLE table_name;）</td>
  </tr>
</table>

- 案例

  - 创建一个外部表student2

  ```sql
  CREATE EXTERNAL TABLE student2 (classNo string, stuNo string, score int) row format delimited fields terminated by ',' location '/tmp/student';
  ```

  - 装载数据

    ```sql
    load data local inpath '/root/tmp/student.txt' overwrite into table student2;
    ```

- 显示表信息

  ```sql
  desc formatted table_name;
  ```

- 删除表查看结果

  ```sql
  drop table student;
  ```

- 再次创建外部表 student2

- 不插入数据直接查询查看结果

  ```sql
  select * from student2;
  ```

### 3 分区表

- 什么是分区表

  - 随着表的不断增大，对于新纪录的增加，查找，删除等(DML)的维护也更加困难。对于数据库中的超大型表，可以通过把它的数据分成若干个小表，从而简化数据库的管理活动，对于每一个简化后的小表，我们称为一个单个的分区。
  - hive中分区表实际就是对应hdfs文件系统上独立的文件夹，该文件夹内的文件是该分区所有数据文件。
  - 分区可以理解为分类，通过分类把不同类型的数据放到不同的目录下。
  - 分类的标准就是分区字段，可以一个，也可以多个。
  - 分区表的意义在于优化查询。查询时尽量利用分区字段。如果不使用分区字段，就会全部扫描。

- 创建分区表

  ```shell
  tom,4300
  jerry,12000
  mike,13000
  jake,11000
  rob,10000
  ```

  

  ```sql
  create table employee (name string,salary bigint) partitioned by (date1 string) row format delimited fields terminated by ',' lines terminated by '\n' stored as textfile;
  ```

- 查看表的分区

  ```sql
  show partitions employee;
  ```

- 添加分区

  ```
  alter table employee add if not exists partition(date1='2018-12-01');
  ```

- 加载数据到分区

  ```
  load data local inpath '/root/tmp/employee.txt' into table employee partition(date1='2018-12-01');
  ```

- 如果重复加载同名文件，不会报错，会自动创建一个*_copy_1.txt

- 外部分区表即使有分区的目录结构, 也必须要通过hql添加分区, 才能看到相应的数据

  ```shell
  hadoop fs -mkdir /user/hive/warehouse/employee/date1=2018-12-04
  hadoop fs -copyFromLocal /tmp/employee.txt /user/hive/warehouse/test.db/employee/date1=2018-12-04/employee.txt
  ```

  - 此时查看表中数据发现数据并没有变化, 需要通过hql添加分区

    ```
    alter table employee add if not exists partition(date1='2018-12-04');
    ```

  - 此时再次查看才能看到新加入的数据

- 总结

  - 利用分区表方式减少查询时需要扫描的数据量
    - 分区字段不是表中的列, 数据文件中没有对应的列
    - 分区仅仅是一个目录名
    - 查看数据时, hive会自动添加分区列
    - 支持多级分区, 多级子目录

### 4 动态分区

- 在写入数据时自动创建分区(包括目录结构)

- 创建表

  ```
  create table employee2 (name string,salary bigint) partitioned by (date1 string) row format delimited fields terminated by ',' lines terminated by '\n' stored as textfile;
  ```

- 导入数据

  ```sql
  insert into table employee2 partition(date1) select name,salary,date1 from employee;
  ```

- 使用动态分区需要设置参数

  ```shell
  set hive.exec.dynamic.partition.mode=nonstrict;
  ```