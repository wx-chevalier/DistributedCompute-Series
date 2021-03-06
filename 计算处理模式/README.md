# 数据处理架构

# OLTP 与 OLAP

数十年来，数据和数据处理在企业中无处不在。多年来，数据的收集和使用一直在增长，公司已经设计并构建了基础架构来管理数据。大多数企业实施的传统架构区分了两种类型的数据处理：事务处理（OLTP, Online Transactional Processing）和分析处理（OLAP, Online Analytical Processing）。在互联网浪潮出现之前，企业的数据量普遍不大，特别是核心的业务数据，通常一个单机的数据库就可以保存。在业务数据处理的早期，对数据库的写入通常对应于正在进行的商业交易：进行销售，向供应商下订单，支付员工工资等等。随着数据库扩展到那些没有不涉及钱易手的业务，我们仍然使用术语交易（transaction）来代指形成一个逻辑单元的一组读写。事务不一定具有 ACID（原子性，一致性，隔离性和持久性）属性。事务处理只是意味着允许客户端进行低延迟读取和写入，而不是只能定期运行（例如每天一次）的批量处理作业。

同时，数据库也开始越来越多地用于数据分析，这些数据分析具有非常不同的访问模式。通常，分析查询需要扫描大量记录，每个记录只读取几列，并计算汇总统计信息（如计数，总和或平均值），而不是将原始数据返回给用户。早期所有的线上请求(OLTP) 和后台分析 (OLAP) 都跑在同一个数据库实例上。后来随着数据量不断增大，在二十世纪八十年代末和九十年代初期，公司倾向于在单独的数据库上运行分析，这个单独的数据库被称为数据仓库（data warehouse）。在这样的背景下，以 Hadoop 为代表的大数据技术开始蓬勃发展，它用许多相对廉价的 x86 机器构建了一个数据分析平台，用并行的能力破解大数据集的计算问题。由此，架构师把存储划分成线上业务和数据分析两个模块。如下图所示，业务库的数据通过 ETL 工具抽取出来，导入专用的分析平台。业务数据库专注提供 TP 能力，分析平台提供 AP 能力，各施其职，看起来已经很完美了。但其实这个架构也有自己的不足。

![OLTP 与 OLAP 关系图](https://s2.ax1x.com/2019/09/01/nSp7uT.jpg)

| 属性         | 事务处理 OLTP                | 分析系统 OLAP            |
| ------------ | ---------------------------- | ------------------------ |
| 主要读取模式 | 查询少量记录，按键读取       | 在大批量记录上聚合       |
| 主要写入模式 | 随机访问，写入要求低延时     | 批量导入（ETL），事件流  |
| 主要用户     | 终端用户，通过 Web 应用      | 内部数据分析师，决策支持 |
| 处理的数据   | 数据的最新状态（当前时间点） | 随时间推移的历史事件     |
| 数据集尺寸   | GB ~ TB                      | TB ~ PB                  |
