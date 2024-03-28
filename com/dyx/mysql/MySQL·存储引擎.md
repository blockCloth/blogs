## 存储引擎介绍
存储引擎就是存储数据、建立索引、更新/查询数据等技术的实现方式。存储引擎是基于表的，而不是基于库的，所以存储引擎也可以称为表类型，我们可以在创建表的时候，来指定选择的存储引擎，如果没有指定则会自动选择默认的存储引擎（`INNODB`）
### 建表时如何指定存储引擎
```sql
CREATE TABLE `表名`  (
  `id` int NULL
) ENGINE = InnoDB;
```
### 查询当前数据库支持的存储引擎
```sql
show engines;

InnoDB	DEFAULT	Supports transactions, row-level locking, and foreign keys	YES	YES	YES
```
## 存储引擎的特点
主要介绍三种存储引擎 `InnoDB、MyISAM、Memory`的特点
### InnoDB
`InnoDB`是一种兼容高可用性和高性能的通用存储引擎，在`MySQL 5.5`之后，`InnoDB`是`MySQL`默认的存储引擎
#### 特点

- `DML`操作遵循`ACID`模型、支持事务；
- 行级锁、提高并发访问性能；
- 支持外键`FOREING KEY`约束，保证数据的完整性和正确性；
#### 文件
 `xxx.ibd：xxx`代表的是表名，`InnoDB`引擎的每张表都会对应这样一个表空间文件，存储该表的表结 构（frm-早期的 、sdi-新版的）、数据和索引。  
 **参数：innodb_file_per_table** 如果该参数开启，代表对于InnoDB引擎的表，每一张表都对应一个ibd文件。 
```sql
show variables like 'innodb_file_per_table';
```
![image.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1711635092818-edadd2ff-42b2-495f-8a3b-8fc63e5b9920.png#averageHue=%23333841&clientId=u002f842d-50e8-4&from=paste&height=62&id=u3b615b20&originHeight=78&originWidth=550&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=7621&status=done&style=none&taskId=u4c8cd6a0-7c12-485f-86d2-eb5f662a694&title=&width=440)
我们直接打开MySQL的 数据存放目录：` MySQL安装路径\Data` ， 可以看到里面有很多的ibd文件，每一个ibd文件就对应一张表
![image.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1711635202773-ef41935a-4bb3-4679-9148-804349c434e5.png#averageHue=%23fcfbfa&clientId=u002f842d-50e8-4&from=paste&height=168&id=u97a614fe&originHeight=210&originWidth=786&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=14487&status=done&style=none&taskId=u28d102a5-1e2c-466c-b7ff-9191efa02ac&title=&width=628.8)
比如：我们有一张表 `menu`，就 有这样的一个`menu.ibd`文件，而在这个`ibd`文件中不仅存放表结构、数据，还会存放该表对应的 索引信息。 而该文件是基于二进制存储的，不能直接基于记事本打开，我们可以使用``提供的一 个指令 `ibd2sdi` ，通过该指令就可以从ibd文件中提取sdi信息，而sdi数据字典信息中就包含该表的表结构。  
![image.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1711635332483-889d24bb-eb8d-4d13-8823-552face1b2a4.png#averageHue=%23111111&clientId=u002f842d-50e8-4&from=paste&height=560&id=u5f3a1bce&originHeight=700&originWidth=1268&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=70887&status=done&style=none&taskId=u6bfee9ec-1dac-4bb8-8323-97a9ce67ba6&title=&width=1014.4)
#### 逻辑存储结构
![image.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1711635419046-427cb040-d6ca-4fa0-9b26-194f98ad33d0.png#averageHue=%238fca51&clientId=u002f842d-50e8-4&from=paste&height=360&id=u18125434&originHeight=450&originWidth=1075&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=79262&status=done&style=none&taskId=u11ab0e7b-602b-43df-b4ec-bb5539c8c38&title=&width=860)

-  **表空间 **: `InnoDB`存储引擎逻辑结构的最高层，`ibd`文件其实就是表空间文件，在表空间中可以 包含多个`**Segment**`段。 
- **段** : 表空间是由各个段组成的， 常见的段有数据段、索引段、回滚段等。`InnoDB`中对于段的管 理，都是引擎自身完成，不需要人为对其控制，一个段中包含多个区。 
- **区** : 区是表空间的单元结构，每个区的大小为1M。 默认情况下， InnoDB存储引擎页大小为 16K， 即一个区中一共有64个连续的页。 
- **页** : 页是组成区的最小单元，页也是`InnoDB` 存储引擎磁盘管理的最小单元，每个页的大小默 认为 16KB。为了保证页的连续性，`InnoDB` 存储引擎每次从磁盘申请 4-5 个区。 
- **行** : `InnoDB` 存储引擎是面向行的，也就是说数据是按行进行存放的，在每一行中除了定义表时 所指定的字段以外，还包含两个隐藏字段(后面会详细介绍)。  
###  MyISAM  
 `MyISAM`是`MySQL`早期的默认存储引擎。 
#### 特点

- 不支持事务，不支持外键 
- 支持表锁，不支持行锁 
- 访问速度快 
#### 文件

-  xxx.sdi：存储表结构信息
-  xxx.MYD: 存储数据
-  xxx.MYI: 存储索引

![image.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1711635696136-7abe529c-b0f5-47b2-a044-6f115619af69.png#averageHue=%23d0ad7c&clientId=u002f842d-50e8-4&from=paste&height=102&id=ue30e99ff&originHeight=128&originWidth=769&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=10564&status=done&style=none&taskId=u3a9836df-c1fe-4c12-9c2a-ccb3f890ff1&title=&width=615.2)
###  Memory  
Memory引擎的表数据时存储在内存中的，由于受到硬件问题、或断电问题的影响，只能将这些表作为 临时表或缓存使用
#### 特点 

- 内存存放 hash索引（默认）
#### 文件 

- xxx.sdi：存储表结构信息  
### 区别及特点
| 特点 | InnoDB | MyISAM | Memory |
| --- | --- | --- | --- |
| 存储限制 | 64TB | 有 | 有 |
| 事务安全 | 支持 | - | - |
| 锁机制 | 行锁 | 表锁 | 表锁 |
| B+tree索引 | 支持 | 支持 | 支持 |
| Hash索引 | - | - | 支持 |
| 全文索引 | 支持（5.6之后） | 支持 | - |
| 空间使用 | 高 | 低 | N/A |
| 内存使用 | 高 | 低 | 中等 |
| 批量插入速度 | 低 | 高 | 高 |
| 支持外键 | 支持 | - | - |

## 存储引擎选择
 在选择存储引擎时，应该根据应用系统的特点选择合适的存储引擎。对于复杂的应用系统，还可以根据 实际情况选择多种存储引擎进行组合。 

- **InnoDB**: 是`Mysql`的默认存储引擎，支持事务、外键。如果应用对事务的完整性有比较高的要求，在并发条件下要求数据的一致性，数据操作除了插入和查询之外，还包含很多的更新、删除操作，那么`InnoDB`存储引擎是比较合适的选择。 
- **MyISAM** ： 如果应用是以读操作和插入操作为主，只有很少的更新和删除操作，并且对事务的完整性、并发性要求不是很高，那么选择这个存储引擎是非常合适的。 
- **MEMORY**：将所有数据保存在内存中，访问速度快，通常用于临时表及缓存。`MEMORY`的缺陷就是对表的大小有限制，太大的表无法缓存在内存中，而且无法保障数据的安全性。  
