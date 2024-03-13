MySQL如何实现全文搜索

### 1、如何进行全文搜索

​	在MySQL中实现全文搜索功能，可以使用MySQL提供的全文搜索索引（Full-Text-Index）进行操作。

​	首先，需要在存储博客内容的表中创建一个全文搜索索引，假设一张名为posts的表，其中把汗了title和content的字段，你可以在这两个字段上面创建全文搜索索引；`ALTER TABLE posts ADD FULLTEXT(post_title);ALTER TABLE posts ADD FULLTEXT(post_content);`

​	其次，在创建完全文搜索索引之后，你就可以使用 MATCH() 和 AGAINST() 函数进行全文搜索 `SELECT * FROM posts WHERE MATCH(title, content) AGAINST('关键词' IN BOOLEAN MODE) ORDER BY MATCH(title, content) AGAINST('关键词' IN BOOLEAN MODE) DESC;`

### 2、MATCH() 和 AGAINST() 的用法

​	在MySQL中，MATCH() 和 AGAINST() 是用于执行全文搜索的函数，它接受一个搜索表达式和一个搜索字符串，并返回相关性得分。

函数的基本语法如下`MATCG(columns) AGAINST(search_string [search_modifier])`

- columns 是一个或者多个列，用于指定要进行全文搜索的字段
- search_string 是要搜索的关键词或者短语
- search_modifier 是可选的搜索修饰符，用于指定搜索模式和行为。常见的修饰符有：
  - IN NATURAL LANGUAGE MODE: 使用自然语言模式进行搜索，并根据相关性对结果进行排序
  - IN BOOLEAN MODE：使用布尔模式进行搜索，可以使用`+、-`等前缀来指定关键词的要求或者排除
  - WITH QUERY EXPANSION：在自然语言模式下，扩展搜索查询以包括相关的词汇

​	我给出一个示例来解释这两个函数的主要用法：

​	`SELECT * FROM posts WHERE MATCH(post_title, post_content) AGAINST('java' IN BOOLEAN MODE) ORDER BY MATCH(post_title, post_content) AGAINST('io' IN BOOLEAN MODE) DESC;`

- `SELECT *`: 这部分指定了查询的结果集。在这里，使用通配符 "*" 表示选择所有的列。你也可以指定具体的列名，例如 `SELECT column1, column2`
- `FROM posts`: 这部分指定了要从哪个表进行查询。在这里，我们使用名为 "posts" 的表。
- `WHERE MATCH(post_title, post_content) AGAINST('java' IN BOOLEAN MODE)`: 这是查询的条件部分。它使用 `MATCH() AGAINST()` 函数来进行全文搜索。`post_title` 和 `post_content` 是要进行搜索的字段，并在这两个字段上创建了全文搜索索引。
  - `MATCH(post_title, post_content)` 指定了要在哪些字段上执行全文搜索。
  - `AGAINST('java' IN BOOLEAN MODE)` 指定了要搜索的关键词为 "java"，并使用布尔模式进行搜索。
- `ORDER BY MATCH(post_title, post_content) AGAINST('io' IN BOOLEAN MODE) DESC`: 这部分用于指定结果的排序方式。它基于关键词 "io" 的相关性得分进行降序排序。
  - `MATCH(post_title, post_content) AGAINST('io' IN BOOLEAN MODE)` 用于获取与关键词 "io" 的相关性得分。
  - `DESC` 表示按相关性得分降序排序。你也可以使用 `ASC` 来进行升序排序。

### 3、解决全文搜索只能搜索4字符及其以上

​		在某些情况下，全局搜索可能无法搜索两个字符的原因是因为 MySQL 的全文搜索引擎默认情况下不支持搜索短词或短字符。这是由于全文搜索引擎的设计原则和性能考虑。

​	MySQL 的全文搜索引擎使用了一个称为 "**全文搜索索引**"（Full-Text Index）的特殊索引来加速搜索。然而，为了提高性能和减少索引大小，MySQL 默认情况下会忽略或过滤掉较短的词语或字符。

​	默认情况下，MySQL 全文搜索引擎的最小搜索词长度是由参数 `ft_min_word_len` 控制的，通常设置为 3。这意味着，如果一个词或字符的长度小于该参数的值，它将被视为停用词而不会被索引和搜索。

​	如果你希望能够搜索两个字符或更短的词语，你可以尝试调整 `ft_min_word_len` 参数的值。请注意，修改该参数需要编辑 MySQL 的配置文件（例如 my.cnf 或 my.ini）并重启 MySQL 服务才能生效。

```sql
ft_min_word_len = 1
ft_max_word_len = 84
innodb_ft_enable_stopword = OFF
```
​	当你完成这些配置任然没有生效的话，你需要在SQL中执行此命令`SHOW GLOBAL VARIABLES LIKE '%ft_m%'`，从而查询出你的设置是否生效.

```
ft_max_word_len 84
ft_min_word_len	1
innodb_ft_max_token_size	84
innodb_ft_min_token_size	1  
```

**注：有些时候配置未生效是因为 innodb_ft_min_token_size 没有设置，因为这参数是 InnoDB 存储引擎特有的。你需要根据自己的数据库进行选择**