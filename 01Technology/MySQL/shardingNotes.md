
## product
mybatis散表 mybatis-shards
分库分表中间件: 360的atlas, MyCat
mysql开源中间件产品，Atlas，cobar，tddl [mysql中间件研究（Atlas，cobar，TDDL）](http://www.guokr.com/blog/475765/)
mycat，heisenberg,Oceanus,vitess,OneProxy [ mysql中间件研究（Atlas，cobar，TDDL，mycat，heisenberg,Oceanus,vitess,OneProxy）] (http://songwie.com/articlelist/44)
常见开源产品 Cobar 	Cobar-client 	TDDL 	Sharding-JDBC
http://www.infoq.com/cn/news/2016/01/sharding-jdbc-dangdang

## 分库分表的几种常见形式以及可能遇到的难题
丁浪 [原文](http://www.infoq.com/cn/articles/key-steps-and-likely-problems-of-split-table )

### 垂直分表
思路: 大表拆小表  
优点: 某种意义上能避免“跨页”的问题（MySQL、MSSQL底层都是通过“数据页”来存储的，“跨页”问题可能会造成额外的性能开销  
缺点: 在发展过程中拆分，则需要改写以前的查询语句，会额外带来一定的成本和风险  

### 垂直分库
思路: 按照业务模块来划分出不同的数据库，而不是像早期一样将所有的数据表都放到同一个数据库中  
优点: 有利于系统的扩展维护, 在高并发场景下，一定程度上能够突破IO、连接数及单机硬件资源的瓶颈，是大型分布式系统中优化数据库架构的重要手段  
缺点: 导致拆分后遇到很多问题（例如：跨库join，分布式事务等)  

### 水平分表
思路: 水平分表也称为横向分表，就是将表中不同的数据行按照一定规律分布到不同的数据库表中（这些表保存在同一个数据库中）, 最常见的方式就是通过主键或者时间等字段进行Hash和取模后拆分  
优点: 水平分表，能够降低单表的数据量，一定程度上可以缓解查询性能瓶颈  
缺点: 本质上这些表还保存在同一个库中，所以库级别还是会有IO瓶颈  

### 水平分库分表
思路: 水平分库分表与上面讲到的水平分表的思想相同，唯一不同的就是将这些拆分出来的表保存在不同的数据库中; 有些系统中使用的“冷热数据分离”（将一些使用较少的历史数据迁移到其他的数据库中。而在业务功能上，通常默认只提供热点数据的查询）  
优点: 在高并发和海量数据的场景下，分库分表能够有效缓解单机和单库的性能瓶颈和压力，突破IO、连接数、硬件资源的瓶颈  
缺点: 会带来一些复杂的技术问题和挑战（例如：跨分片的复杂查询，跨分片事务等）   

### 分库分表的难点
#### 跨库join的问题
基于架构规范，性能，安全性等方面考虑，一般是禁止跨库join的。那该怎么办呢？首先要考虑下垂直分库的设计问题.  
下面总结几种常见的解决思路，并分析其适用场景。

1. 全局表: 就是有可能系统中所有模块都可能会依赖到的一些表。比较类似我们理解的“数据字典”。为了避免跨库join查询，我们可以将这类表在其他每个数据库中均保存一份。同时，这类数据通常也很少发生修改（甚至几乎不会），所以也不用太担心“一致性”问题。  
2. 字段冗余: 这是一种典型的反范式设计，在互联网行业中比较常见，通常是为了性能来避免join查询。
3. 数据同步: 定时A库中的tab_a表和B库中tbl_b有关联，可以定时将指定的表做同步. 需要性能影响和数据时效性中取得一个平衡, 可以通过ETL工具使用
4. 系统层组装: 在系统层面，通过调用不同模块的组件或者服务，获取到数据并进行字段拼装。数据库设计上存在问题但又无法轻易调整的时候使用
5. 简单的列表查询的情况

#### 跨库事务（分布式事务）的问题
按业务拆分数据库之后，不可避免的就是“分布式事务”的问题。以往在代码中通过spring注解简单配置就能实现事务的，现在则需要花很大的成本去保证一致性。这里不展开介绍，感兴趣的读者可以自行参考《分布式事务一致性解决方案》，链接地址： http://www.infoq.com/cn/articles/solution-of-distributed-system-transaction-consistency  
