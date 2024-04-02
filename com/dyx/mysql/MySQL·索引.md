## 索引概述
索引（index）是帮助MySQL**高效获取数据的数据结构（有序）**。在数据之外，数据库系统还维护这满足特点查找算法的数据结构，这些数据结构以某种方式引用（指向）数据，这样可以在这些数据结构上实现高级查找算法，这种数据机构就是索引。
### 演示
假如我们要执行的SQL语句为：` select * from user where age = 45;`
![image.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1711781569633-9a62bc8c-a9f4-4b83-8dea-87c1275625cf.png#averageHue=%23fbfaf9&clientId=u20993996-398f-4&from=paste&height=347&id=u46e78aee&originHeight=434&originWidth=369&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=33189&status=done&style=none&taskId=u31be82e8-5039-48fa-89c3-a11c4e9acfe&title=&width=295.2)
#### 无索引情况
无索引情况，就是从第一行扫描到最后一行，这种称之为全表扫描，效率很低
![image.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1711781646271-9e9a05a9-d852-474c-9fe0-97d09ae51444.png#averageHue=%23fceceb&clientId=u20993996-398f-4&from=paste&height=434&id=ua20a4dcb&originHeight=542&originWidth=560&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=58231&status=done&style=none&taskId=u189b2d79-d103-4a8d-a1d7-7a59f368d5b&title=&width=448)
#### 有索引情况
如果我们针对于这张表建立了索引，假设索引结构就是二叉树，那么也就意味着，会对age这个字段建 立一个二叉树的索引结构。此时我们在进行查询时，只需要扫描三次就可以找到数据了，极大的提高的查询的效率。
![image.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1711781848104-0bbf313d-f834-48a2-9af8-0ad64d6eca98.png#averageHue=%23faf9f8&clientId=u20993996-398f-4&from=paste&height=433&id=u10231c86&originHeight=541&originWidth=958&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=79204&status=done&style=none&taskId=u0e3d2ee6-1e06-48cf-adda-a4129e5a580&title=&width=766.4)
### 特点
| 优势 | 劣势 |
| --- | --- |
| 提高数据检索的效率，降低数据库的IO成本 | 索引列也是要占用空间的 |
| 通过索引列对数据进行排序，降低数据排序的成本，降低CPU的消耗 | 索引提高了查询效率，但是也降低了更新表的速度，如对表进行INSERT、UPDATE、DELETE时，效率降低 |

## 索引分类
### 索引分类
在MySQL中，索引具体可以分为以下几类：主键索引、唯一索引、常规索引、全文索引

| 分类 | 含义 | 特点 | 关键字 |
| --- | --- | --- | --- |
| 主键索引 | 针对于表中主键创建的索引 | 默认自动创建，只能有一个 | PRIMARY |
| 唯一索引 | 避免同一个表中某数据列中的值重复 | 可以有多个 | UNIQUE |
| 常规索引 | 快速定位特点数据 | 可以有多个 | 
 |
| 全文索引 | 查找文本中的关键字，而不是比较索引中的值 | 可以有多个 | FULLTEXT |

### 聚集索引&二级索引
在`InnoDB`存储引擎中，根据索引的存储形式，又可以划分为以下两种：

| 分类 | 含义 | 特点 |
| --- | --- | --- |
| 聚集索引（Clustered Index） | 将数据存储与索引放到了一块，索引结构的叶子节点保存了行数据 | 必须有，而且只有一个 |
| 二级索引（Secondary Index） | 将数据与索引分开存储，索引结构的叶子节点管理的是对应的主键 | 可以存在多个 |

#### 聚集索引选取规则

- 如果存在主键，主键索引就是聚集索引。
- 如果不存在主键，将使用第一个唯一（UNIQUE）索引作为聚集索引。
- 如果表没有主键，或没有合适的唯一索引，则InnoDB会自动生成一个`rowid`作为隐藏的聚集索引。
#### 具体结构
聚集索引和二级索引的具体结构如下：

- 聚集索引的叶子节点下挂的是这一行的数据。
- 二级索引的叶子节点下挂的是该字段值对应的主键值。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1711783530163-d86876be-893e-40f8-9fe0-dcb754e75114.png#averageHue=%23f9f6f4&clientId=u20993996-398f-4&from=paste&height=594&id=u83a578ca&originHeight=743&originWidth=1541&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=194471&status=done&style=none&taskId=u03d1bc6e-9993-48aa-9b12-6d6ce30ccc1&title=&width=1232.8)
#### 查找过程

1.  由于是根据name字段进行查询，所以先根据`name='Arm'`到name字段的二级索引中进行匹配查 找。但是在二级索引中只能查找到 `Arm `对应的主键值 10。
2. 由于查询返回的数据是*，所以此时，还需要根据主键值10，到聚集索引中查找10对应的记录，最 终找到10对应的行row。
3. 最终拿到这一行的数据，直接返回即可。

**回表查询：这种先到二级索引中查找数据，找到主键值，然后再到聚集索引中根据主键值，获取 数据的方式，就称之为回表查询。**
![image.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1711783594144-06347042-4318-479b-89f8-ab10edaafaae.png#averageHue=%23faf9f7&clientId=u20993996-398f-4&from=paste&height=652&id=ud4547604&originHeight=815&originWidth=1560&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=144827&status=done&style=none&taskId=uff3eeaa1-4abe-467f-b3fa-79ae795f4be&title=&width=1248)
## 索引语法

1. 创造索引
```sql
CREATE { UNIQUE | FULLTEXT } INDEX index_name ON table_name(index_col_name,...);	
```

2. 查看索引
```sql
SHOW INDEX FROM table_name;	
```

3. 删除索引
```sql
DROP INDEX index_name ON table_name;
```
## explain
`EXPLAIN`或者`DESC`命令获取`MySQL `如何执行`SELECT `语句的信息，包括在`SELECT `语句执行 过程中表如何连接和连接的顺序。
```sql
-- 直接在select语句之前加上关键字 explain / desc
explain / desc  select 字段列表 from table_name where 条件；
```
![image.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1711784055579-14bb194c-bc19-4a5d-8607-f80276be04c5.png#averageHue=%230a0908&clientId=u20993996-398f-4&from=paste&height=345&id=ub1a451ef&originHeight=431&originWidth=1417&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=29506&status=done&style=none&taskId=ubf3abd67-e3ab-4bd4-8ded-21db521b945&title=&width=1133.6)
### 字段含义
| 字段  |  含义 |
| --- | --- |
| id | select查询的序列号，表示查询中执行select子句或者是操作表的顺序 (id相同，执行顺序从上到下；id不同，值越大，越先执行)。 |
| select_type | 表示 SELECT 的类型，常见的取值有**SIMPLE**（简单表，即不使用表连接 或者子查询）、**PRIMARY**（主查询，即外层的查询）、
**UNION**（UNION 中的第二个或者后面的查询语句）、 **SUBQUERY**（SELECT/WHERE之后包含了子查询）等 |
| type | 表示连接类型，性能由好到差的连接类型为**NULL、system、const、 eq_ref、ref、range、 index、all **。 |
| possible_key | 显示可能应用在这张表上的索引，一个或多个。 |
| key | 实际使用的索引，如果为NULL，则没有使用索引。 |
| key_len | 表示索引中使用的字节数，该值为索引字段最大可能长度，并非实际使用长度，在不损失精确性的前提下，长度越短越好。 |
| rows | MySQL认为必须要执行查询的行数，在innodb引擎的表中，是一个估计值，可能并不总是准确的。 |
| filtered | 表示返回结果的行数占需读取行数的百分比， filtered 的值越大越好。 |

## 索引使用规则
### 最左前缀法则
如果索引了多列（联合索引），要遵守最左前缀法则。最左前缀法则指的是查询从索引的最左列开始， 并且不跳过索引中的列。如果跳跃某一列，索引将会部分失效(后面的字段索引失效)。
在`tb_user` 表中，有一个联合索引，这个联合索引涉及到三个字段，顺序分别为：profession， age，status。
![image.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1711785025331-e28c3983-7b67-4183-a5b9-8d8e8416d1cc.png#averageHue=%230a0807&clientId=u20993996-398f-4&from=paste&height=198&id=ZQjDt&originHeight=248&originWidth=1876&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=32651&status=done&style=none&taskId=u1a3b69c8-3399-473f-90f3-09793fb1fb4&title=&width=1500.8)
对于最左前缀法则指的是，查询时，最左变的列，也就是profession必须存在，否则索引全部失效。 而且中间不能跳过某一列，否则该列后面的字段索引将失效。 
```sql
explain select * from tb_user where profession = '' and age = 31 and status = '0';
```
![image.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1711785126776-2e476f32-ff3d-4f82-9a19-82bc4c1f6851.png#averageHue=%23070706&clientId=u20993996-398f-4&from=paste&height=227&id=u29703d63&originHeight=284&originWidth=1893&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=17774&status=done&style=none&taskId=u0382dfd0-d4e7-4e45-8b55-2628492eade&title=&width=1514.4)

```sql
explain select * from tb_user where profession = '' and age = 31;
```
![image.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1711785180569-340ade40-02d4-4f53-a76d-f99a27b07670.png#averageHue=%230a0908&clientId=u20993996-398f-4&from=paste&height=177&id=uee2f614a&originHeight=221&originWidth=1664&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=18086&status=done&style=none&taskId=u0e8005b0-2bf6-41c6-914f-1b80947b328&title=&width=1331.2)

```sql
explain select * from tb_user where profession = '';
```
![image.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1711785230589-c5387fd6-e4ae-417e-845b-26aa52532255.png#averageHue=%230a0908&clientId=u20993996-398f-4&from=paste&height=174&id=uf307b501&originHeight=217&originWidth=1580&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=17009&status=done&style=none&taskId=uf87c9db2-5100-44b8-8c7d-d4827dc7512&title=&width=1264)
我们发现只要联合索引最左边的字段 profession存在，索引就会生效，只不过索引的长度不同。如果不满足最左前缀法则，查询会使用索引吗？

很明显，通过下面两组测试，我们可以发现索引并未生效，因为不满足最左前缀法则，联合索引最左边的列`profession`不存在
```sql
explain select * from tb_user where age = 31 and status = '0';
explain select * from tb_user where status = '0';
```
![image.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1711785578591-962f7a32-5e25-40ad-b004-b2a99cb6fdc0.png#averageHue=%230a0908&clientId=u20993996-398f-4&from=paste&height=363&id=uc7cb5798&originHeight=454&originWidth=1492&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=33389&status=done&style=none&taskId=ufcf4f45e-3827-4c89-affc-1bf1e113a39&title=&width=1193.6)
### 范围查询
联合索引中，出现范围查询(`> , <`)，范围查询右侧的列索引失效。在业务允许的情况下，尽可能的使用类似于`>= `或     `<=` 这类的范围查询，而避免使用`> / <`
```sql
explain select * from tb_user where profession = '' and age > 30 and status = '0';
```
> 当范围查询使用> 或     < 时，走联合索引了，但是索引的长度为49，就说明范围查询右边的status字 段是没有走索引的。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1711785816352-088cf3ba-7ac8-40e7-b1e2-bea6e743ff13.png#averageHue=%230b0908&clientId=u20993996-398f-4&from=paste&height=174&id=u37590f4d&originHeight=218&originWidth=1769&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=18761&status=done&style=none&taskId=u3f1a6d9f-d8c9-4516-87d3-aee0ef800e9&title=&width=1415.2)

```sql
explain select * from tb_user where profession = '' and age >= 30 and status = '0';
```
> 当范围查询使用>= 或     <= 时，走联合索引了，但是索引的长度为54，就说明所有的字段都是走索引 的。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1711785898940-5186855b-2898-4272-a362-a275c1d72989.png#averageHue=%230a0908&clientId=u20993996-398f-4&from=paste&height=185&id=u91d9701c&originHeight=231&originWidth=1768&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=18881&status=done&style=none&taskId=u46815d1e-49cc-4187-a3c3-d0f020e7880&title=&width=1414.4)
### 索引失效情况

1. **索引列运算**
   1. 如果在索引列上进行运算操作，索引将失效。
2. **字符串不加引号**
   1. 在索引列上进行运算操作，索引将失效。
3. **模糊匹配**
   1. 如果仅仅是尾部模糊匹配，索引不会失效；如果是头部模糊匹配，索引就会失效
4. **or连接条件**
   1. 用or分割开的条件， 如果or前的条件中的列有索引，而后面的列中没有索引，那么涉及的索引都不会被用到。
5. **数据分布影响**
   1. 如果MySQL评估使用索引比全表更慢，则不使用索引
### SQL提示
我们可以在查询的时候，自己来指定使用哪个索引，此时就可以借助于 MySQL的SQL提示来完成。SQL提示，是优化数据库的一个重要手段，简单来说，就是在SQL语句中加入一些人为的提示来达到优 化操作的目的。

1. use index：建议MySQL使用哪一个索引完成此次查询（仅仅是建议，mysql内部还会再次进 行评估）。
```sql
explain select * from tb_user use index(idx_user_pro) where profession = '软件工 程';
```
![image.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1711787027307-91633df1-464f-48c1-b2ab-7b913d654903.png#averageHue=%23fbf9f8&clientId=u20993996-398f-4&from=paste&height=79&id=u8680170a&originHeight=99&originWidth=1306&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=6799&status=done&style=none&taskId=u014016fe-f02c-4193-b929-b9b36c8a2f7&title=&width=1044.8)

2. ignore index：忽略指定的索引。
```sql
explain select * from tb_user ignore index(idx_user_pro) where profession = '软件工程';
```
![image.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1711787096899-e19034dd-84cc-4b2b-a5c0-9837d64d9a79.png#averageHue=%23fbfaf9&clientId=u20993996-398f-4&from=paste&height=83&id=u1370476a&originHeight=104&originWidth=1280&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=6816&status=done&style=none&taskId=u4123481d-58be-4c3c-9809-67e86f781a7&title=&width=1024)

3. force index：强制使用索引。
```sql
explain select * from tb_user force index(idx_user_pro) where profession = '软件工程';
```
![image.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1711787179272-f4cc31b4-584e-4f24-a905-843c0c58cd46.png#averageHue=%23fcfcfb&clientId=u20993996-398f-4&from=paste&height=128&id=u7a744411&originHeight=160&originWidth=1252&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=6989&status=done&style=none&taskId=u4ce37bf8-b67d-48d6-b42d-50d67262d53&title=&width=1001.6)
### 前缀索引
 当字段类型为字符串（varchar，text，longtext等）时，有时候需要索引很长的字符串，这会让索引变得很大，查询时，浪费大量的磁盘IO， 影响查询效率。此时可以只将字符串的一部分前缀，建立索引，这样可以大大节约索引空间，从而提高索引效率。  
#### 语法
```sql
create index idx_xxx on table_name(column(n));
```
#### 前缀长度
 可以根据索引的选择性来决定，而选择性是指不重复的索引值（基数）和数据表的记录总数的比值， 索引选择性越高则查询效率越高， 唯一索引的选择性是1，这是最好的索引选择性，性能也是最好的。  
```sql
select count(distinct email) / count(*) from tb_user ;
select count(distinct substring(email,1,5)) / count(*) from tb_user ;
```
## 索引设计原则

1.  针对于数据量较大，且查询比较频繁的表建立索引。 
2. 针对于常作为查询条件（where）、排序（order by）、分组（group by）操作的字段建立索 引。
3. 尽量选择区分度高的列作为索引，尽量建立唯一索引，区分度越高，使用索引的效率越高。
4.  如果是字符串类型的字段，字段的长度较长，可以针对于字段的特点，建立前缀索引。 
5. 尽量使用联合索引，减少单列索引，查询时，联合索引很多时候可以覆盖索引，节省存储空间， 避免回表，提高查询效率。
6. 要控制索引的数量，索引并不是多多益善，索引越多，维护索引结构的代价也就越大，会影响增删改的效率。
7. 如果索引列不能存储NULL值，请在创建表时使用NOT NULL约束它。当优化器知道每列是否包含 NULL值时，它可以更好地确定哪个索引最有效地用于查询。  
