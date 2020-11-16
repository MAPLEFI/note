# Mysql

## 执行计划(暂时只了解，不做重点深入)

| id            | The Select indentifller                        |
| ------------- | ---------------------------------------------- |
| select_type   | The Select Type                                |
| table         | The table for the output row                   |
| partitions    | The matching partitons                         |
| type          | The join type                                  |
| possible_keys | The possible indexes to choose                 |
| key           | The index actually chosen                      |
| key_len       | The length of the chosen key                   |
| ref           | The columns compared to the index              |
| rows          | Estimate of rows to be examined                |
| filtered      | Percentage of rows filtered by table condition |
| extra         | Additional information                         |

### id

select查询的序列号，包含一组数字，表示查询中执行slect自居或者操作表的顺序

id号分为三种情况:

   1.如果id相同，那么执行顺序从上到下

```
explain select * from emp e join  dept d on e.deptno - d.deptno join salgrade sg on e.sa between sg.losal and sg.hisal;
```

 2.如果id不同，如果是子查询id的学号会递增id值越大优先级越高，越先被执行

```
explain select * from emp e where e.deptno in (select d.deptno from dept d where d.dname - 'SALES');
```

**查询优化器:一条SQL语句的查询可以有不同的执行方案，至于最终选择哪种方案，需要通过优化器进行选择，选择执行成本最低的方案。在一条单表查询的语句真正执行前，MYSQL的查询优化器会找出执行该语句所有可能使用的方案，对比之后找出成本最低的方案，这个成本最低的方案就是所谓的执行计划。优化过程大致如下：1.根据搜索条件找出所有可能使用的索引2.计算全表扫描的代价3.计算使用不同索引执行查询的代价4.对比各种执行方案的代价，找出成本最低的那一个**

## 通过索引优化

### 什么是索引？

索引是一种数据结构对数据库表种得一列或多列值进行排序的一种结构，使用索引可以快速访问数据库表中的特定信息

### 索引分类

主键索引

唯一索引

普通索引

全文索引(比如在一个文章中搜索JAVA在这个单词)

组合索引(上面针对单列，组合索引针对多列)

Mysql再创建的时候会为主键和唯一键创建索引

### 索引数据结构

B+树 InnoDB引擎默认使用的是B+树

哈希索引 底层为哈希表是一种以KV来存储数据，如果单个查询效率还可但是如果遇到区间查询那么无法通过索引进行查询，所以哈希索引只适用于等值查询的场景，二B+树是一种多路平衡查询树，所以他的结点是天然有序的(左子节点小于父节点，右子节点大于父节点)，所以对于范围查询的时候不需要做全表扫描

### 索引数据结构的选择

#### 哈希索引和B+Tree之间的区别

哈希索引适合等值查询，但是不无法进行范围查询 哈希索引没办法利用索引完成排序 哈希索引不支持多列联合索引的最左匹配规则，如果有大量重复键值的情况下哈希索引的效率会很低，因为存在哈希碰撞问题，而B+树不会出现碰撞问题并且支持范围查询

### 回表

在拿到第一次查询结果之后还会回到某一个表继续查询

### 索引最左匹配原则

MYSQL查询会遵循最左前缀匹配原则，即最左优先在检索数据时从调教的最左边开始匹配,MYSQL中用到索引的条件是必须要遵守最左前缀原则，该原则如果遇到联合索引那么会将优先匹配联合索引的最小的索引即单个索引，联合索引的顺序是有意义的，前后不同的联合索引起到的作用完全不同，比如(key2,key1,key3)会先匹配(key2)然后匹配(key2,key1)最后匹配(key2,key1,key3)如果在第二次匹配没有匹配上那么不会进行第三次匹配

### 索引覆盖

一个查询语句的执行只用从索引中就能取得，不必从数据中读取称之为索引覆盖

当一条查询语句符合覆盖索引条件时，MYSQL只需要通过索引就可以返回查询所需要的数据，这样避免了查到索引后再返回表操作，减少I/O提高效率。比如，表covering_index_sample中有一个普通索引 idx_key1_key2(key1,key2)。当我们通过SQL语句：select key2 from covering_index_sample where key1 = ‘keytest’;的时候，就可以通过覆盖索引查询，无需回表。

### 索引下推

