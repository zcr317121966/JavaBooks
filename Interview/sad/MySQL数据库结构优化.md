> 一个是索引优化，一个是结构优化。。。。

面试官：先来个**三范式**

我：好的

1. 第一范式：第一范式就是原子性，字段不可再分割；

- 数据库表中的所有字段都具有单一属性（即字段不能再分割了）
- 单一属性的列是由基本数据类型所构成的
- 设计出来的表都是简单的二维表

2. 第二范式：第二范式需要确保数据库表中的每一列都和主键相关，而不能只与主键的某一部分相关（主要针对联合主键而言）
3. 第三范式：所有非主键字段和主键字段之间不能产生传递依赖

[例子](https://www.jianshu.com/p/3e97c2a1687b)

**反范式设计**：反范式化就是为了性能和读取效率的考虑而适当的对数据库设计范式的要求进行违反，**而允许存在少量的数据冗余，也就是使用空间来换取（查询）时间**；比如举个例子：**比如订单表中应该保留当前购买商品的价格、商品的名称（商品的价格是会变动的，这很危险）**

因此，简单说一下**范式化设计的优缺点**：

优点：

- 可以尽量的减少数据冗余
- 范式化的更新操作比反范式化更快
- 范式化的表通常比反范式化更小

缺点：

- 对于查询需要关联多个表
- 更难进行索引优化

**反范式化设计的优缺点**：

优点：

- 可以减少表的关联
- 可以更好的进行索引优化

缺点：

- 存在数据冗余和数据维护异常
- 对数据修改需要更多的成本

**如何选择varchar和char类型**：

- varchar用于存储变长字符串，只占用的必要的存储空间。
- char类型是定长的，char类型的最大宽度为255
- 场景：varchar适用于存储很少被更新的字符串列；char适合存储长度近似的值，适合存储短字符串，适合存储经常更新的字符串

**分库分表**：

1. **垂直分表**

也就是“大表拆小表”，基于列字段进行的。一般是表中的字段较多，将不常用的， 数据较大，长度较长（比如text类型字段）的拆分到“扩展表“。 一般是针对那种几百列的大表，也避免查询时，数据量太大造成的“跨页”问题。

2. **垂直分库**

垂直分库针对的是一个系统中的不同业务进行拆分，比如用户User一个库，商品Producet一个库，订单Order一个库。 切分后，要放在多个服务器上，而不是一个服务器上。为什么？ 我们想象一下，一个购物网站对外提供服务，会有用户，商品，订单等的CRUD。没拆分之前， 全部都是落到单一的库上的，这会让数据库的单库处理能力成为瓶颈。按垂直分库后，如果还是放在一个数据库服务器上， 随着用户量增大，这会让单个数据库的处理能力成为瓶颈，还有单个服务器的磁盘空间，内存，tps等非常吃紧。所以我们要拆分到多个服务器上，这样上面的问题都解决了，以后也不会面对单机资源问题。垂直分库一定程度上能够突破IO、连接数及单机硬件资源的瓶颈。

3. **水平分表**

针对数据量巨大的单张表（比如订单表），按照某种规则（RANGE,HASH取模等），切分到多张表里面去。 但是这些表还是在同一个库中，所以库级别的数据库操作还是有IO瓶颈。不建议采用。

切分规则：

- RANGE：从0到10000一个表，10001到20000一个表；
- HASH取模：一个商场系统，一般都是将用户，订单作为主表，然后将和它们相关的作为附表，这样不会造成跨库事务之类的问题。 取用户id，然后hash取模，分配到不同的数据库上。
- 地理区域：比如按照华东，华南，华北这样来区分业务，七牛云应该就是如此。
- 时间：按照时间切分，就是将6个月前，甚至一年前的数据切出去放到另外的一张表，因为随着时间流逝，这些表的数据 被查询的概率变小，所以没必要和“热数据”放在一起，这个也是“冷热数据分离”。

4. **水平分库分表**

将单张表的数据切分到多个服务器上去，每个服务器具有相应的库与表，只是表中数据集合不同。 水平分库分表能够有效的缓解单机和单库的性能瓶颈和压力，突破IO、连接数、硬件资源等的瓶颈。

**分库分表带来的问题：**

1. **事务支持**

分库分表后，就成了分布式事务了。如果依赖数据库本身的分布式事务管理功能去执行事务，将付出高昂的性能代价； 如果由应用程序去协助控制，形成程序逻辑上的事务，又会造成编程方面的负担。

2. **join**

TODO 分库分表后表之间的关联操作将受到限制，我们无法join位于不同分库的表，也无法join分表粒度不同的表， 结果原本一次查询能够完成的业务，可能需要多次查询才能完成。 粗略的解决方法： 全局表：基础数据，所有库都拷贝一份。 字段冗余：这样有些字段就不用join去查询了。 系统层组装：分别查询出所有，然后组装起来，较复杂。

**分库分表中间件：**

Mycat 和 ShardingSphere（包括 Sharding-JDBC、Sharding-Proxy 和 Sharding-Sidecar 3 款产品）。

[对比](https://my.oschina.net/u/4318872/blog/4281049)