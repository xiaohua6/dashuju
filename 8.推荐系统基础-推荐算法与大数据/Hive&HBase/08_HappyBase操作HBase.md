### 5.4 HappyBase操作Hbase

- 什么是HappyBase

  - **HappyBase** is a developer-friendly [Python](http://python.org/) library to interact with [Apache HBase](http://hbase.apache.org/). HappyBase is designed for use in standard HBase setups, and offers application developers a Pythonic API to interact with HBase. Below the surface, HappyBase uses the [Python Thrift library](http://pypi.python.org/pypi/thrift) to connect to HBase using its [Thrift](http://thrift.apache.org/) gateway, which is included in the standard HBase 0.9x releases.

- HappyBase 是FaceBook员工开发的操作HBase的python库, 其基于Python Thrift, 但使用方式比Thrift简单, 已被广泛应用

- 启动hbase thrift server : hbase-daemon.sh start thrift

- 安装happy base

  - pip install happybase

- 如何使用HappyBase

  - 建立连接

  ```python
  import happybase
  connection = happybase.Connection('somehost')
  ```

  - 当连接建立时, 会自动创建一个与 HBase Thrift server的socket链接. 可以通过参数禁止自动链接, 然后再需要连接是调用 [`Connection.open()`](https://happybase.readthedocs.io/en/latest/api.html#happybase.Connection.open):

  ```python
  connection = happybase.Connection('somehost', autoconnect=False)
  # before first use:
  connection.open()
  ```

  - [`Connection`](https://happybase.readthedocs.io/en/latest/api.html#happybase.Connection)  这个类提供了一个与HBase交互的入口, 比如获取HBase中所有的表:  [`Connection.tables()`](https://happybase.readthedocs.io/en/latest/api.html#happybase.Connection.tables):

  ```python
  print(connection.tables())
  ```

  - 操作表
    - Table类提供了大量API, 这些API用于检索和操作HBase中的数据。 在上面的示例中，我们已经使用Connection.tables（）方法查询HBase中的表。 如果还没有任何表，可使用Connection.create_table（）创建一个新表：

  ```python
  connection.create_table('users',{'cf1': dict()})
  ```

  - 创建表之后可以传入表名获取到Table类的实例:

    ```
    table = connection.table('mytable')
    ```

  - 查询操作

  ```python
  # api
  table.scan() #全表查询
  table.row('row_key') # 查询一行
  table.rows([row_keys]) # 查询多行
  #封装函数
  def scanQuery():
      # 创建和hbase的连接
      connection = happybase.Connection('192.168.19.137')
      #通过connection找到user表 获得table对象
      table = connection.table('user')
      filter = "ColumnPrefixFilter('username')"
      #row_start 指定起始rowkey 缩小查询范围
      #filter 添加过滤器
      for key,value in table.scan(row_start='rowkey_10',filter=filter):
          print(key,value)
      # 关闭连接
      connection.close()
  def getQuery():
      connection = happybase.Connection('192.168.19.137')
      # 通过connection找到user表 获得table对象
      table = connection.table('user')
      result = table.row('rowkey_22',columns=['base_info:username'])
      #result = table.row('rowkey_22',columns=['base_info:username'])
      result = table.rows(['rowkey_22','rowkey_16'],columns=['base_info:username'])
      print(result)
      # 关闭连接
      connection.close()
  ```

  - 插入数据

  ```python
  #api
  table.put(row_key, {'cf:cq':'value'})
  def insertData():
      connection = happybase.Connection('192.168.19.137')
      # 通过connection找到user表 获得table对象
      table = connection.table('users')
      table.put('rk_01',{'cf1:address':'beijing'})
      # 关闭连接
      for key,value in table.scan():
          print(key,value)
      connection.close()
  ```

  - 删除数据

  ```python
  #api
  table.delete(row_key, cf_list)
      
  def deleteData():
      connection = happybase.Connection('192.168.19.137')
      # 通过connection找到user表 获得table对象
      table = connection.table('users')
      table.delete('rk_01',['cf1:username'])
      # 关闭连接
      for key,value in table.scan():
          print(key,value)
      connection.close()
  ```

  - 删除表

  ```python
  #api
  conn.delete_table(table_name, True)
  #函数封装
  def delete_table(table_name):
      pretty_print('delete table %s now.' % table_name)
      conn.delete_table(table_name, True)
  ```



- 完整代码

```python
import happybase

def connectHBase():
    #创建和hbase的连接
    connection = happybase.Connection('192.168.19.137')
    #获取hbase中的所有表
    print(connection.tables())
    #关闭连接
    connection.close()

def createTable():
    connection = happybase.Connection('192.168.19.137')
    connection.create_table('users',{'cf1': dict()})
    print(connection.tables())
    connection.close()

def scanQuery():
    # 创建和hbase的连接
    connection = happybase.Connection('192.168.19.137')
    #通过connection找到user表 获得table对象
    table = connection.table('user')
    filter = "ColumnPrefixFilter('username')"
    #row_start 指定起始rowkey 缩小查询范围
    #filter 添加过滤器
    for key,value in table.scan(row_start='rowkey_10',filter=filter):
        print(key,value)
    # 关闭连接
    connection.close()
def getQuery():
    connection = happybase.Connection('192.168.19.137')
    # 通过connection找到user表 获得table对象
    table = connection.table('user')
    result = table.row('rowkey_22',columns=['base_info:username'])
    #result = table.row('rowkey_22',columns=['base_info:username'])
    result = table.rows(['rowkey_22','rowkey_16'],columns=['base_info:username'])
    print(result)
    # 关闭连接
    connection.close()
def insertData():
    connection = happybase.Connection('192.168.19.137')
    # 通过connection找到user表 获得table对象
    table = connection.table('users')
    table.put('rk_01',{'cf1:address':'beijing'})
    # 关闭连接
    for key,value in table.scan():
        print(key,value)
    connection.close()
def deleteData():
    connection = happybase.Connection('192.168.19.137')
    # 通过connection找到user表 获得table对象
    table = connection.table('users')
    table.delete('rk_01',['cf1:username'])
    # 关闭连接
    for key,value in table.scan():
        print(key,value)
    connection.close()

def deletetable():
    #创建和hbase的连接
    connection = happybase.Connection('192.168.19.137')
    #获取hbase中的所有表
    connection.delete_table('users',disable=True)
    print(connection.tables())
    #关闭连接
    connection.close()
def main():
    #connectHBase()
    #createTable()
    scanQuery()
    #getQuery()
    #insertData()
    #deleteData()
    #deletetable()
  
  if __name__ == "__main__":
      main()
```

