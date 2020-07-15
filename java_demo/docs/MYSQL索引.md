

1.说说什么是索引？

2.索引具体采用的哪些数据结构？

3.InnoDB为什么采用B+树？

4.B+ Tree的叶子节点都可以存哪些东西？

5.在创建联合索引的时候，多个字段之间顺序是如何选择的呢？

6.什么情况下设置了索引但无法使用？

7.索引对数据库系统的负面影响是什么？

### 一、简介

#### 1.为何要有

​	一般的应用系统，插入和一般的更新操作很少出现性能问题。在生产环境中，往往会因为数据量或复杂的查询而导致查询效率低下。为了提高查询效率，需要进行优化，而索引则是首先会考虑的点。

#### 2.是什么

​	在MySQL中，索引（index）也叫做“键（key）”，它是存储引擎用于快速找到记录的一种数据结构。

​	本质：数据结构。

#### 3.工作原理

​	我们知道，在看一本书某章的时候，首先我们会查找目录索引，找到对应的页码然后快速找到相应的内容。mysql索引也一样，存储引擎利用类似的方法使用索引，先在索引中找到对应的值，然后根据匹配的索引记录找到对应的数据行，然后返回结果。

​	例如，我们想在一个10W条记录表 table 中查询name等于“张三”的数据行，select * from table where name ='张三'。那么在没有对name字段建立索引的情况下，我们需要扫描全表也就是扫描10W条数据来找到这条数据；如果我们为name字段建立索引，我们只需要查找索引，然后根据索引找到对应的数据行，只需要查找一条记录，性能会得到很大的提高。

#### 4.优缺点

​	优点：提高查询效率

​	缺点：创建索引和维护索引需要耗费时间，这个时间随着数据量的增加而增加；索引需要占用物理空间，不光是表需要占用数据空间，每个索引也需要占用物理空间；当对表进行增、删、改、的时候索引也要动态维护，这样就降低了数据的维护速度。

### 二、Mysql索引的实现

#### 1.MyISAM

​	MyISAM引擎：frm格式文件存储表结构，MYI格式文件存储索引，MYD格式文件存储数据。

​	MyISAM引擎使用B+Tree作为索引结构，叶节点data域存放数据记录的地址（索引文件和数据文件是分离的）

​	![MyISAM](.\png\MyISAM.png)

​	MyISAM中索引检索的算法为首先按照B+Tree搜索算法搜索索引，如果指定的Key存在，则取出其data域的值，然后以data域的值为地址，读取相应数据记录。 

​	在MyISAM中，主索引和辅索引（Secondary key）在结构上没有任何区别，只是主索引要求key是唯一的，而辅索引的key可以重复。

#### 2.InnoDB

​	InnoDB引擎：frm格式文件存储表结构，ibd格式文件存储索引和数据。

​	InnoDB也使用B+Tree作为索引结构，不同的是，叶节点data域保存了完整的数据记录。

​	![innodb](.\png\innodb.png)

​	InnoDB中主键的搜索十分高效，但是辅助索引搜索需要检索两遍索引：首先检索辅助索引获得主键，然后用主键到主索引中检索获得记录。

​	![innodb2](.\png\innodb2.png)

#### 3.区别

- MyISAM索引文件和数据文件是分离的，而InnoDB表数据文件本身就是主索引
- InnoDB的辅索引data域存储相应记录主键的值，而MyISAM的辅索引存储的是地址

### 三、为何B+树适合作为索引的结构

**如果数据结构是树的话，查找速度与树的高度有关。**

使用反推的方式来看：

​	1.没有索引：全部遍历（数据量小，时间复杂性O(n)）；

​	2.Hash：时间复杂度是O(1)，遍历只进行一次（哈希对于增删改查的平均时间复杂度都是O(1)），但是进行范围查找和排序查找时，时间复杂度就变为O(n)；

​	3.二叉树（一个节点只能有两个子节点，也就是一个节点度不能查过2；左子节点小于本节点，右子节点大于本节点）：查找次数与深度有关，深度为1，查找一次，深度为N，查找为N次，尤其对于id递增来说，树就退化成链表，查找复杂度就变为O(n);