MYSQL5.6引入索引下推优化，默认开启 使用SET optimizer_switch='index_condition_pushdown=off'可以将其关闭。

SELECT * FROM people WHERE zipcode=‘95054’ AND lastname LIKE ‘%etrunia%’ AND address LIKE ‘%Main Street%’;

如果没有使用索引下推技术，则MySQL会通过zipcode='95054’从存储引擎中查询对应的数据，返回到MySQL服务端，然后MySQL服务端基于lastname LIKE '%etrunia%'和address LIKE '%Main Street%'来判断数据是否符合条件。 

如果使用了索引下推技术则MYSQL**首先会返回符合zipcode='95054'的索引****然后根据lastname LIKE '%etrunia%'和address LIKE '%Main Street%'来判断索引是否符合条件**，如果符合条件，则根据索引来定位对应的数据，如果不符合直接reject掉。有了索引下推优化，可以在有like条件查询的情况下减少回表次数

### 联合索引

联合索引是指将多个字段作为一个索引称为联合索引，一般的情况下是按照最左前缀匹配，因为MYSQL所以查询会遵循最左前缀匹配的原则，即最左优先，在检索数据时从联合索引的最左边开始匹配，索引当我们创建一个联合索引的时候比如(key1,key2,key3)相当于创建了(key1),(key1,key2)和(key1,key2,key3)三个索引这就是最左匹配原则

### 索引优化

可以通过explain查看sql语句的执行计划，通过执行计划来分析索引使用情况

### 索引失效

某些情况下创建了索引但是执行的时候没用通过索引，即根据执行计划计算选择出的最佳方案没有使用该索引，或者是使用了like的情况如果%在左边那么会使用索引，如果%只在右边出现那么不会使用索引,也有可能发生了隐式转换导致索引失效(比如name是varchar类型的确进行int类型匹配比如name>123就会发生索引失效)

### 聚簇索引和非聚簇索引

B+Tree的**叶子节点可能存储的是整行数据**这种称为**主键索引**也被称为**聚簇索引**，B+Tree的**叶子节点也可能存储了主键的值**这种称为**非主键索引**，也被称为**非聚簇索引**

聚簇索引比非聚簇索引查询要快，聚簇索引由于存储的是真行数据所以直接查询到该叶子节点就是我们所需要的数据了，但是非聚簇索引查到主键之后还需要进行回表再进行一次查询

### 索引具体的优点

优化了查询的效率

通过创建唯一性索引可以保证数据库表中的每一行数据的唯一性

可以加速表与表之间的连接

在使用分组和排序进行检索时，可以减少查询中分组和排序得时间

主键自动建立唯一索引

### 索引的缺点

创建索引和维护索引要耗费时间，这种时间随着数据量的增加而增加

索引需要占用物理空间，数据量越大占用空间越大

会降低表的增删改的效率，因为每次增删改索引都需要进行动态维护

频繁更新的字段不适合创建索引，因为每次更新不单单是更新记录，还会更新索引保存索引文件

where条件里用不到的字段不创建索引

表记录太少不需要创建索引

经常增删改的表不需要创建索引

## 通过SQL语句进行优化

对查询进行优化，应该尽量避免全表扫描首先应该考虑在where及order by涉及的列上建立索引



尽量避免where子句中使用!=或< >操作符,否则引擎将放弃使用索引而进行全表扫描



尽量避免在where子句中对字段进行Null值判断否则将进行全表扫描不会走索引



尽量避免在where子句中使用or来连接条件否则将导致引擎放弃使用索引 而进行全表扫描



尽量避免在模糊查询like后使用前% 比如 like '%abd'此时将不会使用索引而是进行全表扫描

in和not in也要慎用否则会导致全表扫描，比如select id from t where num in (1,2,3)对于连续的数值能用between就不用in,select id from t where num between 1 and 3

尽量避免where子句对字段进行表达式操作会导致放弃索引进行全表扫描,比如select id from t where num/2=100,应该改为select id from t where num=100*2，尽量不要在where 条件的=左边进行运算会导致全表扫描



尽量避免在where 子句中对字段进行函数操作，浙江导致引擎放弃使用索引而进行全表扫描，比如 select id from t where substring(name,1,3)='abc' 应该改为select id from t where name like 'abc%'

任何地方都不要使用select * from t用具体的字段列表代替*不要返回用不到的任何字段
