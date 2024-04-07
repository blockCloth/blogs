## 介绍
在MySQL中，触发器（Trigger）是一种特殊的存储过程，它与数据库表相关联，并在特定的数据操作（如插入、更新、删除）发生时自动触发执行。触发器可以在数据操作前（**BEFORE**）或者数据操作后（**AFTER**）执行，允许开发者在数据库中定义一些自动化的业务逻辑，以确保数据的完整性和一致性。使用别名OLD和NEW来引用触发器中发生变化的记录内容，这与其他的数据库是相似的。

| 触发器类型 | NEW 和 OLD |
| --- | --- |
| INSERT 型触发器 | NEW 表示将要或者已经新增的数据 |
| UPDATE 型触发器 | OLD 表示修改之前的数据，NEW 表示将要或已经修改后的数据 |
| DELETE 型触发器 | OLD 表示将要或者已经删除的数据 |

触发器通常用于执行以下操作：

1. **数据验证**：在插入、更新、删除操作之前对数据进行验证，以确保数据符合预期的条件。
2. **数据补充**：在数据插入或更新时，自动填充一些字段或计算一些衍生数据。
3. **数据同步**：在某个表发生变化时，同步更新其他相关的表。
4. **日志记录**：记录特定数据操作的日志，以便后续分析或审计。
## 语法

1. 创建
```sql
CREATE TRIGGER trigger_name
{BEFORE | AFTER} {INSERT | UPDATE | DELETE}
ON table_name
FOR EACH ROW
BEGIN
    -- 触发器执行的逻辑代码
END;
```

2. 查看
```sql
show triggers;
```

3. 删除
```sql
DROP TRIGGER  [schema_name.]trigger_name; -- 如果没有指定 schema_name，默认为当前数据库。
```
## 示例
通过触发器记录`**tb_user **`表的数据变更日志，将变更日志插入到日志表user_logs中, 包含增加, 修改, 删除；
```sql
# 创建操作日志表
create table user_logs
(
    id             int(11)     not null auto_increment,
    operation      varchar(20) not null comment '操作类型, insert/update/delete',
    operate_time   datetime    not null comment '操作时间',
    operate_id     int(11)     not null comment '操作的ID',
    operate_params varchar(500) comment '操作参数',
    primary key (`id`)
) engine = innodb
  default charset = utf8;

# 创建 insert 触发器
create trigger tb_user_insert_trigger
    before insert
    on tb_user
    for each row
begin
    insert into user_logs
    values (null, 'insert', now(), NEW.id,concat('插入后的数据：id=', NEW.id, ',name=', NEW.name, ',phone=', NEW.phone, ',email=,', NEW.email,',profession=', NEW.profession));
    
end;


insert into tb_user(id, name, phone, email, profession, age, gender, status, createtime)
VALUES (25, '二皇子', '18809091212', 'erhuangzi@163.com', '软件工程', 23, '1', '1', now());


# 创建update触发器
create trigger tb_user_update_trigger
    after update
    on tb_user
    for each row
begin
    insert into user_logs
    values (null, 'update', now(), NEW.id,
            concat('插入前的数据：id=', OLD.id, ',name=', OLD.name, ',phone=', OLD.phone, ',email=,', OLD.email,',profession=', OLD.profession, ' | 插入后的数据：id=', NEW.id, ',name=', NEW.name, ',phone=',NEW.phone, ',email=,', NEW.email,',profession=', NEW.profession));
            
end;

update tb_user
set profession = '会计'
where id = 23;

# 创建delete触发器
create trigger tb_user_delete_trigger
    after delete
    on tb_user
    for each row
begin
    insert into user_logs
    values (null, 'delete', now(), OLD.id,
            concat('删除前的数据：id=', OLD.id, ',name=', OLD.name, ',phone=', OLD.phone, ',email=,', OLD.email,',profession=', OLD.profession));
            
end;

delete from tb_user where id = 25;
```
![image.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1712449530983-45236a81-5422-44ee-88ad-d50dba8722c5.png#averageHue=%232f353e&clientId=uaed90544-01c3-4&from=paste&height=194&id=ud9c92b73&originHeight=243&originWidth=1316&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=57993&status=done&style=none&taskId=u6174e9aa-ce8c-4050-a332-a826818549c&title=&width=1052.8)
