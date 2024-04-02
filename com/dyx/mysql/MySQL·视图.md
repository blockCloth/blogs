## 介绍
视图（View）是一种虚拟表，它是基于一个或多个实际表的查询结果构建而成的。视图本身并不包含数据，而是通过执行定义该视图的查询来获取数据。在数据库中，视图提供了以下几个方面的优势和用途：

1. **简化复杂的查询**：视图可以将复杂的查询逻辑封装在一个命名的对象中。这样，当需要执行某个复杂查询时，只需引用该视图，而无需重新编写查询语句。
2. **安全性**：视图可以限制用户对数据的访问权限。通过在视图上设置适当的权限，可以隐藏敏感数据或限制用户只能访问他们需要的数据部分，从而增强数据的安全性。
3. **简化数据访问**：视图可以将多个表中的数据整合在一起，并按照用户的需求进行格式化。这样，用户可以通过简单的查询语句访问需要的数据，而不必了解底层表结构的复杂性。
4. **提高性能**：视图可以对频繁使用的查询进行优化，将这些查询的结果缓存起来，从而提高查询性能。此外，视图还可以减少对底层表的访问次数，降低数据库系统的负载。
5. **逻辑数据独立性**：视图将数据的物理存储与逻辑表示分离开来，使得应用程序可以更容易地适应数据结构的变化，而不必修改已有的查询逻辑。
## 语法

1. 创建
- **CREATE [OR REPLACE] VIEW**：这是创建视图的关键字。**CREATE VIEW**用于创建新的视图，而**CREATE OR REPLACE VIEW**用于创建或替换已存在的同名视图。
- **视图名称**：这是要创建的视图的名称，您可以根据需求自定义视图的名称。
- **(列名列表)**：这是可选的部分，用于指定视图所包含的列名列表。如果不指定列名列表，视图将包含 SELECT 语句中指定的所有列。
- **AS SELECT 语句**：这是定义视图的查询语句，它指定了视图的数据来源和数据筛选条件。在这里，您可以编写一个 SELECT 查询语句，用于从一个或多个表中检索数据，并根据需要应用筛选条件。
- **WITH [CASCADED | LOCAL] CHECK OPTION**：这是可选的部分，用于指定对视图的修改操作的检查选项。**WITH CASCADED CHECK OPTION**表示对视图的任何修改都必须满足视图定义的所有检查条件和约束条件，而**WITH LOCAL CHECK OPTION**表示对视图的修改必须满足视图自身的检查条件，但不必满足基表的检查条件。
```sql
create [or replace] view 视图名称[(列明列表)] as select 语句 [with [cascaded | local] check option];
```

2. 查询
```sql
查看创建视图语句：show create view 视图名称;
查看视图数据：select * from 视图名称 .....;
```

3. 修改
```sql
方式一：CREATE [OR REPLACE] VIEW 视图名称[(列名列表)] AS SELECT语句 [ WITH
[ CASCADED | LOCAL ] CHECK OPTION ]

方式二：ALTER VIEW 视图名称[(列名列表)] AS SELECT语句 [ WITH [ CASCADED |
LOCAL ] CHECK OPTION ]
```

4. 删除
```sql
drop view [if exists] 视图名称;
```
## 检查选项
当使用**WITH CHECK OPTION**子句创建视图时，**MySQL**会通过视图检查正在更改的每个行，例如 插 入，更新，删除，以使其符合视图的定义。 **MySQL**允许基于另一个视图创建视图，它还会检查依赖视 图中的规则以保持一致性。为了确定检查的范围，mysql提供了两个选项： **CASCADED** 和 **LOCAL** ，默认值为 **CASCADED** 。  
### CASCADED
**CASCADING CHECK OPTION（级联检查选项）**是用于视图创建语句的一部分。它的作用是确保在对视图进行更新操作时，更新操作不会违反视图定义的检查条件，同时还会检查视图所依赖的基表的检查条件。
具体来说，**CASCADING CHECK OPTION**用于保证以下两个方面的一致性：

1. **视图定义的一致性**：对于视图中的每一行，更新操作必须满足视图定义的所有检查条件和约束条件。如果更新操作导致违反视图定义的条件，那么更新操作将被拒绝。
2. **基表的一致性**：除了视图本身的检查条件外，还会检查视图所依赖的基表的检查条件。这可以确保更新操作不会违反基表的检查条件，从而保持视图和基表之间的一致性。

比如，v2视图是基于v1视图的，如果在v2视图创建的时候指定了检查选项为**cascaded**，但是v1视图 创建时未指定检查选项。则在执行检查时，不仅会检查v2，还会级联检查v2的关联视图v1。
![image.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1712017325370-8db063d4-371e-42fb-b679-9b619906f66d.png#averageHue=%23faf3ed&clientId=u36853eeb-58ff-4&from=paste&height=226&id=u2e56b4cd&originHeight=282&originWidth=1284&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=16133&status=done&style=none&taskId=uc0cbdc31-56e7-469e-82a8-273dedb79ee&title=&width=1027.2)
### LOCAL
**LOCAL CHECK OPTION**（本地检查选项）与 **CASCADING CHECK OPTION** 类似，但是它只检查视图本身定义的检查条件，而不会检查视图所依赖的基表的检查条件。换句话说，**LOCAL CHECK OPTION** 只会确保对视图的更新操作不会违反视图定义的检查条件和约束条件，而不会检查基表的检查条件。
比如，v2视图是基于v1视图的，如果在v2视图创建的时候指定了检查选项为local ，但是v1视图创 建时未指定检查选项。 则在执行检查时，知会检查v2，不会检查v2的关联视图v1。
![image.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1712017442137-68d1ad0e-39c3-418a-aafd-1ab6cd608aa5.png#averageHue=%23fbf4ef&clientId=u36853eeb-58ff-4&from=paste&height=219&id=u846e1724&originHeight=274&originWidth=1255&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=13815&status=done&style=none&taskId=ua543e83a-2890-4ff5-b38d-92e287dfc80&title=&width=1004)
## 视图的更新
 要使视图可更新，视图中的行与基础表中的行之间必须存在一对一的关系。如果视图包含以下任何一项，则该视图不可更新：

- 聚合函数或窗口函数（**SUM()、 MIN()、 MAX()、 COUNT() **等）
- **DISTINCT**
- **GROUP** **BY**
- **HAVING**
- **UNION** 或者 **UNION** **ALL ** 

