# Spring 的事务管理

## 1、何为事务

根据百度百科，事务（Transaction），一般是指要做的或所做的事情。 可以理解成是一个完整动作，假设你去银行取钱，那这个过程发生的一系列动作，包括取钱、银行账户余额减少，这两个动作必须是都要执行的，缺一不可。那么这个过程就是事务，或者说事务是一系列动作的综合。



事务通常有4个特性：

- 原子性：事务是一个原子操作，由一系列动作组成。事务的原子性确保动作要么全部完成，要么完全不起作用。 
- 一致性：一旦事务完成（不管成功还是失败），系统必须确保它所建模的业务处于一致的状态，而不会是部分完成部分失败。在现实中的数据不应该被破坏。 
- 隔离性：可能有许多事务会同时处理相同的数据，因此每个事务都应该与其他事务隔离开来，防止数据损坏。 
- 持久性：一旦事务完成，无论发生什么系统错误，它的结果都不应该受到影响，这样就能从任何系统崩溃中恢复过来。通常情况下，事务的结果被写到持久化存储器中。 



## 2、 Spring 中的事务

Spring 并不直接管理事务，而是提供事务管理器，具体由相关持久化厂商（例如 JDBC ）实现。

在 Spring 中，事务管理器为：

~~~java
Public interface PlatformTransactionManager(){  
    // 由TransactionDefinition得到TransactionStatus对象
    TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException; 
    // 提交
    Void commit(TransactionStatus status) throws TransactionException;  
    // 回滚
    Void rollback(TransactionStatus status) throws TransactionException;  
    } 
~~~

~~~java
// TransactionDefinition 定义
public interface TransactionDefinition {
    int getPropagationBehavior(); // 返回事务的传播行为
    int getIsolationLevel(); // 返回事务的隔离级别，事务管理器根据它来控制另外一个事务可以看到本事务内的哪些数据
    int getTimeout();  // 返回事务必须在多少秒内完成
    boolean isReadOnly(); // 事务是否只读，事务管理器能够根据这个返回值进行优化，确保事务是只读的
}  
~~~



## 3 、Spring 事务的传播行为

~~~java
public interface TransactionDefinition {
    // Spring 的事务传播行为类型(PROPAGATION_XX)
    int PROPAGATION_REQUIRED = 0; 
    int PROPAGATION_SUPPORTS = 1;
    int PROPAGATION_MANDATORY = 2;
    int PROPAGATION_REQUIRES_NEW = 3;
    int PROPAGATION_NOT_SUPPORTED = 4;
    int PROPAGATION_NEVER = 5;
    int PROPAGATION_NESTED = 6;
 ｝
~~~
根据事务传播行为的不同类型分为3种情况，一种是支持当前事务，一种是不支持当前事务，一种是其他情况

| 事务传播行为（支持当前事务） | 当前存在事务 |    当前不存在事务    |
| :--------------------------: | :----------: | :------------------: |
|     PROPAGATION_REQUIRED     |  加入该事务  |     创建一个事务     |
|     PROPAGATION_SUPPORTS     |  加入该事务  | 以非事务行为继续执行 |
|    PROPAGATION_MANDATORY     |  加入该事务  |       抛出异常       |

| 事务传播行为（不支持当前事务） |                      当前存在事务                      |    当前不存在事务    |
| :----------------------------: | :----------------------------------------------------: | :------------------: |
|    PROPAGATION_REQUIRES_NEW    |           挂起当前事务，再重新创建一个新事务           |     创建一个事务     |
|   PROPAGATION_NOT_SUPPORTED    | 挂起当前事务，然后以非事务行为继续运行（不创建新事务） | 以非事务行为继续运行 |
|       PROPAGATION_NEVER        |                        抛出异常                        | 以非事务行为继续运行 |

| 事务传播行为（其他情况） |               当前存在事务               | 当前不存在事务 |
| :----------------------: | :--------------------------------------: | :------------: |
|    PROPAGATION_NESTED    | 创建一个事务作为当前事务的嵌套事务来运行 |  创建一个事务  |

3种情况不同的地方其实就是对当前存在事务的条件下，做出何种选择？

支持当前事务是加入当前事务；

不支持当前事务是挂起当前事务或者抛出异常；

而最后一种是创建一个嵌套事务。



## 4、Spring 事务的隔离级别

~~~java
public interface TransactionDefinition}{
    // 事务的隔离级别（ISOLATION_XX）
    int ISOLATION_DEFAULT = -1;
    int ISOLATION_READ_UNCOMMITTED = 1;
    int ISOLATION_READ_COMMITTED = 2;
    int ISOLATION_REPEATABLE_READ = 4;
    int ISOLATION_SERIALIZABLE = 8;
}
~~~

隔离即隔离事务，防止数据损坏。隔离级别定义了一个事务可能受其他并发事务影响的程度。 

在数据库中，我们知道对一个数据并发操作下，可能会出现 脏读、不可重复读、幻读、这些问题

- 脏读：A事务写入数据100，B事务读取数据100（此时A还未提交），A事务回滚操作，那么刚才B读取到的数据就是无效的，是脏数据。
- 不可重复读：A事务写入数据100，并且提交了操作。B事务读取数据100，此时A事务修改数据成50，并且提交操作。此时B事务再次读取变为50，出现不可重复读的情况。（这里B的第一次读取和第二次读取要理解成是同一次操作，即第一次读完数据后并没有结束这次事务）
- 幻读：假设事务A查询成绩大于80分的人数，得到第一次查询的结果，此时事务B新增一条满足条件的数据，（同理事务A第一次读取完数据后并没有结束这次事务），那么事务A再次读取得到的结果就跟第一次读取得到的结果不同。

不可重复读和幻读的区别：虽然都是多次查询得到不同的结果。不过**不可重复读重点在于修改**，而**幻读重点在于增加和删除**。或者说不可重复读面向的是一条记录（查询某确切数据），而幻读面向的是一个范围（查询某范围内）。

知道可能导致的一些问题后，再来看 Spring 的不同隔离级别会导致哪些问题

|          隔离级别          |                             含义                             |
| :------------------------: | :----------------------------------------------------------: |
|     ISOLATION_DEFAULT      |                 使用后端数据库默认的隔离级别                 |
| ISOLATION_READ_UNCOMMITTED |  允许读取尚未提及的数据，**会导致脏读、不可重复读或者幻读**  |
|  ISOLATION_READ_COMMITTED  | 允许读取并发事务已经提交的数据，**可以阻止脏读，但是幻读或不可重复读仍有可能发生** |
| ISOLATION_REPEATABLE_READ  | 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，**可以阻止脏读和不可重复读，但幻读仍有可能发生。** |
|   ISOLATION_SERIALIZABLE   | 完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰。**该可以防止脏读、不可重复读以及幻读**。但是严重影响程序的性能。 |

## 5、Spring 事务超时

当事务运行时间过长时，可能会严重影响其他事务的执行，导致出现等待时间过长，浪费资源的现象。所以会设置一个时间，超过这个时间就会回滚事务，不继续等待结果，这就是事务超时。



## 6、 Spring 事务只读

如果事务是只读的话，数据库能够针对事务的只读特性做一些优化。