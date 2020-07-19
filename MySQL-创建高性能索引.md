## 目录

- [MySQL-创建高性能索引](#mysql--------)
  * [一、索引基础](#------)
  * [二、高性能的索引策略](#----------)
    + [1. 独立的列](#1-----)
    + [2. 前缀索引和索引的选择性](#2------------)
    + [3. 多列索引](#3-----)
    + [4. 选择合适的索引顺序](#4----------)
    + [5. 聚簇索引](#5-----)
    + [6. 覆盖索引](#6-----)



# MySQL-创建高性能索引

前面学习了 [MySQL-数据类型优化]((https://onlythepiano.github.io/mysql-数据类型优化/)) ，和 [MySQL-查询性能优化](https://onlythepiano.github.io/mysql-查询性能优化/) 。而这里学习索引。索引 (在MySQL中也叫做 `键(key)`) 是存储引擎用于快速找到记录的一种数据结构。

索引对于良好的性能非常关键。尤其是当表中的数据量越来越大时，索引对性能的影响愈发重要。在数据量较小且负载较低时，不恰当的索引对性能的影响可能还不明显，但当数据量逐渐增大时，性能则会急剧下降生。

## 一、索引基础

[索引](https://onlythepiano.github.io/索引/)

[B 树](https://onlythepiano.github.io/b树/)

**- 这里总结下来索引的三个优点：**

1. 索引大大减少了服务器需要扫描的数据量。
2. 索引可以帮助服务器避免排序和临时表。
3. 索引可以将随机I/O变为顺序I/O。



## 二、高性能的索引策略

**如何创建和使用索引以实现高性能查询呢？**



### 1. 独立的列

“独立的列” 指的是索引列不能是表达式的一部分，也不能是函数的参数。

~~~sql
SELECT actor_id FROM actor WHERE actor_id + 1 = 5;
~~~

这个是常见的错误，WHERE中的表达式其实等价于actor_id=4，但是MySQL无法自动解析。我们应该养成简化WHERE条件的习惯，始终将索引列单独放在比较符号的一侧。



### 2. 前缀索引和索引的选择性

当我们需要索引很长的字符列时，通常可以索引开始部分的字符，这样就可以大大节约索引空间。但是这样选择性低的索引使得 MySQL 在查找时过滤掉的行减少。也就是 `索引选择性` 降低了，但是一般情况下，某个列前缀的选择性也是足够高的，足以满足查询性能。


对于BLOB、TEXT 或者很长的 VARCHAR 类型的列，必须使用 `前缀索引` ，因为 MySQL 不允许索引这些列的完整长度。



**那么，如何决定前缀的合适长度呢 ？**

为了决定前缀的合适长度，需要找到最常见的值的列表，然后和最常见的前缀列表进行比较。

(1) 找到最常见的列表

![image-20200719102149794](https://github.com/OnlyThePiano/Notes/blob/master/images/image-20200719102149794.png)

(2) 查找最频繁出现的前缀列表，这里先从 3 个前缀开始

 ![image-20200719102356756](https://github.com/OnlyThePiano/Notes/blob/master/images/image-20200719102356756.png)

(3) 增加前缀长度，直到这个前缀的选择性接近完整列的选择性。。。。。一直加到了 7 ，发现才比较接近

![image-20200719102544623](https://github.com/OnlyThePiano/Notes/blob/master/images/image-20200719102544623.png)

(4) 完成类前缀长度的查找，接着完成 `前缀索引` 的创建

~~~sql
ALTER TABLE city.demo ADD KEY (city(7));
~~~



### 3. 多列索引

在 MySQL5.0 和更新的版本中，引入了 `索引合并（index merger）` 的策略；查询能够同时使用这两个单列索引进行扫描，并将结果进行合并。MySQL 会使用这类技术优化复杂查询。

这种算法有三个变种：OR条件的联合(union) ，AND条件的相交(intersection)，组合前两种情况的联合及相交。下面的查询就是使用了两个索引扫描的联合，通过 `EXPLAIN中的Extra列` 可以看到这点：

~~~sql
//film_actor actor_id 上各有一个单列索引
//
SELECT film_id, actor_id
FROM film_actor
WHERE actor_id = 1 OR film_id = 1;
~~~

![image-20200719124144356](https://github.com/OnlyThePiano/Notes/blob/master/images/image-20200719124144356.png)

**- 索引合并策略有时候是一种优化的结果，但实际上更多时候说明了表上的索引建得很糟糕：**

* 当出现服务器对多个索引做相交操作时 (通常有多个 AND 条件)：

  通常意味着需要一个包含所有相关列的多列索引，而不是多个独立的单列索引。

* 当服务器需要对多个索引做联合操作时 (通常有多个OR条件) ：

  通常需要耗费大量CPU和内存资源在算法的缓存、排序和合并操作上。特别是当其中有些索引的选择性不高，需要合并扫描返回的大量数据的时候。

* 更重要的是，`优化器` 不会把这些计算到 “查询成本” (cost) 中，`优化器` 只关心随机页面读取。这会使得查询的成本被 “低估”，导致该执行计划还不如直接走全表扫描。这样，还不如像在MySQL4.1或者更早的时代一样，将查询改写成UNION的方式往往更好。如下改写：

~~~sql
SELECT film_id, actor_id FROM film_actor WHERE actor_id = 1
UNION ALL
SELECT film_id, actor_id FROM film_actor WHERE film_id = 1 AND actor <> 1;
~~~



所以，如果在 `EXPLAIN` 中看到有索引合并，应该好好检查一下查询和表的结构，看是不是已经时最优的。



### 4. 选择合适的索引顺序

在一个多列 B-Tree 索引中，索引列的顺序意味着索引首先按照最左列进行排序，其次是第二列，等等。所以，索引可以按照升序或者降序进行扫描，以满足精确符合列顺序的 ORDER BY、GROUP BY 和 DISTINCT 等子句的查询需求。



**- 那么，如何选择索引的列顺序呢 ？**

对于如何选择索引的列顺序有一个经验法则：将选择性最高的列放到索引最前列。

因为，这时候索引的作用只是用于优化 `WHERE` 条件的查找。性能不只是依赖于所有索引列的选择性，也和值的分布有关。



**(1) 有具体数据查询时的方法 ：**

~~~sql
SELECT * FROM payment WHERE staff_id = 2 AND customer = 584;
~~~

是应该创建一个(staff_ id, customer_ _id) 索引还是应该颠倒一下顺序？我们需要可以跑一些查询来确定在这个表中值的分布情况，并确定哪个列的选择性更高。如下：

![image-20200719134342306](https://github.com/OnlyThePiano/Notes/blob/master/images/image-20200719134342306.png)

由上图结果显示，应该把 custmoer_id 放在前面，因为它的选择性更高。



**(2) 没有具体数据查询时的方法 ：**

![image-20200719134958656](https://github.com/OnlyThePiano/Notes/blob/master/images/image-20200719134958656.png)

由上图结果显示，应该把 custmoer_id 放在前面，因为它的选择性更高。



**注意：** 当查询预测时，某些条件值得基数比正常值高，就可能会导致服务器性能问题，如下：

![image-20200719135341174](https://github.com/OnlyThePiano/Notes/blob/master/images/image-20200719135341174.png)

![image-20200719135400026](https://github.com/OnlyThePiano/Notes/blob/master/images/image-20200719135400026.png)从上面的结果来看符合组 (groupId) 条件几乎满足表中的所有行，符合用户 (userId) 条件的有 130 万条记录；也就是说索引 (groupId, userId) 基本上没什么用。

因为这些数据是从其他应用中迁移过来的，迁移的时候把所有的消息都赋予了管理员组的用户。解决办法是修改应用程序代码，区分这类特殊用户和组，禁止针对这类用户和组执行这个查询。



### 5. 聚簇索引

[聚簇索引和覆盖索引](https://github.com/OnlyThePiano/Notes/blob/master/MySQL-聚簇索引和覆盖索引.md)

### 6. 覆盖索引

[聚簇索引和覆盖索引](https://github.com/OnlyThePiano/Notes/blob/master/MySQL-聚簇索引和覆盖索引.md)

