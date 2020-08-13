## 分布式处理框架 MapReduce

### 3.2.1 什么是MapReduce

- 源于Google的MapReduce论文(2004年12月)
- Hadoop的MapReduce是Google论文的开源实现
- MapReduce优点: 海量数据离线处理&易开发
- MapReduce缺点: 实时流式计算

### 3.2.2 MapReduce编程模型

- MapReduce分而治之的思想
  - 数钱实例：一堆钞票，各种面值分别是多少
    - 单点策略
      - 一个人数所有的钞票，数出各种面值有多少张
    - 分治策略
      - 每个人分得一堆钞票，数出各种面值有多少张
      - 汇总，每个人负责统计一种面值
    - 解决数据可以切割进行计算的应用
- MapReduce编程分Map和Reduce阶段
  - 将作业拆分成Map阶段和Reduce阶段
  - Map阶段 Map Tasks 分：把复杂的问题分解为若干"简单的任务"
  - Reduce阶段: Reduce Tasks 合：reduce
- MapReduce编程执行步骤

  - 准备MapReduce的输入数据
  - 准备Mapper数据
  - Shuffle
  - Reduce处理
  - 结果输出

- **编程模型**
  - 借鉴函数式编程方式

  - 用户只需要实现两个函数接口：

    - Map(in_key,in_value)

      --->(out_key,intermediate_value) list

    - Reduce(out_key,intermediate_value) list

      --->out_value list

  - Word Count 词频统计案例

    ![](img/image-mapreduce.png)
