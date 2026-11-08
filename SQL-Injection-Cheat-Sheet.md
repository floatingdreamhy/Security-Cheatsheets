---

# SQL 注入速查表

本 SQL 注入速查表包含一些有用语法的示例，可用于执行 SQL 注入攻击中经常出现的各种任务。

## 字符串拼接

你可以将多个字符串拼接成一个字符串。

| 数据库 | 语法 |
|---|---|
| Oracle | `'foo'\|\|'bar'` |
| Microsoft | `'foo'+'bar'` |
| PostgreSQL | `'foo'\|\|'bar'` |
| MySQL | `'foo' 'bar'`（注意两个字符串之间的空格）<br>`CONCAT('foo','bar')` |

## 子字符串

你可以从指定偏移量开始、按指定长度提取字符串的一部分。注意：偏移量索引从 1 开始。以下每个表达式都会返回字符串 `ba`。

| 数据库 | 语法 |
|---|---|
| Oracle | `SUBSTR('foobar', 4, 2)` |
| Microsoft | `SUBSTRING('foobar', 4, 2)` |
| PostgreSQL | `SUBSTRING('foobar', 4, 2)` |
| MySQL | `SUBSTRING('foobar', 4, 2)` |

## 注释

你可以使用注释来截断查询，并删除原始查询中位于你的输入之后的部分。

| 数据库 | 语法 |
|---|---|
| Oracle | `--comment` |
| Microsoft | `--comment`<br>`/*comment*/` |
| PostgreSQL | `--comment`<br>`/*comment*/` |
| MySQL | `#comment`<br>`-- comment`（注意双横线后有一个空格）<br>`/*comment*/` |

## 绕过过滤器

如果应用程序阻止了特定关键字或字符，你可能仍然可以使用以下技术注入 SQL。基于编码的绕过方式，例如对关键字进行 URL 编码或使用 `CHAR()` 构造字符串，会在“使用编码混淆攻击”中单独介绍。

SQL 关键字不区分大小写，因此你可以改变输入的大小写，以绕过阻止特定关键字的过滤器。例如，如果 `SELECT` 被阻止，可以尝试 `SeLeCT`。

你还可以使用内联注释代替空白，这样，查找“关键字前后带空格”的过滤器就不会匹配。例如：

```sql
SELECT/**/username/**/FROM/**/users
