单库模式：一个mysql数据库承载所有相关数据。

读写分离集群模式：在原有的基础上增加中间层，与后端数据集构成读写分离的集群。整体基础结构：原有的主库派生出字库1，字库2，
利用mysql原有的主从同步机制（即为：binlog日志同步），将主库的数据变化在从库中复现，保证数据同步。主库一般用于写入处理，
从库负责读取。细节：如果直接面对主库进行操作无法完成读写分离，需要在前端分配分片中间件（阿里mycat，京东ShardingSphere）,
该中间件通过curd请求,来决定由哪个库处理。MHA中间件实现高可用（即：主服务器坏了，MHA中间件可以将某个从表提升为主服务器）。
所有节点数据均保持同步。适用于读多写少，单表不过千万的互联网应用。

分库分表（分片）集群模式：一个mysql数据库撑不住的情况下。将数据库的数据分到不同的节点数据库（即：节点数据库的数据合起来为完整的数据体）。
需要用到中间件进行路由。（对sql进行解析，将请求发到对应的数据库，分发请求的过程叫路由）。不具备高可用性。

分片算法：
1.范围法，对主键（即为分片键进行划分。如id），mysql默认提供的特性（分区表为典型的范围法），易扩展，适用范围检索，但数据不均匀，局部负载大，适用流水账应用。
2.HASH法，对id取模。取模和一致性Hash（独特的环形算法）。数据分布均匀。扩展复杂，数据迁移大。建议提前部署。

主流模式：读写分离和分片的组合运用









省流助手003：为什么大厂做垂直分表?
一张表的字段太多需要做垂直分表。

什么是水平分表？
以行为单位对数据进行拆分（范围法，hash法）。特点：所有的表结构完全相同。用于解决数据量大的存储问题。

什么是垂直分表？
将表按列拆分成2张以上的小表，通过主外键关联获取数据。

为什么要这么做？
需要了解mysql的InnoDB处理引擎。
行数据称为：row
管理数据基本单位称为页：page；每一页的默认大小：16k
保存页的单位称为区：Extent。
关系：区由连续页组成，页由连续行组成。1024/16=64(即：一个1M的区有64个页)

InnoDB1.0后新特性，压缩页。
压缩页：对数据底层进行压缩，使实际大小小于逻辑大小。
在跨页检索数据的过程中，压缩和解压缩的效率低。在表设计时，尽可能在页内多存储行数据，减少跨页检索，增加页内检索。

分析：
1行数据为1K，1页16K，即1页16条数据，1亿的数据需要625万页
垂直分页后，1行数据为64字节（1K=1024字节），即1页256条数据，1亿的数据需要39万页。分页后的数据根据id等关系进行快速提取。
通过将重要字段单独剥离成小表，让每页容纳更多行数据，页减少后，缩小数据扫描范围，达到提高执行效率的目的。

垂直分表条件：
1.单表数据达千万
2.字段超20个，且包含vachar，CLOB,BLOB等字段

字段放大小表的依据：