​	4.平衡二叉树（二叉树的升级版，插入或删除节点后，会导致失衡，然后通过旋转，进一步保持平衡）：树的高度与数据库数量有关，数据量越大，树高也就会越高，那么查找次数就会变多。

​	5.B树（平衡多路查找树）：B树相对于平衡二叉树的不同是，每个节点包含的关键字增多了。树的节点关键字增多后树的层级比原来的二叉树少了，减少数据查找的次数和复杂度。

​	6.B+树（平衡多路查找树升级版）：B+树相对于B树来说，中间节点没有多余数据，只有索引，意味着同样的数据量下，B+树更加矮壮，同样大小的磁盘页能容纳更多的数据，IO操作较少；在范围查找方面，B树需要不断遍历，而B+树通过二分查找，找到范围下限，然后通过叶子节点的链表顺序遍历，直到找到上限，效率比较高。

### 四、分类

#### 1.分类

##### 1.主键索引（PRIMARY KEY）

创建表的时候设置的主键，或者创建表后添加的主键。

```sql
#随表一起建索引：
CREATE TABLE customer (id INT(10) UNSIGNED  AUTO_INCREMENT ,customer_no VARCHAR(200),customer_name VARCHAR(200),
  PRIMARY KEY(id) 
);
#使用AUTO_INCREMENT关键字的列必须有索引(只要有索引就行)。
CREATE TABLE customer2 (id INT(10) UNSIGNED,customer_no VARCHAR(200),customer_name VARCHAR(200),
  PRIMARY KEY(id) 
);
#单独建主键索引：
ALTER TABLE customer add PRIMARY KEY customer(customer_no);  
#删除建主键索引：
ALTER TABLE customer drop PRIMARY KEY ;  
#修改建主键索引：
#必须先删除掉(drop)原索引，再新建(add)索引
```

##### 2.单值索引（KEY或INDEX）

由关键字KEY或INDEX定义的索引，主要为经常用来排序或查询条件的数据列服务。

```sql
#随表一起建索引：
CREATE TABLE customer (id INT(10) UNSIGNED  AUTO_INCREMENT ,customer_no VARCHAR(200),customer_name VARCHAR(200),
  PRIMARY KEY(id),
  KEY (customer_name)  
);
#随表一起建立的索引 索引名同 列名(customer_name)
#单独建单值索引：
CREATE INDEX idx_customer_name ON customer(customer_name); 
#删除索引：
DROP INDEX idx_customer_name ;
```

##### 3.唯一索引（UNIQUE INDEX）

唯一索引是在表上一个或者多个字段组合建立的索引。

类似普通索引，但值不能重复（可以为空）。

```sql
#随表一起建索引：
CREATE TABLE customer (id INT(10) UNSIGNED  AUTO_INCREMENT ,customer_no VARCHAR(200),customer_name VARCHAR(200),
  PRIMARY KEY(id),
  KEY (customer_name),
  UNIQUE (customer_no)
);
#建立 唯一索引时必须保证所有的值是唯一的（除了null），若有重复数据，会报错。   
#单独建唯一索引：
CREATE UNIQUE INDEX idx_customer_no ON customer(customer_no); 
#删除索引：
DROP INDEX idx_customer_no on customer ;
```

##### 4.复合索引

两个或以上字段的索引叫做复合索引。

**最左前缀匹配**：根据业务需求，一般where子句中使用最频繁的一列放在最左边。例如索引是key index （***a,b,c***）。 可以支持***a | a,b| a,b,c*** 3种组合进行查找，但不支持 b,c进行查找 ，这些查询与where的顺序无关，mysql会自动优化。

```sql
#随表一起建索引：
CREATE TABLE customer (id INT(10) UNSIGNED  AUTO_INCREMENT ,customer_no VARCHAR(200),customer_name VARCHAR(200),
  PRIMARY KEY(id),
  KEY (customer_name),
  UNIQUE (customer_name),
  KEY (customer_no,customer_name)
);
#单独建索引：
CREATE INDEX idx_no_name ON customer(customer_no,customer_name); 
#删除索引：
DROP INDEX idx_no_name  on customer ;
```

##### 5.全文索引

目前搜索引擎使用的一种关键技术。

