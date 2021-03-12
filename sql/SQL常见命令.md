# SQL

## 一、查询

### SELECT 顺序

1. SELECT 返回表达式
2. FROM 数据表
3. WHERE 行级过滤
4. GROUP BY 分组说明
5. HAVING 组级过滤
6. ORDER BY 输出排序
7. LIMIT 分页查询

### 1、条件查询 WHERE LIKE

```sql
SELECT * FROM <表名> WHERE <条件表达式>
```

* 条件表达式可以用 AND/OR 连接多个条件
* 条件表达式前面加NOT表示非

```sql
SELECT * FROM <表名> WHERE name LIKE 'jet%'
```

* 搜索匹配表达式
* %匹配多个任意字符
* _匹配任意一个字符



### 2、排序 ORDER BY

```sql
ORDER BY score DESC, gender
```

* DESC代表降序，默认ACS升序
* ORDER BY 要放到WHERE后面

### 3、分页查询 LIMIT

```sql
LIMIT <M> OFFSET <N>
LIMIT <N>,<M>
```

* 取出从N条开始的M条结果，N索引从0开始
* `LIMIT`总是设定为`pageSize`；
* `OFFSET`计算公式为`pageSize * (pageIndex - 1)`。

### 4、聚合函数 GROUP BY

```sql
SELECT COUNT(*) num FROM students GROUP BY <COL> HAVING <ex>;
```

| 函数  | 说明                                   |
| :---- | :------------------------------------- |
| SUM   | 计算某一列的合计值，该列必须为数值类型 |
| AVG   | 计算某一列的平均值，该列必须为数值类型 |
| MAX   | 计算某一列的最大值                     |
| MIN   | 计算某一列的最小值                     |
| COUNT | 计算某一列数量                         |

* DISTINCT 用于只统计不同值
* GROUP BY 在WHERE之后 ORDER BY 之前
* HAVING 过滤分组

### 5、多表查询

```sql
SELECT * FROM table1,table2
```

* 实现表一，表二卡迪尔积查询

### 6、连接查询 JOIN

```sql
SELECT * FROM table1 JOIN table2 ON table1.id=table2.id
```

* 实现结果集的链接

### 7、正则表达式  REGEXP

```sql
SELECT * FROM <表名> WHERE name REGEXP <正则表达式>
```

* A|B   匹配A或B
* [123] 匹配123之一

- [0-9] 匹配0-9
- \\\ .  匹配'.'
- 匹配字符类
  - [:alnum:] 任意字母和数字（同[a-zA-Z0-9]）
  - [:alpha:] 任意字符（同[a-zA-Z]）
  - [:blank:] 空格和制表（同[\\t]）
  - [:cntrl:] ASCII控制字符（ASCII 0到31和127）
  - [:digit:] 任意数字（同[0-9]）
  - [:graph:] 与[:print:]相同，但不包括空格
  - [:lower:] 任意小写字母（同[a-z]）
  - [:print:] 任意可打印字符
  - [:punct:] 既不在[:alnum:]又不在[:cntrl:]中的任意字符
  - [:space:] 包括空格在内的任意空白字符（同[\\f\\n\\r\\t\\v]）
  - [:upper:] 任意大写字母（同[A-Z]）
  - [:xdigit:] 任意十六进制数字（同[a-fA-F0-9]）
- 匹配多实例
  -  ‘*’ 匹配0或多个
  - '+' 匹配1或多个
  - '?' 匹配0或1个
  - {n} 匹配n个
  - {n,} 匹配不少于n个
  - {n,m} 匹配n-m个
- 定位符
  - ^ 文本开头
  - $ 文本结尾
  - [[:<:]] 词开始
  - [[:>:]] 词结尾

### 8、创建字段

#### 拼接字段Concat

```sql
#Mysql
Concat(vend_name, '(',vend_country,')')
#Other
vend_name||'('||vend_country||')'
```

#### 算术计算 + - * /

