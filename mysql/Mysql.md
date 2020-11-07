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

## 通过索引优化

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

哈希索引适合等值查询，但是不无法进行范围查询 哈希索引没办法利用索引完成排序 哈希索引不支持多列联合索引的最左匹配规则，如果有大量重复键值的情况下哈希索引的效率会很低，因为存在哈希碰撞问题

### 回表

在拿到第一次查询结果之后还会回到某一个表继续查询

### 最左匹配

MYSQL查询会遵循最左前缀匹配原则，即最左优先在检索数据时从调教的最左边开始匹配,MYSQL中用到索引的条件是必须要遵守最左前缀原则，该原则如果遇到联合索引那么会将优先匹配联合索引的最小的索引即单个索引，联合索引的顺序是有意义的，前后不同的联合索引起到的作用完全不同，比如(key2,key1,key3)会先匹配(key2)然后匹配(key2,key1)最后匹配(key2,key1,key3)

### 索引覆盖

一个查询语句的执行只用从索引中就能取得，不必从数据中读取称之为索引覆盖

当一条查询语句符合覆盖索引条件时，MYSQL只需要通过索引就可以返回查询所需要的数据，这样避免了查到索引后再返回表操作，减少I/O提高效率。比如，表covering_index_sample中有一个普通索引 idx_key1_key2(key1,key2)。当我们通过SQL语句：select key2 from covering_index_sample where key1 = ‘keytest’;的时候，就可以通过覆盖索引查询，无需回表。

### 索引下推

### 联合索引

联合索引是指将多个字段作为一个索引称为联合索引，一般的情况下是按照最左前缀匹配，因为MYSQL所以查询会遵循最左前缀匹配的原则，即最左优先，在检索数据时从联合索引的最左边开始匹配，索引当我们创建一个联合索引的时候比如(key1,key2,key3)相当于创建了(key1),(key1,key2)和(key1,key2,key3)三个索引这就是最左匹配原则

### 索引优化

### 索引失效

### 聚簇索引和非聚簇索引

B+Tree的**叶子节点可能存储的是整行数据**这种称为**主键索引**也被称为**聚簇索引**，B+Tree的**叶子节点也可能存储了主键的值**这种称为**非主键索引**，也被称为**非聚簇索引**

聚簇索引比非聚簇索引查询要快，聚簇索引由于存储的是真行数据所以直接查询到该叶子节点就是我们所需要的数据了，但是非聚簇索引查到主键之后还需要进行回表再进行一次查询



### 索引匹配方式

### 索引具体的优化点

