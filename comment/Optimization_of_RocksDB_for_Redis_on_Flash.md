## 概要
这篇论文主要是介绍ROF（Redis On Flash）所使用Rocksdb在数据迁移场景的调优，测试环境分别是EC2和GCE。

## 精要
- 目的：减小数据库复制的时间
- 方法：
  - 1. 增加并发度来消除瓶颈: 增加Rocksdb后台线程数和Redis-IO线程数
  - 2. 吞吐和延迟更稳定: 避免频繁Compaction让性能更稳定, 减少长尾效应
  - 3. 产生更大的吞吐：Compaction预读、增加Redis-IO线程（1写多读）
  - 4. 优化读取速度：顺序读vs随机读
- 度量： 每次优化，都会测量四种工作负载场景
  - 只写，写50M key(1k value)，总共50GB，这个压力代表所当前常用的数据库规模
  - 只读，读10%的数据集
  - benchmark，混合读写，50-50
  - 数据库复制，从主节点读50GB写入从节点
- 手段
分析服务器的活动：统计每次调优过程中的运行时间、吞吐量和延迟。我们也检测各项系统指标，Redis和RocksDB的线程负载、IO状态、RocksDB的每层的状态、放大因子、放慢速度和写停止(slowdowns、write stalls)。
- 负优化结果
  - bulk load模式
  - 同步写设置bytes_per_sync
  - 调整block_size参数