```sql
ALTER TABLE table_name ADD FULLTEXT (column);
```

##### 6.前缀索引

有时候需要索引很长的字符列，这会让索引变得大且慢。索引开始的部分字符，大大节约索引空间，提高索引效率。

```sql
ALTER TABLE table_name ADD INDEX(column(prefix_length));
```

#### 2.常用索引

```
唯一索引：
    -主键索引PRIMARY KEY：加速查找+约束（不为空、不能重复）
    -唯一索引UNIQUE:加速查找+约束（不能重复）
联合索引：
    -PRIMARY KEY(id,name):联合主键索引
    -UNIQUE(id,name):联合唯一索引
    -INDEX(id,name):联合普通索引
```

### 五、区别与联系

#### 1.聚簇索引和非聚簇索引（辅助索引）

​	按数据存储方式来分类，索引可以分为**聚簇索引**和**非聚簇索引**，聚簇索引的特点是叶子节点包含了完整的记录行，而非聚簇索引的叶子节点只有所以字段和主键ID。

​	InnoDB中，每个表必须有一个聚簇索引，默认是根据**主键**建立的。如果表中没有主键，InnoDB会选择一个合适的列作为聚簇索引，如果找不到合适的列，会使用一列隐藏的列`DB_ROW_ID`作为聚簇索引。

#### 2.单列索引和复合索引

​	只包含一个字段的索引叫做**单列索引**，包含两个或以上字段的索引叫做**复合索引**（或组合索引）。建立复合索引时，字段的顺序极其重要。

#### 3.唯一索引与主键

​	唯一索引是在表上一个或者多个字段组合建立的索引，这个（或这几个）字段的值组合起来在表中不可以重复。一张表可以建立任意多个唯一索引，但一般只建立一个。

​	主键是一种特殊的唯一索引，区别在于，唯一索引可以允许null值，而主键列不允许为null值。一张表最多建立一个主键，也可以不建立主键。

#### 4.聚簇索引与唯一索引

​	严格来说，**聚簇索引不一定是唯一索引**，聚簇索引的索引值并不要求是唯一的，**唯一聚簇索引**才是！在一个有聚簇索引的列上是可以插入两个或多个相同值的，这些相同值在硬盘上的物理排序与聚簇索引的排序相同，仅此而已。

### 六、使用

#### 1.如何选择

- 经常在where 从句，group by 从句，order by 从句，on 从句中出现的列
- 索引字段越小越好（索引长度影响索引文件的大小，影响增删的速度，并间接影响查询速度）
- 离散度大的列放到联合索引的前面，例如：select count(distinct x_id), count(distinct y_id) from 表名，谁的值大，说明谁的离散度更高
- 查询中很少使用的列不应该创建索引，如果创建会增加空间需求和降低mysql的性能
- 很少数据的列不应该创建索引，如果选择性超过 20% 那么全表扫描比使用索引性能更优。（例如：性别、类型等字段）
- 定义为text和image和bit数据类型的列不应该增加索引

#### 2.索引没起作用

- 复合索引不符合最左前缀原则
- 对索引字段做了运算 ：substr(name,-2)='xxx'
- 模糊查询使用了前模糊：like '%xxx'
- not in或者 != 
- 连续between
- 有or
- 如果列类型是字符串，需要使用引号引用起来 

#### 3.分析索引有效性-Explain

explain 这个命令用来查看一个SQL语句的执行计划，分析sql语句有没有用上索引，有没有全表扫描

explain select * from user

![explain1](png/explain1.png)

- type：显示了连接使用了哪种类别,有无使用索引；system>const>eq_ref>ref>range>index>all

  system：系统表，少量数据，往往不需要进行磁盘IO

  const：根据主键或唯一索引只取出确定的一行数据。是最快的一种

  eq_ref：查找唯一性索引，返回的数据至多一条。属于精确查找

  ref：查找非唯一性索引，返回匹配某一条件的多条数据。属于精确查找、数据返回可能是多条

  range：索引或主键，在某个范围内使用

  index：仅仅只有索引被扫描

  all：全表扫描，性能最差

- key：显示MySQL实际决定使用的键（索引）。如果没有选择索引，键是NULL

- rows：查找数据所需读取的行数

