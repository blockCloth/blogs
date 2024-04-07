## 介绍
**存储过程**是事先经过编译并存储在数据库中的一段 SQL 语句的集合，调用存储过程可以简化应用开发人员的很多工作，减少数据在数据库和应用服务器之间的传输，对于提高数据处理的效率是有好处的。存储过程思想上很简单，就是数据库 SQL 语言层面的代码封装与重用。  
![image.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1712018071397-fdd4ba91-8990-4f7b-84bf-20f57b57bfcf.png#averageHue=%23fdfdfd&clientId=u38a882d5-e74f-4&from=paste&height=251&id=ue8f3d8f8&originHeight=314&originWidth=1491&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=36329&status=done&style=none&taskId=u40404899-bb5b-4349-aecf-22436314464&title=&width=1192.8)
### 特点

- **封装，复用**：可以把某一业务SQL封装在存储过程中，需要用到 的时候直接调用即可。
- **可以接收参数，也可以返回数据**：再存储过程中，可以传递参数，也可以接收返回值。
- **减少网络交互，效率提升**：如果涉及到多条SQL，每执行一次都是一次网络传输。而如果封装在存储过程中，我们只需要网络交互一次可能就可以了。
## 语法

1. 创建
```sql
create procedure 存储过程名称(参数列表)
begin
  #sql 逻辑
end;
```

2. 调用
```sql
call 名称(参数);
```

3. 查看
```sql
SELECT * FROM INFORMATION_SCHEMA.ROUTINES WHERE ROUTINE_SCHEMA = 'xxx'; -- 查询指定数据库的存储过程及状态信息

SHOW CREATE PROCEDURE 存储过程名称 ; -- 查询某个存储过程的定义
```

4. 删除
```sql
DROP PROCEDURE [ IF EXISTS ] 存储过程名称；
```
## 变量
在MySQL中包含三种变量：**系统变量、用户变量、局部变量**
### 系统变量（System Variables）
**系统变量**是由 MySQL **自身定义和管理的变量**，用于控制 MySQL 的行为和配置。系统变量以 `**@@**` 符号开头，后面跟着变量名，如 `@@max_connections`。你可以使用 SET 语句来修改系统变量的值，但修改只会在当前会话中生效，不会影响全局的设置。要永久更改系统变量的值，需要修改 MySQL 的配置文件。总共分为**全局变量（GLOBAL）和会话变量（SESSION）**
> 注：如果没有指定 GLOBAL | SESSION 时，默认是session会话
> 1. 在MySQL重启之后，设置的全局变量都会失效，如果想不失效，需要在`my.cnf`文件中设置
> 2. 全局变量（GLOBAL）：针对所有会话都有效
> 3. 会话变量（SESSION）：只对当前会话生效，换一个会话就不生效了

```sql
-- 查看系统变量
SHOW [ SESSION | GLOBAL ] VARIABLES ; -- 查看所有系统变量
SHOW [ SESSION | GLOBAL ] VARIABLES LIKE '......'; -- 可以通过LIKE模糊匹配方式查找变量

SELECT @@[SESSION | GLOBAL] 系统变量名; -- 查看指定变量的值

-- 设置系统变量
SET [ SESSION | GLOBAL ] 系统变量名 = 值 ;
SET @@[SESSION | GLOBAL]系统变量名 = 值 ;

```
### 用户变量（User-defined Variables）
**用户变量**是由用户在会话中自定义的变量。它们以 `@` 符号开头，后面跟着变量名，如 `**@myVariable**`。用户变量可以在会话中使用，并且可以在 SELECT 查询、存储过程、触发器和用户定义的函数中进行操作。它们的值可以是任意的 MySQL 数据类型。
```sql
-- 方式一：赋值可以使用 = ，也可以使用 := 
SET   @var_name = expr  [, @var_name = expr] ... ;
SET   @var_name := expr  [, @var_name := expr] ... ;

-- 方式二
SELECT  @var_name := expr  [, @var_name := expr] ... ;
SELECT 字段名 INTO @var_name FROM 表名;

-- 使用,没有设置值时，默认为null
select  @var_name;
```
### 局部变量（Local Variables）
**局部变量**是在存储过程、函数或触发器中声明和使用的变量。它们用于在这些程序中存储临时数据。局部变量可以有不同的作用域和生命周期，只在声明的程序块中有效。
```sql
-- 声明，变量类型就是数据库字段类型：INT、BIGINT、CHAR、VARCHAR、DATE、TIME等
DECLARE 变量名    变量类型    [DEFAULT ... ] ;

-- 赋值
SET  变量名    = 值;
SET  变量名    := 值;
SELECT 字段名 INTO 变量名 FROM 表名 ... ;
```
## IF
**IF**语句用于在存储过程中执行条件判断和流程控制。它的语法结构与其他编程语言中的条件语句类似，允许根据条件执行不同的逻辑。

- **condition** 是一个条件表达式，可以是任何返回布尔值（TRUE或FALSE）的表达式。
- 如果**condition**为TRUE，则执行**IF**块中的代码，否则执行**ELSE**块中的代码。
- **ELSE**块是可选的，可以省略。
```sql
IF  condition  THEN
.....
ELSEIF condition THEN -- 可选
..... 
ELSE -- 可选
.....
END  IF;
```
### 示例
```sql
# if判断
# score >= 85分，等级为优秀。
# score >= 60分 且 score < 85分，等级为及格。
# score < 60分，等级为不及格。
create procedure p2(in source int)
begin
    declare result varchar(10) default '';

    if source >= 85 then
        set result := '优秀';
    elseif source >= 60 then
        set result := '及格';
    else
        set result := '不及格';
    end if;

    select result;
end;

call p2(85);
```
## 参数
参数的类型，主要分为三种：**IN、OUT、INOUT**

- **IN**：该类参数作为输入参数，也就是调用时需要传入的值，也是默认的参数类型
- **OUT**：该参数类型为输出参数，该参数可以作为返回值
- **INOUT**：既可以作为输入参数，也可以作为输出参数
```sql
CREATE PROCEDURE 存储过程名称 ([ IN/OUT/INOUT  参数名  参数类型 ]) 
BEGIN
-- SQL语句
END ;
```
### 示例
```sql
# 将传入的200分制的分数，进行换算，换算成百分制，然后返回。
create procedure p3(inout source double)
begin
    set source := source * 0.5;
end;

set @source = 142;
call p3(@source);
select @source;
```
## CASE
**CASE**语句是用于在查询中进行条件判断和返回不同结果的一种方式。**CASE**语句可以替代多个**IF-THEN-ELSE**语句，使得查询语句更加简洁和可读。它的基本语法结构如下：

- **condition1**, **condition2**, ... 是条件表达式，可以是任何返回布尔值（TRUE或FALSE）的表达式。
- **result1**, **result2**, ... 是当对应的条件为真时返回的结果。
- **default_result** 是当没有任何条件匹配时返回的默认结果。**ELSE**部分是可选的。
```sql
CASE 
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    ...
    ELSE default_result
END
```
### 示例
```sql
# 根据传入的月份，判定月份所属的季节（要求采用case结构）。
# 1-3月份，为第一季度
# 4-6月份，为第二季度
# 7-9月份，为第三季度
# 10-12月份，为第四季度
create procedure p4(in mouth int, out result varchar(10))
begin
    case
        when mouth >= 1 and mouth <= 3 then set result := '第一季度';
        when mouth >= 4 and mouth <= 6 then set result := '第二季度';
        when mouth >= 7 and mouth <= 9 then set result := '第三季度';
        when mouth >= 10 and mouth <= 12 then set result := '第四季度';
        else set result := '非法参数';
        end case;
end;

call p4(17, @result);
select @result;
```
## WHILE
**WHILE**语句用于实现循环结构，允许在满足特定条件时重复执行一组语句，直到条件不再满足为止。

- **condition** 是一个条件表达式，可以是任何返回布尔值（TRUE或FALSE）的表达式。
- 循环体中的代码块会被重复执行，直到条件不再满足为止。
```sql
WHILE condition DO
    -- 循环执行的代码块
END WHILE;
```
### 示例
```sql
# 计算从1累加到n的值，n为传入的参数值。
-- 先判定条件，如果条件为true，则执行逻辑，否则，不执行逻辑
-- A. 定义局部变量, 记录累加之后的值;
-- B. 每循环一次, 就会对n进行减1 , 如果n减到0, 则退出循环
create procedure p5(n int)
begin
    declare total int default 0;

    while n > 0
        do
            set total := total + n;
            set n := n - 1;
        end while;

    select total;
end;

call p5(1000);
```
## **REPEAT**
**REPEAT**语句用于实现循环结构，允许在满足特定条件时重复执行一组语句，直到条件不再满足为止。与**WHILE**循环不同，**REPEAT**循环的循环体至少会被执行一次，然后再根据条件决定是否继续执行。
```sql
REPEAT
    -- 循环执行的代码块
UNTIL condition
END REPEAT;
```
### 示例
```sql
# 计算从1累加到n的值，n为传入的参数值。(使用repeat实现)
create procedure p6(n int)
begin
    declare total int default 0;
    repeat
        set total := total + n;
        set n := n - 1;
    until n <= 0
        end repeat;
    select total;
end;

call p6(100);
```
## LOOP
**LOOP**语句用于创建一个无限循环，直到通过条件退出循环。与**WHILE**和**REPEAT**循环不同，**LOOP**循环没有预定义的退出条件，需要在循环体内部使用**LEAVE**语句或条件判断来手动退出循环。

- 循环体中的代码块会被无限重复执行，直到遇到**LEAVE**语句或条件判断使得循环退出。
- **ITERATE**用于控制循环体内部的执行流程，当满足某些条件时，可以立即跳过当前循环的剩余代码，直接进入下一次循环
- **condition** 是一个可选的条件表达式，如果满足条件，则执行**LEAVE**语句退出循环。
```sql
LOOP
    -- 循环执行的代码块
    IF condition THEN
        LEAVE;
    END IF;

    IF condition2 THEN
        ITERATE;
    END IF;
END LOOP;

```
### 示例
```sql
# 计算从1到n之间的偶数累加的值，n为传入的参数值。
-- A. 定义局部变量, 记录累加之后的值;
-- B. 每循环一次, 就会对n进行-1 , 如果n减到0, 则退出循环 ----> leave xx
-- C. 如果当次累加的数据是奇数, 则直接进入下一次循环. --------> iterate xx
create procedure p8(n int)
begin
    declare total int default 0;

    sum:
    loop
        if n <= 0 then
            leave sum;
        end if;

        if n % 2 = 1 then
            set n := n - 1;
            iterate sum;
        end if;

        set total := total + n;
        set n := n - 1;
    end loop sum;

    select total;
end;

call p8(100);
```
## 游标
在MySQL中，**游标（Cursor）**是一种用于在查询结果集中逐行遍历数据的数据库对象。它可以被视为指向结果集中当前行的指针，允许用户在结果集中进行逐行处理数据。
```sql
// 声明游标
declare 游标名称 cursor for 查询语句；

// 打开游标
open 游标名称；

// 获取游标记录
fetch 游标名称 into 变量 [,变量];

// 关闭游标
close 游标名称；
```
### 实例
```sql
# 根据传入的参数uage，来查询用户表tb_user中，所有的用户年龄小于等于uage的用户姓名
# （name）和专业（profession），并将用户的姓名和专业插入到所创建的一张新表
# (id,name,profession)中。
-- 定义临时变量存储数据
-- 声明游标，存储查询结果集
-- 准备创建表结构
-- 开启游标
-- 获取游标中的数据
-- 插入数据到新表
-- 关闭游标
create procedure p9()
begin
    declare uname varchar(100);
    declare upro varchar(100);
    #开启游标
    declare my_cursor cursor for select name, profession from tb_user;

    #创建表结构
    drop table if exists tb_user_pro;
    create table if not exists tb_user_pro
    (
        id         int primary key auto_increment,
        name       varchar(100),
        profession varchar(100)
    );

    #开启游标
    open my_cursor;

    while true
        do
            #获取游标记录
            fetch my_cursor into uname,upro;

            #插入记录
            insert into tb_user_pro values (null, uname, upro);
        end while;

    #关闭游标
    close my_cursor;
end;

call p9();
```
## 条件处理程序
条件处理程序（Handler）可以用来定义在流程控制结构执行过程中遇到问题时相应的处理步骤。具体语法为：
```sql
DECLARE   handler_action   HANDLER FOR    condition_value  [, condition_value] ...   statement ;
handler_action 的取值：
    CONTINUE: 继续执行当前程序
    EXIT: 终止执行当前程序
    
condition_value 的取值：
    SQLSTATE  sqlstate_value: 状态码，如: 02000

    
    SQLWARNING: 所有以01开头的SQLSTATE代码的简写
    NOT FOUND: 所有以02开头的SQLSTATE代码的简写
    SQLEXCEPTION: 所有没有被SQLWARNING 或 NOT FOUND捕获的SQLSTATE代码的简写
```
