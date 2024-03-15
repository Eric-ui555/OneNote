## 简单查询

### 数据表

```sql
-- order_items
+--------------+-----------------+-----------------+-----------------+
| order_id     | product_id      |quantity         | unit_price      |
+--------------+-----------------+-----------------+-----------------+
| 2            | 1               | 2               | 9.10            |
| 2            | 4               | 4               | 1.66            |
| 2            | 6               | 2               | 2.94            |
+--------------+-----------------+-----------------+-----------------+
```

### Like运算符

检索遵循特定字符串模式的行

```sql
SELECT *
FROM customers
WHERE last_name LIKE 'b%'  -- 以b开头

-- 'b%': 以b开头
-- '%b': 以b结尾
-- '%b%': 任意包含b
-- '_y': 表示第二个字符是y
```

`%`：表示**任意字符数**

`_`：表示**单字符**，可以有任意数量

```sql
-- practice
-- Get THE customers whose
-- address contain TRAIL or AVENUE
-- phone numbers end with 9

SELECT * FROM customers where address like '%TRAIL%' OR  address like '%AVENUE%'
SELECT * FROM customers where phone like '%9'
```

### REGEXP运算符

```sql
SELECT *
FROM customers
WHERE last_name REGEXP 'field'
```

`^`：表示字符串开头， `^field`表示要以filed开头；

`$`：表示字符串末尾， `field$`：表示以field结尾；

`|`：表示逻辑上的OR，多个搜寻模式，`field|mac`：表示查询包含field或mac；

`[]`：表示匹配任意在括号里列举的单字符，[gim]e：表示ge、ie、me

`-`: 表示范围，[a-h]e：可以表示从ae，be，ce，...，he

```sql
-- practice
-- Get the customers whose
-- first names are ELKE or AMBUR
-- last names end with EY or ON
-- laste names starts with MY or contains SE
-- last names contains B followed by R or U

SELECT * FROM costomers where first_name REGEXP 'ELKA|AMBUR'
SELECT * FROM costomers where last_name REGEXP 'EY$|ON$'
SELECT * FROM costomers where last_name REGEXP '^MY|SE'
SELECT * FROM costomers where last_name REGEXP 'B[RU]'
```

IS NULL运算符

```sql
SELECT *
FROM customers
WHERE phone IS NULL
```

```sql
-- Get the customers that are not shipped

SELECT * FROM orders where shipped_id IS NULL
```

### ORDER BY子句

```sql
SELECT *
FROM customers
ORDER BY state, first_name 

-- 默认升序
-- DESC：降序
-- 多列排序
```

```sql
-- Get the orders_item whose
-- order_id is 2 and desc order by quantity * unit_price

SELECT *, quatity * unit_price AS total_price
FROM order_items 
where order_id = 2 
ORDER BY total_price DESC
```

### LIMIT子句

返回限定查询的记录

```sql
-- 返回前三条记录
SELECT *
FROM customers
LIMIT 3

-- Pages 6-9: LIMIT 6,3
-- LIMIT子句要永远放到最后
```

```sql
-- Get the top three loyal customers

SELECT *
FROM customers
ORDER BY points DESC
LIMIT 3
```



## 插入更新

### 插入分层行

新增一个订单（order），里面包含两个订单项目/两种商品（order_items），请同时更新订单表和订单项目表

```sql
USE sql_store;

INSERT INTO orders (customer_id, order_date, status)
VALUES (1, '2019-01-01', 1);

-- 可以先试一下用 SELECT last_insert_id() 看能否成功获取到的最新的 order_id

INSERT INTO order_items  -- 全是必须字段，就不用指定了
VALUES 
    (last_insert_id(), 1, 2, 2.5),
    (last_insert_id(), 2, 5, 1.5)
```

### 创建表复制

运用 `CREAT TABLE 新表名 AS 子查询` 快速创建表 orders 的副本表 orders_archived

```sql
CREATE TABLE orders_archived AS
    SELECT * FROM orders  -- 子查询
```

子查询： 任何一个充当另一个`SQL`语句的一部分的 `SELECT……` 查询语句都是子查询，子查询是一个很有用的技巧。

不再用全部数据，而选用原表中部分数据创建副本表，如，用今年以前的 orders 创建一个副本表 orders_archived，其实就是在子查询里增加了一个WHERE语句进行筛选。注意要先 drop 删掉 或 truncate 清空掉之前建的 orders_archived 表再重建或重新插入数据。

法1. `DROP TABLE 要删的表名`、`CREATE TABLE 新表名 AS 子查询`

```sql
DROP TABLE orders_archived;  -- 也可右键该表点击 drop
CREATE TABLE orders_archived AS
    SELECT * FROM orders
    WHERE order_date < '2019-01-01'
```

### 更新多行

让所有非90后顾客的积分增加50点

```sql
UPDATE customers
SET points = points + 50
WHERE birth_date < '1990-01-01'
```

### 在Updates中用子查询

将 orders 表里那些 `分数>3k` 的用户的订单 comments 改为 'gold customer'

思考步骤：

1. WHERE 行筛选出要求的顾客
2. SELECT 列筛选他们的id
3. 将前两步 作为子查询 用在修改语句中的 WHERE 条件中，执行修改

```sql
USE sql_store;

UPDATE orders
SET comments = 'gold customer'
WHERE customer_id IN
    (SELECT customer_id
    FROM customers
    WHERE points > 3000)
```

### 删除行

选出顾客id为3/顾客名字叫'Myworks'的发票记录

```sql
USE sql_invoicing;

DELETE FROM invoices
WHERE client_id = 3
-- WHERE可选，省略就是会删除整个表的所有行/记录
/WHERE client_id = 
    (SELECT client_id  
    /*Mosh 错写成了 SELECT *，将报错：
    Error Code: 1241. Operand should contain 1 column(s) 
    Operand n. [计] 操作数；[计] 运算对象；运算元 */
    FROM clients
    WHERE name = 'Myworks')
```

## 连接语句

### 数据表

```sql
-- order_items
+--------------+-----------------+-----------------+-----------------+
| order_id     | product_id      | quantity        | unit_price      |
+--------------+-----------------+-----------------+-----------------+
| 1            | 1               | 2               | 9.10            |
| 2            | 2               | 4               | 1.66            |
| 3            | 3               | 2               | 2.94            |
+--------------+-----------------+-----------------+-----------------+
```

```sql
-- product
+--------------+------------------+-----------------+-----------------+
| product_id   | name             |quantity_in_stock| unit_price      |
+--------------+------------------+-----------------+-----------------+
| 1            | Foam DInner Plate| 70              | 1.21            |
| 2            | Prok-Bacon       | 49              | 4.65            |
| 3            | Letture          | 38              | 3.35            |
+--------------+------------------+-----------------+-----------------+
```

### 内连接|Inner Joins

```sql
SELECT * 
FROM orders
JOIN customers 
	ON orders.customer_id = customers.customer_id
```

```sql
-- 连接product表
-- 每笔订单都返回产品id和名字，连通order_items表的数量和单价

SELECT oi.order_id, oi.product_id, quatity, oi.unit_prices
FROM order_items oi
JOIN product p 
	ON oi.product_id = p.product_id 
```

### 跨数据库连接|Joining Across Database

```sql
-- 当前order_items表连接sql_invertory数据库中的product表

SELECT *
FROM order_items oi
JOIN sql_invertory.product p 
	ON oi.product_id = p.product_id 
	
-- 在表之前添加前缀即可
```

### 自连接|Self Joins

```sql
-- employee
+--------------+------------+------------+-----------+-----------+-------------+-----------+
| employee_id  | first_name | last_name  | job_title | salary    | reports_to  | office_id |
+--------------+------------+------------+-----------+-----------+-------------+-----------+
```

```sql
-- employer 选择每个员工和他们的管理人员的名字

SELECT 
	e.employee_id,
	e.first_name ,
	m.first_name AS manager
FROM employee e
JOIN employee m
 ON e.reports_to = m.employee_id
```

### 多表连接|Joining Mutiple Tables

```sql
SELECT
	o.order_id,
	o.order_date,
	o.first_name,
	o.last_name,
	os.name AS status
FROM orders o
JOIN customers c
	on o.customer_id = c.customer_id
JOIN order_statuses os
	on o.status = os.order_status_id

-- 可以使用多个JOIN
```

### 复合连接条件

```sql
SELECT *
FROM order_items oi
JOIN order)item_notes oin
	ON oi.order_id = oin.order_id
	AND oi.product_id = oin.product_id 
	
-- 使用AND进行复合连接
```

### 隐式连接条件

```sql
SELECT * 
FROM orders
JOIN customers 
	ON orders.customer_id = customers.customer_id
	
-- 要用显式连接，而不是交叉连接

-- 以下是交叉连接
SELECT * 
FROM order o, customers c
WHERE p.customer_id = c.customer_id
```

外连接

左外连接：返回**左表中的所有行**，如果左表中行在右表中没有匹配行，则结果中右表中的列返回空值。

右外连接：恰与左连接相反，**返回右表中的所有行**，如果右表中行在左表中没有匹配行，则结果中左表中的列返回空值。

全连接：返回左表和右表中的所有行。当某行在另一表中没有匹配行，则另一表中的列返回空值

### 左外连接

```sql
SELECT
	p.product_id,
	p.name,
	oi.quantity
FROM products p
LEFT JOIN order_items oi
	ON p.product_id = oi.product_id
```

### 多表外连接

```sql
--  多表外连接
SELECT 
	o.order_id,
	o.order_date,
	c.first_name,
	s.name
FROM orders o
Left JOIN customers c on  o.customer_id = c.customer_id
LEFT JOIN shippers s on o.shipper_id = s.shipper_id;
```

### 自外连接

```sql
--  自外连接
SELECT
	e.employee_id,
	e.first_name AS employee,
	e.salary,
	m.first_name AS manager 
FROM
	employees e
	LEFT JOIN employees m ON e.reports_to = m.employee_id
```

### USING子句

`on  o.customer_id = c.customer_id`二者相同时可以用`USING (customer_id )`代替

```sql
-- USING 子句
SELECT
	o.order_id,
	o.order_date,
	c.first_name,
	s.NAME 
FROM
	orders o
	LEFT JOIN customers c USING(customer_id)
	LEFT JOIN shippers s USING (shipper_id);
```

```sql
-- USING 子句
SELECT
	p.date,
	c.name as client,
	p.amount,
	pm.name as payment_method
FROM
	payments p
JOIN clients c USING(client_id)
JOIN payment_methods pm on p.payment_method = pm.payment_method_id
```

### 自然连接

特殊的等值连接，他要求两个关系中进行比较的分量必须是想用的属性组，并且在结果中把重复的属性列去掉。

不建议使用，因为无法控制

```sql
-- 自然连接
SELECT
	o.order_id,
	c.first_name
FROM orders o
NATURAL JOIN customers c;
```

### 交叉连接

返回所有可能组合的结果

```sql
-- 交叉连接 
-- 显式
SELECT
	c.first_name,
	p.NAME 
FROM
	customers c
	CROSS JOIN products p
	
-- 隐式
SELECT
	c.first_name,
	p.NAME 
FROM
	customers c，products p
```

### 联合|Union

`union` 连接两段select查询，要求两段查询返回的**列必须相同**

```sql
-- UNION
SELECT
	o.order_id,
	o.order_date,
	"Actived" AS STATUS 
FROM
	orders o 
WHERE
	o.order_date >= "2019-01-01" UNION
SELECT
	o.order_id,
	o.order_date,
	"InActived" AS STATUS 
FROM
	orders o 
WHERE
	o.order_date < "2019-01-01"
```

![image-20240108200253015](高级查询.assets/image-20240108200253015.png)

## 聚合函数

### 数值函数|Numeric Function

MAX()

MIN()

AVG()

SUM()

COUNT()

```sql
SELECT
	MAX( payment_total ) AS highest,
	MIN( payment_total ) AS lowest,
	AVG( payment_total ) AS average,
	SUM( payment_total ) AS total,
	COUNT( payment_total ) AS count
FROM
	invoices;
```

### GROUP BY子句

```sql
SELECT
	client_id,
	SUM( payment_total ) AS total_sales 
FROM
	invoices 
GROUP BY
	client_id;
```

在 payments 表中，按日期和支付方式分组统计总付款额

每个分组显示一个日期和支付方式的独立组合，可以看到某特定日期特定支付方式的总付款额。这个例子里每一种支付方式可以在不同日子里出现，每一天也可以出现多种支付方式，这种情况，才叫真·多字段分组。不过上一个例子里那种假·多字段分组，把 state 加在分组依据里也没坏处还能落个心安，也还是加上别省比较好

```sql
SELECT 
    date, 
    pm.name AS payment_method,
    SUM(amount) AS total_payments
FROM payments p
JOIN payment_methods pm
    ON p.payment_method = pm.payment_method_id
GROUP BY date, payment_method
-- 用的是 SELECT 里的列别名
ORDER BY date
```

### HAVING子句

HAVING 和 WHERE 都是是条件筛选语句，条件的写法相通，数学、比较（包括特殊比较）、逻辑运算都可以用（如 AND、REGEXP 等等）

两者本质区别:

- WHERE 是对 FROM JOIN 里原表中的列进行 **事前筛选**，所以WHERE可以对没选择的列进行筛选，但必须用原表列名而不能用SELECT中确定的列别名
- 相反 HAVING …… 对 SELECT …… 查询后（**通常是分组并聚合查询后**）的结果列进行 事后筛选，若SELECT里起了别名的字段则**必须用别名进行筛选**，且**不能对SELECT里未选择的字段进行筛选**。唯一特殊情况是，当HAVING筛选的是聚合函数时，该聚合函数可以不在SELECT里显性出现，见最后补充

**案例**：筛选出总发票金额大于500且总发票数量大于5的顾客

```sql
USE sql_invoicing;

SELECT 
    client_id,
    SUM(invoice_total) AS total_sales,
    COUNT(*/invoice_total/invoice_date) AS number_of_invoices
FROM invoices
GROUP BY client_id
HAVING total_sales > 500 AND number_of_invoices > 5
-- 均为 SELECT 里的列别名
```

练习：在 sql_store 数据库（有顾客表、订单表、订单项目表等）中，找出在 'VA' 州且消费总额超过100美元的顾客（这是一个面试级的问题，还很常见）

思路：

1. 需要的信息在顾客表、订单表、订单项目表三张表中，先将三张表合并
2. WHERE 事前筛选 'VA' 州的
3. 按顾客分组，并选取所需的列并聚合得到每位顾客的付款总额
4. HAVING 事后筛选超过 100美元 的

```sql
SELECT
	c.customer_id,
	c.first_name,
	c.last_name,
	SUM( oi.quantity * oi.unit_price ) AS total_price 
FROM
	orders o
	JOIN customers c USING ( customer_id )
	JOIN order_items oi USING ( order_id ) 
WHERE
	state = 'VA' 
GROUP BY
	c.customer_id,
	c.first_name,
	c.last_name 
HAVING
	total_price >= 100
```

**补充**

**学第六章第6节时发现，当 HAVING 筛选的是聚合函数时，该聚合函数可以不在SELECT里显性出现。（作为一种需要记住的特殊情况）**如：下面这两种写法都能筛选出总点数大于3k的州，如果不要求显示总点数，应该用后一种

```sql
SELECT state, SUM(points)
FROM customers
GROUP BY state
HAVING SUM(points) > 3000

或

SELECT state
FROM customers
GROUP BY state
HAVING SUM(points) > 3000
```

### `ROLLUP`运算符

`GROUP BY …… WITH ROLL UP` 自动汇总型分组，若是多字段分组的话汇总也会是多层次的，注意这是**MySQL扩展语法**，不是SQL标准语法

**案例**

分组查询各客户的发票总额**以及所有人的总发票额**

```sql
USE sql_invoicing;

SELECT 
    client_id,
    SUM(invoice_total)
FROM invoices
GROUP BY client_id WITH ROLLUP
```

多字段分组 例1：分组查询各州、市的总销售额（发票总额）以及州层次和全国层次的**两个层次的汇总额**

```sql
SELECT 
    state,
    city,
    SUM(invoice_total) AS total_sales
FROM invoices
JOIN clients USING (client_id) 
GROUP BY state, city WITH ROLLUP
```

**练习**

分组计算各个付款方式的总付款 并汇总

```sql
SELECT 
    pm.name AS payment_method,
    SUM(amount) AS total
FROM payments p
JOIN payment_methods pm
    ON p.payment_method = pm.payment_method_id
GROUP BY pm.name WITH ROLLUP
```

**★总结**

根据之后三篇参考文章，据说标准的 SQL 查询语句的执行顺序应该是下面这样的：

1. FROM JOIN 选择和连接本次查询所需的表
2. ON/USING WHERE 按条件筛选行
3. GROUP BY 分组
4. HAVING （事后/分组后）筛选行
5. SELECT 筛选列
   - 注意1：若进行了分组，这一步常常要聚合）
   - 注意2：SELECT 和 HAVING 在 MySQL 里的执行顺序我还有点疑问，见后面的叙述
6. DISTINCT 去重
7. UNION 纵向合并
8. ORDER BY 排序
9. LIMIT 限制

## 复杂查询

### 子查询

```sql
-- Find products that are more expensive than Letture(id=3)
SELECT
	* 
FROM
	products 
WHERE
	unit_price > (
		SELECT
			unit_price 
		FROM
			products 
		WHERE
		product_id = 3 
)
```

```sql
-- Find employees whose earn more than average
SELECT
	* 
FROM
	employees 
WHERE
	salary > ( SELECT AVG( salary ) FROM employees )
```

### IN 运算符

在 sql_store 库 products 表中找出那些从未被订购过的产品

```sql
USE sql_store;

SELECT *
FROM products
WHERE product_id NOT IN (
    SELECT DISTINCT product_id
    FROM order_items
)
```

**练习**: 在 sql_invoicing 库 clients 表中找到那些没有过发票记录的客户

思路：和上一个例子完全一致，在invoices里用DISTINCT找到所有有过发票记录的客户列表，再用NOT IN来筛选

```sql
USE sql_invoicing;

SELECT *
FROM clients
WHERE client_id NOT IN (
    SELECT DISTINCT client_id
    FROM invoices
)
```

### 子查询vs连接

Subqueries vs Joins (5:07)

**小结**

子查询（Subquery）是将一张表的查询结果作为另一张表的查询依据并层层嵌套，其实也可以先将这些表链接（Join）合并成一个包含所需全部信息的详情表再直接在详情表里筛选查询。两种方法一般是可互换的，具体用哪一种取决于 **效率/性能（Performance） 和 可读性（readability）**，之后会学习 执行计划，到时候就知道怎样编写并更快速地执行查询，现在主要考虑可读性

**案例**

上节课的案例，找出从未订购（没有invoices）的顾客：

法1. 子查询

先用子查询查出有过发票记录的顾客名单，作为筛选依据

```sql
USE sql_invoicing;

SELECT *
FROM clients
WHERE client_id NOT IN (
    SELECT DISTINCT client_id
    /*其实这里加不加DISTINCT对子查询返回的结果有影响
    但对最后的结果其实没有影响*/
    FROM invoices
)
```

法2. 链接表

用顾客表 LEFT JOIN 发票记录表，再直接在这个合并详情表中筛选出没有发票记录的顾客

```sql
USE sql_invoicing;

SELECT DISTINCT client_id, name …… 
-- 不能SELECT DISTINCT *
FROM clients
LEFT JOIN invoices USING (client_id)
-- 注意不能用内链接，否则没有发票记录的顾客（我们的目标）直接就被筛掉了
WHERE invoice_id IS NULL
```

就上面这个案例而言，子查询可读性更好，但有时子查询会过于复杂（嵌套层数过多），用链接表更好（下面的练习就是）。总之在选择方法时，可读性是很重要的考虑因素

**练习**

在 sql_store 中，选出买过生菜（id = 3）的顾客的id、姓和名

分别用子查询法和链接表法实现并比较可读性

法1. 完全子查询

```sql
USE sql_store;

SELECT customer_id, first_name, last_name
FROM customers
WHERE customer_id IN (  
    -- 子查询2：从订单表中找出哪些顾客买过生菜
    SELECT customer_id
    FROM orders
    WHERE order_id IN (  
        -- 子查询1：从订单项目表中找出哪些订单包含生菜
        SELECT DISTINCT order_id
        FROM order_items
        WHERE product_id = 3
    )
)
```

法2. 混合：子查询 + 表连接

```sql
USE sql_store;

SELECT customer_id, first_name, last_name
FROM customers
WHERE customer_id IN (  
    -- 子查询：哪些顾客买过生菜
    SELECT customer_id
    FROM orders
    JOIN order_items USING (order_id)  
    -- 表连接：合并订单和订单项目表得到 订单详情表
    WHERE product_id = 3
)
```

法3. 完全表连接

直接链接合并3张表（顾客表、订单表和订单项目表）得到 带顾客信息的订单详情表，该合并表包含我们所需的所有信息，可直接在合并表中用WHERE筛选买过生菜的顾客（注意 DISTINCT 关键字的运用）。

```sql
USE sql_store;

SELECT DISTINCT customer_id, first_name, last_name
FROM customers
LEFT JOIN orders USING (customer_id)
LEFT JOIN order_items USING (order_id)
WHERE product_id = 3
```

这个案例中，先将所需信息所在的几张表全部连接合并成一张大表再来查询筛选明显比层层嵌套的多重子查询更加清晰明了

### ALL 关键字

**小结**

`> (MAX (……))` 和 `> ALL(……)` 等效可互换

“比这里面最大的还大” = “比这里面的所有的都大”

**案例**

sql_invoicing 库中，选出金额大于3号顾客所有发票金额（或3号顾客最大发票金额） 的发票

法1. 用MAX关键字

```sql
USE sql_invoicing;

SELECT *
FROM invoices
WHERE invoice_total > (
    SELECT MAX(invoice_total)
    FROM invoices
    WHERE client_id = 3
)
```

法2. 用ALL关键字

```sql
USE sql_invoicing;

SELECT *
FROM invoices
WHERE invoice_total > ALL (
    SELECT invoice_total
    FROM invoices
    WHERE client_id = 3
)
```

其实就是把内层括号的MAX拿到了外层括号变成ALL：
MAX法是用MAX()返回一个顾客3的最大订单金额，再判断哪些发票的金额比这个值大；
ALL法是先返回顾客3的所有订单金额，是一列值，再用ALL()判断比所有这些金额都大的发票有哪些。
两种方法是完全等效的

### ANY关键字

**案例1**

`> ANY (……)` 与 `> (MIN (……))` 等效的例子：
sql_invoicing 库中，选出金额大于3号顾客任何发票金额（或最小发票金额） 的发票

```sql
USE sql_invoicing;

SELECT *
FROM invoices

WHERE invoice_total > ANY (
    SELECT invoice_total
    FROM invoices
    WHERE client_id = 3
)

或

WHERE invoice_total > (
    SELECT MIN(invoice_total)
    FROM invoices
    WHERE client_id = 3
)
```

**案例2**

`= ANY (……)` 与 `IN (……)` 等效的例子:
选出至少有两次发票记录的顾客

```sql
USE sql_invoicing;

SELECT *
FROM clients
WHERE client_id IN (  -- 或 = ANY ( 
    -- 子查询：有2次以上发票记录的顾客
    SELECT client_id
    FROM invoices
    GROUP BY client_id
    HAVING COUNT(*) >= 2
)
```

### 相关子查询

之前都是非关联主/子（外/内）查询，比如子查询先查出整体的某平均值或满足某些条件的一列id，作为主查询的筛选依据，这种子查询与主查询无关，会先一次性得出查询结果再返回给主查询供其使用。

而下面这种相关联子查询例子里，子查询要查询的是某员工所在办公室的平均值，子查询是依赖主查询的，**注意这种关联查询是在主查询的每一行/每一条记录层面上依次进行的**，这一点可以为我们写关联子查询提供线索（注意表别名的使用），另外也正因为这一点，相关子查询会比非关联查询执行起来慢一些。

```sql
-- 查询薪水大于本部门平均值的员工
SELECT
	* 
FROM
	employees e 
WHERE
	salary > ( SELECT AVG( salary ) FROM employees WHERE e.office_id = office_id )
```

在 sql_invoicing 库 invoices 表中，找出高于每位顾客平均发票金额的发票

```sql
USE sql_invoicing;

SELECT *
FROM invoices i
WHERE  invoice_total > (
    -- 子查询：目前客户的平均发票额
    SELECT AVG(invoice_total)
    FROM invoices
    WHERE client_id = i.client_id
)
```

### EXISTS运算符

`IN + 子查询` 等效于 `EXIST + 相关子查询`，如果前者子查询的结果集过大占用内存，用后者逐条验证更有效率。另外 EXIST() 本质上是根据是否为空返回 TRUE 和 FALSE，所以也可以加 NOT 取反。

找出有过发票记录的客户，第4节学过用子查询或表连接来实现

法1. 子查询

```sql
USE sql_invoicing;

SELECT *
FROM clients
WHERE client_id IN (
    SELECT DISTINCT client_id
    FROM invoices
)
```

法2. 链接表

```sql
USE sql_invoicing;

SELECT DISTINCT client_id, name …… 
FROM clients
JOIN invoices USING (client_id)
-- 内链接，只留下有过发票记录的客户
```

第3种方法是用EXISTS运算符实现

```sql
USE sql_invoicing;

SELECT *
FROM clients c
WHERE EXISTS (
    SELECT client_id  
    /* 就这个子查询的目的来说，SELECT的选择不影响结果，
    因为EXISTS()函数只根据是否为空返回 TRUE 和 FALSE */
    FROM invoices
    WHERE client_id = c.client_id
)
```

### SELECT子句中的子查询

得到一个有如下列的表格：invoice_id, invoice_total, avarege（总平均发票额）, difference（前两个值的差）

```sql
USE sql_invoicing;

SELECT 
    invoice_id,
    invoice_total,
    (SELECT AVG(invoice_total) FROM invoices) AS invoice_average,
    /*不能直接用聚合函数，因为“比较强势”，会压缩聚合结果为一条
    用括号+子查询(SELECT AVG(invoice_total) FROM invoices) 
    将其作为一个数值结果 152.388235 加入主查询语句*/
    invoice_total - (SELECT invoice_average) AS difference
    /*SELECT表达式里要用原列名，不能直接用别名invoice_average
    要用列别名的话用子查询（SELECT 同级的列别名）即可
    说真的，感觉这个子查询有点难以理解，但记住会用就行*/
FROM invoices
```

**练习**

得到一个有如下列的表格：client_id, name, total_sales（各个客户的发票总额）, average（总平均发票额）, difference（前两个值的差）

```sql
USE sql_invoicing;

SELECT 
    client_id,
    name,
    (SELECT SUM(invoice_total) FROM invoices WHERE client_id = c.client_id) AS total_sales,
    -- 要得到【相关】客户的发票总额，要用相关子查询 WHERE client_id = c.client_id
    (SELECT AVG(invoice_total) FROM invoices) AS average,
    (SELECT total_sales - average) AS difference   
    /* 如前所述，引用同级的列别名，要加括号和 SELECT，
    和前两行子查询的区别是，引用同级的列别名不需要说明来源，
    所以没有 FROM …… */
FROM clients c
```

### FROM子句的子查询

将上一节练习里的查询结果当作来源表，查询其中 total_sales 非空的记录

```sql
USE sql_invoicing;

SELECT * 
FROM (
    SELECT 
        client_id,
        name,
        (SELECT SUM(invoice_total) FROM invoices WHERE client_id = c.client_id) AS total_sales,
        (SELECT AVG(invoice_total) FROM invoices) AS average,
        (SELECT total_sales - average) AS difference   
    FROM clients c
) AS sales_summury
/* 在FROM中使用子查询，即使用 “派生表” 时，
必须给派生表取个别名（不管用不用），这是硬性要求，不写会报错：
Error Code: 1248. Every derived table（派生表、导出表）
must have its own alias */
WHERE total_sales IS NOT NULL
```

复杂的子查询再嵌套进 FROM 里会让整个查询看起来过于复杂，上面这个最好是将子查询结果储存为叫 sales_summury 的视图，然后再直接使用该视图作为来源表，之后会讲。

## 内置函数

### 数值函数

```sql
SELECT ROUND(5.7365, 2)  -- 四舍五入
SELECT TRUNCATE(5.7365, 2)  -- 截断
SELECT CEILING(5.2)  -- 天花板函数，大于等于此数的最小整数
SELECT FLOOR(5.6)  -- 地板函数，小于等于此数的最大整数
SELECT ABS(-5.2)  -- 绝对值
SELECT RAND()  -- 随机函数，0到1的随机值
```

### 字符串函数 | String Function

[MySQL :: MySQL 8.0 Reference Manual :: 12.8 String Functions and Operators](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html)

查看全部搜索关键词 **`mysql string functions`**

长度、转大小写：

```sql
SELECT LENGTH('sky')  -- 字符串字符个数/长度（LENGTH）
SELECT UPPER('sky')  -- 转大写
SELECT LOWER('Sky')  -- 转小写
```

用户输入时时常多打空格，下面三个函数用于处理/修剪（trim）字符串前后的空格，L、R 表示 LEFT、RIGHT：

```sql
SELECT LTRIM('  Sky')  -- 去除左侧空格
SELECT RTRIM('Sky  ')  -- 去除右侧空格
SELECT TRIM(' Sky ')   -- 去除两侧空格
```

切片：

```sql
-- 取左边，取右边，取中间
SELECT LEFT('Kindergarden', 4)  -- 取左边（LEFT）4个字符
SELECT RIGHT('Kindergarden', 6)  -- 取右边（RIGHT）6个字符
SELECT SUBSTRING('Kindergarden', 7, 6)  
-- 取中间从第7个开始的长度为6的子串（SUBSTRING）
-- 注意是从第1个（而非第0个）开始计数的
-- 省略第3参数（子串长度）则一直截取到最后
```

定位：

```sql
SELECT LOCATE('gar', 'Kindergarden')  -- 定位（LOCATE）首次出现的位置
-- 没有的话返回0（其他编程语言大多返回-1，可能因为索引是从0开始的）
-- 这个定位/查找函数依然是不区分大小写的
```

替换：

```sql
SELECT REPLACE('Kindergarten', 'garten', 'garden')
```

连接：

```sql
USE sql_store;

SELECT CONCAT(first_name, ' ', last_name) AS full_name
-- concatenate v. 连接
FROM customers
```

### 日期函数

本节学基本的处理时间日期的函数，下节课学日期时间的格式化

1. NOW, CURDATE, CURTIME
2. YEAR, MONTH, DAY, HOUR, MINUTE, SECOND, DAYNAME, MONTHNAME
3. EXTRACT(单位 FROM 日期时间对象)， 如 EXTRACT(YEAR FROM NOW())

**实例**

当前时间

```sql
SELECT NOW()  -- 2020-09-12 08:50:46
SELECT CURDATE()  -- current date, 2020-09-12
SELECT CURTIME()  -- current time, 08:50:46
```

以上函数将返回时间日期对象

提取时间日期对象中的元素：

```sql
SELECT YEAR(NOW())  -- 2020
```

还有MONTH, DAY, HOUR, MINUTE, SECOND。

以上函数均返回整数，还有另外两个返回字符串的：

```sql
SELECT DAYNAME(NOW())  -- Saturday
SELECT MONTHNAME(NOW())  -- September
```

**标准SQL语句**有一个类似的函数 EXTRACT()，若需要在不同DBMS中录入代码，最好用EXTRACT()：

```sql
SELECT EXTRACT(YEAR FROM NOW())
```

当然第一参数也可以是MONTH, DAY, HOUR ……
总之就是：`EXTRACT(单位 FROM 日期时间对象)`

**练习**

返回【今年】的订单

用时间日期函数而非手动输入年份，代码更可靠，不会随着时间的改变而失效

```sql
USE sql_store;

SELECT * 
FROM orders
WHERE YEAR(order_date) = YEAR(now())
```

### 格式化日期和时间

`DATE_FORMAT(date, format)` 将 date 根据 format 字符串进行格式化。

`TIME_FORMAT(time, format)` 类似于 DATE_FORMAT 函数，但这里 format 字符串只能包含用于小时，分钟，秒和微秒的格式说明符。其他说明符产生一个 NULL 值或0。

```sql
SELECT DATE_FORMAT(NOW(), '%M %d, %Y')  -- September 12, 2020
-- 格式说明符里，大小写是不同的，这是目前SQL里第一次出现大小写不同的情况
SELECT TIME_FORMAT(NOW(), '%H:%i %p')  -- 11:07 AM
```

### 计算日期和时间

有时需要对日期事件对象进行运算，如增加一天或算两个时间的差值之类，介绍一些最有用的日期时间计算函数：

1. DATE_ADD, DATE_SUB
2. DATEDIFF
3. TIME_TO_SEC

增加或减少一定的天数、月数、年数、小时数等等

```sql
SELECT DATE_ADD(NOW(), INTERVAL -1 DAY)
SELECT DATE_SUB(NOW(), INTERVAL 1 YEAR)
```

但其实不用函数，直接加减更简洁：

```sql
NOW() - INTERVAL 1 DAY
NOW() - INTERVAL 1 YEAR 
```

计算日期差异

~~~sql
SELECT DATEDIFF('2019-01-01 09:00', '2019-01-05')  -- -4
-- 会忽略时间部分，只算日期差异

借助 TIME_TO_SEC 函数计算时间差异

TIME_TO_SEC：计算从 00:00 到某时间经历的秒数

```sql
SELECT TIME_TO_SEC('09:00')  -- 32400
SELECT TIME_TO_SEC('09:00') - TIME_TO_SEC('09:02')  -- -120
~~~

### `IFNULL`和`COALESCE`函数

两个用来替换空值的函数：IFNULL, COALESCE. 后者更灵活

将 orders 里 [shipper.id](https://link.zhihu.com/?target=http%3A//shipper.id/) 中的空值替换为 'Not Assigned'（未分配）

```sql
USE sql_store;

SELECT 
    order_id,
    IFNULL(shipper_id, 'Not Assigned') AS shipper
    /* If expr1 is not NULL, IFNULL() returns expr1; 
    otherwise it returns expr2. */
FROM orders
```

将 orders 里 [shipper.id](https://link.zhihu.com/?target=http%3A//shipper.id/) 中的空值替换为 comments，若 comments 也为空则替换为 'Not Assigned'（未分配）

```sql
USE sql_store;

SELECT 
    order_id,
    COALESCE(shipper_id, comments, 'Not Assigned') AS shipper
    /* Returns the first non-NULL value in the list, 
    or NULL if there are no non-NULLvalues. */
FROM orders
```

COALESCE 函数是返回一系列值中的首个非空值，更灵活

（coalesce vi. 合并；结合；联合）

**练习**

返回一个有如下两列的查询结果：

- customer (顾客的全名)

- phone (没有的话，显示'Unknown')

```sql
USE sql_store;

SELECT 
    CONCAT(first_name, ' ', last_name) AS customer,
    IFNULL/COALESCE(phone, 'Unknown') AS phone   
FROM customers
```

### IF函数

根据是否满足条件返回不同的值:`IF(条件表达式, 返回值1, 返回值2)` 返回值可以是任何东西，数值 文本 日期时间 空值null 均可

**案例**

将订单表中订单按是否是今年的订单分类为active（活跃）和archived（存档），之前讲过用UNION法，即用两次查询分别得到今年的和今年以前的订单，添加上分类列再用UNION合并，这里直接在SELECT里运用IF函数可以更容易地得到相同的结果

```sql
USE sql_store;

SELECT 
    *,
    IF(YEAR(order_date) = YEAR(NOW()),
       'Active',
       'Archived') AS category
FROM orders
```

**练习**

得到包含如下字段的表：

1. product_id
2. name (产品名称)
3. orders (该产品出现在订单中的次数)
4. frequency (根据是否多于一次而分类为'Once'或'Many times')

```sql
USE sql_store;

SELECT 
    product_id,
    name,
    COUNT(*) AS orders,
    IF(COUNT(*) = 1, 'Once', 'Many times') AS frequency
    /* 因为之后的内连接筛选掉了无订单的商品，
    所以这里不变考虑次数为0的情况 */
FROM products
JOIN order_items USING(product_id)
GROUP BY product_id
```

### CASE运算符

The CASE Operator (5:23)

**小结**

当分类多余两种时，可以用IF嵌套，也可以用CASE语句，后者可读性更好

CASE语句结构：

```sql
CASE 
    WHEN …… THEN ……
    WHEN …… THEN ……
    WHEN …… THEN ……
    ……
    [ELSE ……] （ELSE子句是可选的）
END
```

**案例**

不是将订单分两类，而是分为三类：今年的是 'Active', 去年的是 'Last Year', 比去年更早的是 'Achived'：

```sql
USE sql_store;

SELECT
    order_id,
    CASE
        WHEN YEAR(order_date) = YEAR(NOW()) THEN 'Active'
        WHEN YEAR(order_date) = YEAR(NOW()) - 1 THEN 'Last Year'
        WHEN YEAR(order_date) < YEAR(NOW()) - 1 THEN 'Achived'
        ELSE 'Future'  
    END AS 'category'
FROM orders
```

ELSE 'Future' 是可选的，实验发现若分类不完整，比如只写了今年和去年的两个分类条件，则不在这两个分类的记录的 category 字段会是 null.

**练习**

得到包含如下字段的表：customer, points, category（根据积分 <2k、2k~3k（包含两端）、>3k 分为青铜、白银和黄金用户）

之前也是用过 UNION 法，分别查询增加分类字段再合并，很麻烦。

```sql
USE sql_store;

SELECT
    CONCAT(first_name, ' ', last_name) AS customer,
    points,
    CASE
        WHEN points < 2000 THEN 'Bronze'
        WHEN points BETWEEN 2000 AND 3000 THEN 'Silver'
        WHEN points > 3000 THEN 'Gold'
        -- ELSE null
    END AS category
FROM customers
ORDER BY points DESC
```

其实也可以用IF嵌套，甚至代码还少些，但感觉没有CASE语句结构清晰、可读性好

```sql
SELECT
    CONCAT(first_name, ' ', last_name) AS customer,
    points,
    IF(points < 2000, 'Bronze', 
        IF(points BETWEEN 2000 AND 3000, 'Silver', 
        -- 第二层的条件表达式也可以简化为 <= 3000
            IF(points > 3000, 'Gold', null))) AS category
FROM customers
ORDER BY points DESC
```

## 视图

### 创建视图

Creating Views (5:36)

**小结**

就是创建虚拟表，自动化一些重复性的查询模块，简化各种复杂操作（包括复杂的子查询和连接等）

注意视图虽然可以像一张表一样进行各种操作，但**并没有真正储存数据**，数据仍然储存在原始表中，视图**只是储存起来的模块化的查询结果**，是为了方便和简化后续进一步操作而储存起来的虚拟表。

**案例**

创建 sales_by_client 视图

```sql
USE sql_invoicing;

CREATE VIEW sales_by_client AS
    SELECT 
        client_id,
        name,
        SUM(invoice_total) AS total_sales
    FROM clients c
    JOIN invoices i USING(client_id)
    GROUP BY client_id, name;
    -- 虽然实际上这里加不加上name都一样
```

若要删掉该视图用 `DROP VIEW sales_by_client` 或通过右键菜单

创建视图后可就当作 sql_invoicing 库下一张表一样进行各种操作

```sql
USE sql_invoicing;

SELECT 
    s.name,
    s.total_sales,
    phone
FROM sales_by_client s
JOIN clients c USING(client_id)
WHERE s.total_sales > 500
```

**练习**

创建一个客户差额表视图，可以看到客户的id，名字以及**差额（发票总额-支付总额）**

```sql
USE sql_invoicing;

CREATE VIEW clients_balance AS
    SELECT 
        client_id,
        c.name,
        SUM(invoice_total - payment_total) AS balance
    FROM clients c
    JOIN invoices USING(client_id)
    GROUP BY client_id
```

### 更新或删除视图

Altering or Dropping Views (2:52)

**小结**

修改视图可以先DROP在CREATE（也可以用CREATE OR REPLACE）

视图的查询语句可以在编辑模式下查看和修改，但最好是保存为sql文件并放在源码控制妥善管理

**案例**

想在上一节的顾客差额视图的查询语句最后加上按差额降序排列

法1. 先删除再重建

```sql
USE sql_invoicing;

DROP VIEW IF EXISTS clients_balance;
-- 若不存在这个视图，直接 DROP 会报错，所以要加上 IF EXISTS 先检测有没有这个视图

CREATE VIEW clients_balance AS 
    ……
    ORDER BY balance DESC
```

法2. 用REPLACE关键字，即用 `CREATE OR REPLACE VIEW clients_balance AS`，和上面等效，不过上面那种分成两个语句的方式要用的多一点

```sql
USE sql_invoicing;

CREATE OR REPLACE VIEW clients_balance AS
    ……
    ORDER BY balance DESC
```

### 可更新视图

Updatable Views (5:12)

**小结**

视图作为虚拟表/衍生表，除了可用在查询语句SELECT中，也可以用在增删改（INSERT DELETE UPDATE）语句中，但后者有一定的前提条件。

如果一个视图的原始查询语句中没有如下元素：

1. DISTINCT 去重
2. GROUP BY/HAVING/聚合函数 (后两个通常是伴随着 GROUP BY 分组出现的)
3. UNION 纵向连接

则该视图是可更新视图（Updatable Views），可以增删改，否则只能查。

另外，增（INSERT）还要满足附加条件：视图必须包含底层原表的所有必须字段

总之，一般通过原表修改数据，但当出于安全考虑或其他原因没有某表的直接权限时，可以**通过视图来修改底层数据（？）**，前提是视图是可更新的。

之后会将关于安全和权限的内容

**案例**

创建视图（新虚拟表）invoices_with_balance（带差额的发票记录表）

```sql
USE sql_invoicing;

CREATE OR REPLACE VIEW invoices_with_balance AS
SELECT 
    /* 这里有个小技巧，要插入表中的多列列名时，
    可从左侧栏中连选并拖入相关列 */
    invoice_id, 
    number, 
    client_id, 
    invoice_total, 
    payment_total, 
    invoice_date,
    invoice_total - payment_total AS balance,  -- 新增列
    due_date, 
    payment_date
FROM invoices
WHERE (invoice_total - payment_total) > 0
/* 这里不能用列别名balance，会报错说不存在，
必须用原列名的表达式，这还是执行顺序的问题
之前讲WHERE和HAVING作为事前筛选和事后筛选的区别时提到过 */
```

该视图满足条件，是可更新视图，故可以增删改：

1. 删：

删掉id为1的发票记录

```sql
DELETE FROM invoices_with_balance
WHERE invoice_id = 1
```

1. 改：

将2号发票记录的期限延后两天

```sql
UPDATE invoices_with_balance
SET due_date = DATE_ADD(due_date, INTERVAL 2 DAY)
WHERE invoice_id = 2
```

1. 增：

在视图中用INSERT新增记录的话还有另一个前提，即视图必须包含其底层所有原始表的所有必须字段
例如，若这个 invoices_with_balance 视图里没有 invoice_date 字段（invoices 中的必须字段），那就无法通过该视图向 invoices 表新增记录，因为 invoices 表不会接受 invoice_date 字段为空的记录

### WITH CHECK OPTION 子句

THE WITH CHECK OPTION Clause (2:18)

**小结**

在视图的原始查询语句最后加上 `WITH CHECK OPTION` 可以防止执行那些会让视图中某些行（记录）消失的修改语句。

**案例**

接前面的 invoices_with_balance 视图的例子，该视图与原始的 orders 表相比增加了balance(invouce_total - payment_total) 列，且只显示 balance 大于0的行（记录），若将某记录（如2号订单）的 payment_total 改为和 invouce_total 相等，则 balance 为0，该记录会从视图中消失：

```sql
UPDATE invoices_with_balance
SET payment_total = invoice_total
WHERE invoice_id = 2
```

更新后会发现invoices_with_balance视图里2号订单消失。

但在视图原始查询语句最后加入 `WITH CHECK OPTION` 后，对3号订单执行类似上面的语句后会报错：

```sql
UPDATE invoices_with_balance
SET payment_total = invoice_total
WHERE invoice_id = 3

-- Error Code: 1369. CHECK OPTION failed 'sql_invoicing.invoices_with_balance'
```

### 视图的其他优点

Other Benefits of Views (2:37)

**小结**

三大优点：

简化查询、增加抽象层和减少变化的影响、数据安全性

具体来讲：

1. （首要优点）简化查询 simplify queries
2. 增加抽象层，减少变化的影响 Reduce the impact of changes：视图给表增加了一个抽象层（模块化），这样如果数据库设计改变了（如一个字段从一个表转移到了另一个表），只需修改视图的查询语句使其能保持原有查询结果即可，不需要修改使用这个视图的那几十个查询。相反，如果没有视图这一层的话，所有查询将直接使用指向原表的原始查询语句，这样一旦更改原表设计，就要相应地更改所有的这些查询。
3. 限制对原数据的访问权限 Restrict access to the data：在视图中可以对原表的行和列进行筛选，这样如果你禁止了对原始表的访问权限，用户只能通过视图来修改数据，他们就无法修改视图中未返回的那些字段和记录。但注意这通常并不简单，需要良好的规划，否则最后可能搞得一团乱。

## 存储过程

### 什么是存储过程

What are `Stored Procedures` (2:18)

**小结**

存储过程三大作用：

1. 储存和管理SQL代码 Store and organize SQL
2. 性能优化 Faster execution
3. 数据安全 Data security

**导航**

之前学了增删改查，包括复杂查询以及如何运用视图来简化查询。

假设你要开发一个使用数据库的应用程序，你应该将SQL语句写在哪里呢？

如果将SQL语句内嵌在应用程序的代码里，将使其混乱且难以维护，所以应该将SQL代码和应用程序代码分开，将SQL代码储存在所属的数据库中，具体来说，是放在储存过程（`stored procedure`）和函数中。

储存过程是一个包含SQL代码模块的数据库对象，在应用程序代码中，我们调用储存过程来获取和保存数据（get and save the data）。也就是说，我们使用储存过程来储存和管理SQL代码。

使用储存程序还有另外两个好处。首先，大部分DBMS会对储存过程中的代码进行一些优化，因此有时储存过中的SQL代码执行起来会更快。

此外，就像视图一样，储存过程能加强数据安全。比如，我们可以移除对所有原始表的访问权限，让各种增删改的操作都通过储存过程来完成，然后就可以决定谁可以执行何种储存过程，用以限制用户对我们数据的操作范围，例如，防止特定的用户删除数据。

所以，储存过程很有用，本章将学习如何创建和使用它。

### 创建一个存储过程

Creating a Stored Procedure (5:34)

**小结**

```sql
DELIMITER $$
-- delimiter n. 分隔符

    CREATE PROCEDURE 过程名()  
        BEGIN
            ……;
            ……;
            ……;
        END$$

DELIMITER ;
```

**实例**

创造一个get_clients()过程

```sql
CREATE PROCEDURE get_clients()  
-- 括号内可传入参数，之后会讲
-- 过程名用小写单词和下划线表示，这是约定熟成的做法
    BEGIN
        SELECT * FROM clients;
    END
```

BEGIN 和 END 之间包裹的是此过程（PROCEDURE）的内容（body），内容里可以有多个语句，但每个语句都要以 `;` 结束，包括最后一个。

为了将过程内容内部的语句分隔符与SQL本身执行层面的语句分隔符 `;` 区别开，要先用 DELIMITER(分隔符) 关键字暂时将SQL语句的默认分隔符改为其他符号，一般是改成双美元符号 `$$` ，创建过程结束后再改回来。注意创建过程本身也是一个完整SQL语句，所以别忘了在END后要加一个暂时语句分隔符 `$$`

**注意**

过程内容中所有语句都要以 `;` 结尾并且因此要暂时修改SQL本身的默认分隔符，这些都是MySQL地特性，在SQL Server等就不需要这样

```sql
DELIMITER $$

    CREATE PROCEDURE get_clients()  
        BEGIN
            SELECT * FROM clients;
        END$$

DELIMITER ;
```

调用此程序：

法1. 点击闪电按钮

法2. 用CALL关键字

```sql
USE sql_invoicing;
CALL get_clients()

或

CALL sql_invoicing.get_clients()
```

**注意**

上面讲的是如何在SQL中调用储存过程，但更多的时候其实是要在应用程序代码（可能是 C#、JAVA 或 Python 编写的）中调用。

**练习**

创造一个储存过程 get_invoices_with_balance（取得有差额（差额不为0）的发票记录）

```sql
DROP PROCEDURE get_invoices_with_balance;
-- 注意DROP语句里这个过程没有带括号

DELIMITER $$

    CREATE PROCEDURE get_invoices_with_balance()
        BEGIN
            SELECT *
            FROM invoices_with_balance 
            -- 这是之前创造的视图
            -- 用视图好些，因为有现成的balance列
            WHERE balance > 0;
        END$$

DELIMITER ;

CALL get_invoices_with_balance();
```

### 使用MySQL工作台创建存储过程

Creating Procedures Using MySQLWorkbench (1:21)

也可以用点击的方式创造过程，右键选择 Create Stored Procedure，填空，Apply。这种方式 Workbench 会帮你处理暂时修改分隔符的问题

这种方式一样可以储存SQL文件

事实证明，mosh很喜欢用这种方式，后面基本都是用这种方式创建过程（毕竟不用管改分隔符的问题，能偷懒又何必自找麻烦呢？谁还不是条懒狗呢？）

### 删除存储过程

Dropping Stored Procedures (2:09)

这一节学习删除储存过程，这在发现储存过程里有错误需要重建时很有用。

**实例**

一个创建过程（get_clients）的标准模板

```sql
USE sql_invoicing;

DROP PROCEDURE IF EXISTS get_clients;
-- 注意加上【IF EXISTS】，以免因为此过程不存在而报错

DELIMITER $$

    CREATE PROCEDURE get_clients()
        BEGIN
            SELECT * FROM clients;
        END$$

DELIMITER ;

CALL get_clients();
```

**最佳实践**

同视图一样，最好把删除和创建每一个过程的代码也储存在不同的SQL文件中，并把这样的文件放在 Git 这样的源码控制下，这样就能与其它团队成员共享 Git 储存库。他们就能在自己的机器上重建数据库以及该数据库下的所有的视图和储存过程

如上面那个实例，可储存在 stored_procedures 文件夹（之前已有 views 文件夹）下的 get_clients.sql 文件。当你把所有这些脚本放进源码控制，你能随时回来查看你对数据库对象所做的改动。

### 参数

Parameters (5:26)

**小结**

```sql
CREATE PROCEDURE 过程名
(
    参数1 数据类型,
    参数2 数据类型,
    ……
)
BEGIN
……
END
```

**导航**

学完了如何创建和删除过程，这一节学习如何给过程添加参数

通常我们使用参数来给储存过程传值，但我们也可以用参数获取调用程序的结果值，第二个我们之后再讲

**案例**

创建过程 get_clients_by_state，可返回特定州的顾客

```sql
USE sql_invoicing;

DROP PROCEDURE IF EXISTS get_clients_by_state;

DELIMITER $$

CREATE PROCEDURE get_clients_by_state
(
    state CHAR(2)  -- 参数的数据类型
)
BEGIN
    SELECT * FROM clients c
    WHERE c.state = state;
END$$

DELIMITER ;
```

参数类型一般设定为 VARCHAR，除非能确定参数的字符数

多个参数可用逗号隔开

`WHERE state = state` 是没有意义的，有两种方法可以区分参数和列名：一种是取不一样的参数名如 p_state 或 state_param，第二种是像上面一样给表起别名，然后用带表前缀的列名以同参数名区分开。

```sql
CALL get_clients_by_state('CA')
```

不传入'CA'会报错，因为**MySQL里所有参数都是必须参数**

**练习**

创建过程 get_invoices_by_client，通过 client_id 来获得发票记录

client_id 的数据类型设置可以参考原表中该字段的数据类型

```sql
USE sql_invoicing;

DROP PROCEDURE IF EXISTS get_invoices_by_client ;

DELIMITER $$

CREATE PROCEDURE get_invoices_by_client
(
    client_id INT  -- 为何不写INT(11)？
) 
BEGIN
SELECT * 
FROM invoices i
WHERE i.client_id = client_id;
END$$

DELIMITER ;

CALL get_invoices_by_client(1);
```

Mosh 创建和调用都直接用的点击法

### 带默认值的参数

Parameters with Default Value (8:18)

**小结**

给参数设置默认值，主要是运用条件语句块和替换空值函数

**回顾**

SQL中的条件类语句：

1. 替换空值 `IFNULL(值1，值2)`
2. 条件函数 `IF(条件表达式, 返回值1, 返回值2)`
3. 条件语句块

```sql
IF 条件表达式 THEN
    语句1;
    语句2;
    ……;
[ELSE]（可选）
    语句1;
    语句2;
    ……;
END IF;
-- 别忘了【END IF】
```

**案例1**

把 get_clients_by_state 过程的默认参数设为'CA'，即默认查询加州的客户

```sql
USE sql_invoicing;

DROP PROCEDURE IF EXISTS get_clients_by_state;

DELIMITER $$

CREATE PROCEDURE get_clients_by_state
(
    state CHAR(2)  
)
BEGIN
    IF state IS NULL THEN 
        SET state = 'CA';  
        /* 注意别忽略SET，
        SQL 里单个等号 '=' 是比较操作符而非赋值操作符
        '=' 与 SET 配合才是赋值 */
    END IF;
    SELECT * FROM clients c
    WHERE c.state = state;
END$$

DELIMITER ;
```

调用

```sql
CALL get_clients_by_state(NULL)
```

注意要调用过程并使用其默认值时时要传入参数 `NULL` ，MySQL不允许不传参数。

**案例2**

将 get_clients_by_state 过程设置为默认选取所有顾客

法1. 用IF条件语句块实现

```sql
……
BEGIN
    IF state IS NULL THEN 
        SELECT * FROM clients c;
    ELSE
        SELECT * FROM clients c
        WHERE c.state = state;
    END IF;    
END$$
……
```

法2. 用IFNULL替换空值函数实现

```sql
……
BEGIN
    SELECT * FROM clients c
    WHERE c.state = IFNULL(state, c.state)
END$$
……
```

若参数为NULL，则返回c.state，利用 c.state = c.state 永远成立来返回所有顾客，思路很巧妙。

**练习**

创建一个叫 get_payments 的过程，包含 client_id 和 payment_method_id 两个参数，数据类型分别为 INT(4) 和 TINYINT(1) (1个字节，能存0~255，之后会讲数据类型，好奇可以谷歌 'mysql int size')，默认参数设置为返回所有记录

这是一个为你的工作预热的练习

```sql
USE sql_invoicing;

DROP PROCEDURE IF EXISTS get_payments;

DELIMITER $$

CREATE PROCEDURE get_payments
(
    client_id INT,  -- 不用写成INT(4)
    payment_method_id TINYINT
)
BEGIN
    SELECT * FROM payments p
    WHERE 
        p.client_id = IFNULL(client_id, p.client_id) AND
        p.payment_method = IFNULL(payment_method_id, p.payment_method);
        -- 再次小心这种实际工作中各表相同字段名称不同的情况
END$$

DELIMITER ;
```

调用：

所有支付记录

```sql
CALL get_payments(NULL, NULL)
```

1号顾客的所有记录

```sql
CALL get_payments(1, NULL)
```

3号支付方式的所有记录

```sql
CALL get_payments(NULL, 3)
```

5号顾客用2号支付方式的所有记录

```sql
CALL get_payments(5, 2)
```

**注意**

注意一个区别：

1. Parameter 形参（形式参数）：创建过程中用的占位符，如 client_id、payment_method_id
2. Argument 实参（实际参数）：调用时实际传入的值，如 1、3、5、NULL

### 参数验证

Parameter Validation (6:40)

**小结**

过程除了可以查，也可以增删改，但修改数据前最好先进行参数验证以防止不合理的修改

主要利用条件语句块和 `SIGNAL` `SQLSTATE` `MESSAGE_TEXT` 关键字

具体来说是在过程的内容开头加上这样的语句：

```sql
IF 错误参数条件表达式 THEN
    SIGNAL SQLSTATE '错误类型'
        [SET MESSAGE_TEXT = '关于错误的补充信息']（可选）
```

**案例**

创建一个 make_payment 过程，含 invoice_id, payment_amount, payment_date 三个参数

（Mosh还是喜欢通过右键 Create Stored Procedure 地方式创建，不必考虑暂时改分隔符的问题，更简便）

```sql
CREATE DEFINER=`root`@`localhost` PROCEDURE `make_payment`(
    invoice_id INT,
    payment_amount DECIMAL(9, 2),
    /*
    9是精度， 2是小数位数。
    精度表示值存储的有效位数，
    小数位数表示小数点后可以存储的位数
    见：
    https://dev.mysql.com/doc/refman/8.0/en/fixed-point-types.html 
    */
    payment_date DATE    
)
BEGIN   
    UPDATE invoices i
    SET 
        i.payment_total = payment_amount,
        i.payment_date = payment_date
    WHERE i.invoice_id = invoice_id;
END
```

为了防止传入像 -100 的 payment_total 这样不合理的参数，要在增加一段参数验证语句，利用的是条件语句块加SIGNAL关键字，和其他编程语言中的抛出异常等类似

具体的错误类型可通过谷歌 "sqlstate error" 查阅（推荐使用IBM的那个表），这里是 '22 Data Exception' 大类中的 '22003 A numeric value is out of range.' 类型

注意还添加了 MESSAGE_TEXT 以提供给用户参数错误的更具体信息。现在传入 负数的 payment_amount 就会报错 'Error Code: 1644. Invalid payment amount '

```sql
……
BEGIN
    IF payment_amount <= 0 THEN
        SIGNAL SQLSTATE '22003' 
            SET MESSAGE_TEXT = 'Invalid payment amount';    
    END IF;

    UPDATE invoices i
    ……
END
```

**注意**

过犹不及（"Too much of a good thing is a bad thing"），加入过多的参数验证会让代码过于复杂难以维护，像 payment_amount 非空这样的验证就不需要添加因为 payment_amount 字段本身就不允许空值因此MySQL会自动报错，。

参数验证工作更多的应该在应用程序端接受用户输入数据时就检测和报告，那样更快也更有效。储存过程里的参数验证只是在有人越过应用程序直接访问储存过程时作为最后的防线。这里只应该写那些最关键和必要的参数验证。

### 输出参数

Output Parameters (3:55)

**小结**

输入参数是用来给过程传入值的，我们也可以用输出参数来获取过程的结果值

具体是在参数的前面加上 **OUT 关键字**，然后再 SELECT 后加上 INTO……

调用麻烦，如无需要，不要多此一举

**案例**

创造 get_unpaid_invoices_for_client 过程，获取特定顾客所有未支付过的发票记录（即 payment_total = 0 的发票记录）

```sql
CREATE DEFINER=`root`@`localhost` PROCEDURE `get_unpaid_invoices_for_client`(
        client_id INT
)
BEGIN
    SELECT COUNT(*), SUM(invoice_total)
    FROM invoices i
    WHERE 
        i.client_id = client_id AND
        payment_total = 0;
END
```

调用

```sql
call sql_invoicing.get_unpaid_invoices_for_client(3);
```

得到3号顾客的 COUNT(*) 和 SUM(invoice_total) （未支付过的发票数量和总金额）分别为2和286

我们也可以通过输出参数（变量）来获取这两个结果值，修改过程，添加两个输出参数 invoice_count 和 invoice_total：

```sql
CREATE DEFINER=`root`@`localhost` PROCEDURE `get_unpaid_invoices_for_client`(
        client_id INT,
        OUT invoice_count INT,
        OUT invoice_total DECIMAL(9, 2)
        -- 默认是输入参数，输出参数要加【OUT】前缀
)
BEGIN
    SELECT COUNT(*), SUM(invoice_total)
    INTO invoice_count, invoice_total
    -- SELECT后跟上INTO语句将SELECT选出的值传入输出参数（输出变量）中
    FROM invoices i
    WHERE 
        i.client_id = client_id AND
        payment_total = 0;
END
```

调用：单击闪电按钮调用，只用输入client_id，得到如下语句结果

```sql
set @invoice_count = 0;
set @invoice_total = 0;
call sql_invoicing.get_unpaid_invoices_for_client(3, @invoice_count, @invoice_total);
select @invoice_count, @invoice_total;
```

先定义以@前缀表示用户变量，将初始值设为0。（变量（variable）简单讲就是储存单一值的对象）再调用过程，将过程结果赋值给这两个输出参数，最后再用SELECT查看。

很明显，通过输出参数获取并读取数据有些麻烦，若无充足的原因，不要多此一举。

### 变量

Variables (4:33)

**小结**

两种变量:

1. 用户或会话变量 `SET @变量名 = ……`
2. 本地变量 `DECLARE 变量名 数据类型 [DEFAULT 默认值]`

**用户或会话变量（User or session variable）：**

上节课讲过，用 SET 语句并在变量名前加 @ 前缀来定义，将在整个用户会话期间存续，在会话结束断开MySQL链接时才被清空，这种变量主要在调用带输出的储存过程时，作为输出参数来获取结果值。

**实例**

```sql
set @invoice_count = 0;
set @invoice_total = 0;
call sql_invoicing.get_unpaid_invoices_for_client(3, @invoice_count, @invoice_total);
select @invoice_count, @invoice_total;
```

**本地变量（Local variable）**

在储存过程或函数中通过 DECLARE 声明并使用，在函数或储存过程执行结束时就被清空，常用来执行过程（或函数）中的计算

**案例**

创造一个 get_risk_factor 过程，使用公式 risk_factor = invoices_total / invoices_count * 5

```sql
CREATE DEFINER=`root`@`localhost` PROCEDURE `get_risk_factor`()
BEGIN
    -- 声明三个本地变量，可设默认值
    DECLARE risk_factor DECIMAL(9, 2) DEFAULT 0;
    DECLARE invoices_total DECIMAL(9, 2);
    DECLARE invoices_count INT;

    -- 用SELECT得到需要的值并用INTO传入invoices_total和invoices_count
    SELECT SUM(invoice_total), COUNT(*)
    INTO invoices_total, invoices_count
    FROM invoices;

    -- 用SET语句给risk_factor计算赋值
    SET risk_factor = invoices_total / invoices_count * 5;

    -- 展示最终结果risk_factor
    SELECT risk_factor;        
END
```

### 函数

Functions (6:28)

**小结**

创建函数和创建过程的两点不同

```sql
1. 参数设置和body内容之间，有一段确定返回值类型以及函数属性的语句段
RETURNS INTEGER
DETERMINISTIC
READS SQL DATA
MODIFIES SQL DATA
……
2. 最后是返回（RETURN）值而不是查询（SELECT）值
RETURN IFNULL(risk_factor, 0);
```

删除

```sql
DROP FUNCTION [IF EXISTS] 函数名
```

**导航**

现在已经学了很多内置函数，包括聚合函数和处理数值、文本、日期时间的函数，这一节学习如何创建函数

函数和储存过程的作用非常相似，唯一区别是函数只能返回单一值而不能返回多行多列的结果集，当你只需要返回一个值时就可以创建函数。

**案例**

在上一节的储存过程 get_risk_factor 的基础上，创建函数 get_risk_factor_for_client，计算特定顾客的 risk_factor

还是用右键 Create Function 来简化创建

创建函数的语法和创建过程的语法极其相似，区别只在两点：

1. 参数设置和body内容之间，有一段确定返回值类型以及函数属性的语句段
2. 最后是返回（RETURN）值而不是查询（SELECT）值

另外，关于函数属性的说明：

1. DETERMINISTIC 决定性的：唯一输入决定唯一输出，和数据的改动更新无关，比如税收是订单总额的10%，则以订单总额为输入税收为输出的函数就是决定性的（？），但这里每个顾客的 risk_factor 会随着其发票记录的增加更新而改变，所以不是DETERMINISTIC的
2. READS SQL DATA：需要用到 SELECT 语句进行数据读取的函数，几乎所有函数都满足
3. MODIFIES SQL DATA：函数中有 增删改 或者说有 INSERT DELETE UPDATE 语句，这个例子不满足

```sql
CREATE DEFINER=`root`@`localhost` FUNCTION `get_risk_factor_for_client`
(
    client_id INT
) 
RETURNS INTEGER
-- DETERMINISTIC
READS SQL DATA
-- MODIFIES SQL DATA
BEGIN
    DECLARE risk_factor DECIMAL(9, 2) DEFAULT 0;
    DECLARE invoices_total DECIMAL(9, 2);
    DECLARE invoices_count INT;

    SELECT SUM(invoice_total), COUNT(*)
    INTO invoices_total, invoices_count
    FROM invoices i
    WHERE i.client_id = client_id;
    -- 注意不再是整体risk_factor而是特定顾客的risk_factor

    SET risk_factor = invoices_total / invoices_count * 5;
    RETURN IFNULL(risk_factor, 0);       
END
```

有些顾客没有发票记录，NULL乘除结果还是NULL，所以最后用 IFNULL 函数将这些人的 risk_factor 替换为 0

调用案例：

```sql
SELECT 
    client_id,
    name,
    get_risk_factor_for_client(client_id) AS risk_factor
    -- 函数当然是可以处理整列的，我第一时间竟只想到传入具体值
    -- 不过这里更像是一行一行的处理，所以应该每次也是传入1个client_id值
FROM clients
```

删除，还是用DROP

```sql
DROP FUNCTION [IF EXISTS] get_risk_factor_for_client
```

**注意**

和视图和过程一样，也最好存入SQL文件并加入源码控制，老生常谈了。

### 其他约定

Other Conventions (1:51)

有各种各样的命名习惯（包括对函数过程的命名习惯以及对更改分隔符的习惯），没有明显的好坏之分，重要的是在一个项目或团队中保持恒定不变，学会入乡随俗

## 触发器

**触发器是一种特殊的`存储过程`。但触发器没有输入和输出参数，因而不能被显示调用。它作为语句的执行结果自动引发，而存储过程则是通过存储过程名称被直接调用。**

触发器的功能

1. 强化约束

- 触发器能够实现比 CHECK 语句更为复杂的约束。
- 触发器可以很方便地引用其他表的列，去进行逻辑上的检查。
- 触发器是在 CHECK 之后执行的
- 触发器可以插入、删除、更新多行

2. 跟踪变化

- 触发器可以检测数据库内的操作，从而禁止数据库中未经许可的更新和变化，以确保输入表中的数据的有效性。
- 例如：库存系统中，触发器检测到当实际库存下降到需要再进货的临界值时，就给管理员相应提示信息或自动生成给供应商的订单。

3. 级联运行

- 触发器可以检测数据库内的操作，并自动地级联影响整个数据库的不同表的各项内容。
- 举例：删除主键的数据，外键表的数据可以修改为NULL或删除该数据。

4. 调用存储过程

- 为了响应数据库更新，可以调用一个或多个触发器。

**小结**

触发器是在插入、更新或删除语句前后自动执行的一段SQL代码（A block of SQL code that automatically gets executed before or after an insert, update or delete statement）通常我们使用触发器来保持数据的一致性

创建触发器的语法要点：`命名三要素，触发条件语句和触发频率语句，主体中 OLD/NEW 的使用`

**案例**

在 sql_invoicing 库中，发票表中同一个发票记录可以对应付款表中的多次付款记录，**发票表中的付款总额应该等于这张发票所有付款记录之和**，为了保持数据一致性，可以通过触发器让每一次付款表中新增付款记录时，发票表中相应发票的付款总额（payement_total）自动增加相应数额

语法上，和创建储存过程等类似，要暂时更改分隔符，用 CREATE 关键字，用 BEGIN 和 END 包裹的主体

几个关键点：

1. 命名习惯（三要素）：触发表_before/after(SQL语句执行之前或之后触发)_触发的SQL语句类型

2. 触发条件语句：`BEFORE/AFTER INSERT/UPDATE/DELETE ON 触发表`

3. 触发频率语句：这里 `FOR EACH ROW` 表明每一个受影响的行都会启动一次触发器。其它有的DBMS还支持表级别的触发器，即不管插入一行还是五行都只启动一次触发器，到Mosh录制为止MySQL还不支持这样的功能

4. 主体：主体里可以对各种表的数据进行修改以保持数据一致性，但注意唯一不能修改的表是触发表，否则会引发无限循环（“触发器自燃”），主体中最关键的是**使用 NEW/OLD 关键字来指代受影响的新/旧行**（若INSERT用NEW，若DELETE用OLD，若UPDATE似乎两个都可以用？）并可跟 '点+字段' 以引用这些行的相应属性

```sql
DELIMITER 
$$

CREATE TRIGGER payments_after_insert
    AFTER INSERT ON payments
    FOR EACH ROW
BEGIN
    UPDATE invoices
    SET payment_total = payment_total + NEW.amount
    WHERE invoice_id = NEW.invoice_id;
END

$$
DELIMITER ;
```

测试：往 payments 里新增付款记录，发现 invoices 表对应发票的付款总额确实相应更新

```sql
INSERT INTO payments
VALUES (DEFAULT, 5, 3, '2019-01-01', 10, 1)
```

**练习**

创建一个和刚刚的触发器作用刚好相反的触发器，每当有付款记录被删除时，自动减少发票表中对应发票的付款总额

```sql
DELIMITER $$

CREATE TRIGGER payments_after_delete
    AFTER DELETE ON payments
    FOR EACH ROW
BEGIN
    UPDATE invoices
    SET payment_total = payment_total - OLD.amount
    WHERE invoice_id = OLD.invoice_id;
END$$

DELIMITER ;
```

测试：删掉付款表里刚刚的那个给3号发票支付10美元的付款记录，则果然发票表里3号发票的付款总额相应减少10美元.

```sql
DELETE FROM payments
WHERE payment_id = 9
```

### 查看触发器

Viewing Triggers (1:20)

用以下命令来查看已存在的触发器及其各要素

```sql
SHOW TRIGGERS
```

如果之前创建时遵行了三要素命名习惯，这里也可以用 LIKE 关键字来筛选特定表的触发器

```sql
SHOW TRIGGERS LIKE 'payments%'
```

删除触发器

Dropping Triggers (0:52)

和删除储存过程的语句一样

```sql
DROP TRIGGER [IF EXISTS] payments_after_insert
-- IF EXISTS 是可选的，但一般最好加上
```

**最佳实践**

最好将**删除和创建**数据库/视图/储存过程/触发器的语句放在同一个脚本中（即将删除语句放在创建语句前，DROP IF EXISTS + CREATE，用于创建或更新数据库/视图/储存过程/触发器，等效于 CREATE OR REPLACE）并将脚本录入源码库中，这样不仅团队都可以创建相同的数据库，还都能查看数据库的所有修改历史

```sql
DELIMITER $$

DROP TRIGGER IF EXISTS payments_after_insert;
/*
实验了一下好像这里用$$也可以,
但为什么可以用;啊？
*/

CREATE TRIGGER payments_after_insert
    AFTER INSERT ON payments
    FOR EACH ROW
BEGIN
    UPDATE invoices
    SET payment_total = payment_total + NEW.amount
    WHERE invoice_id = NEW.invoice_id;
END$$

DELIMITER ;
```

### 使用触发器进行审核

Using Triggers for Auditing (4:52)

**导航**

之前已经学习了如何用触发器来保持数据一致性，触发器的另一个常见用途是为了审核的目的将修改数据的操作记录在日志里。

**小结**

建立一个**审核表（日志表）以记录谁在什么时间做了什么修改**，实现方法就是在**触发器里加上创建日志记录的语句**，日志记录应包含**修改内容信息和操作信息**两部分。

**案例**

用 create-payments-table.sql 创建 payments_audit 表，记录所有对 payements 表的增删操作，注意该表包含 client_id, date, amount 字段来记录修改的内容信息（方便之后恢复操作，如果需要的话）和 action_type, action_date 字段来记录操作信息。注意这是个简化了的 audit 表以方便理解。

具体实现方法是，重建在 payments 表里的的增删触发器 payments_after_insert 和 payments_after_delete，在触发器里加上往 payments_audit 表里添加日志记录的语句

具体而言：

往 payments_after_insert 的主体里加上这样的语句：

```sql
INSERT INTO payments_audit
VALUES (NEW.client_id, NEW.date, NEW.amount, 'insert', NOW());
```

往 payments_after_delete 的主体里加上这样的语句：

```sql
INSERT INTO payments_audit
VALUES (OLD.client_id, OLD.date, OLD.amount, 'delete', NOW());
```

测试：

```sql
-- 增：
INSERT INTO payments
VALUES (DEFAULT, 5, 3, '2019-01-01', 10, 1);

-- 删：
DELETE FROM payments
WHERE payment_id = 10
```

发现 payments_audit 表里果然多了两条记录以记录这两次增和删的操作

**注意**

实际运用中不会为数据库中的每张表建立一个审核表，相反，会有一个整体架构，通过一个总审核表来记录，这在之后设计数据库中会讲到。

**导航**

下节课学习事件

------

### 事件

Events (4:33)

事件是一段根据计划执行的代码，可以执行一次，或者按某种规律执行，比如每天早上10点或每月一次

通过事件我们可以自动化数据库维护任务，比如删除过期数据、将数据从一张表复制到存档表 或者 汇总数据生成报告，所以事件十分有用。

首先，需要打开MySQL事件调度器（event_scheduler），这是一个时刻寻找需要执行的事件的后台程序

查看MySQL所有系统变量：

```sql
SHOW VARIABLES;
SHOW VARIABLES LIKE 'event%';
-- 使用 LIKE 操作符查找以event开头的系统变量
-- 通常为了节约系统资源而默认关闭
```

用SET语句开启或关闭,不想用事件时可关闭以节省资源，这样就不会有一个不停寻找需要执行的事件的后台程序

```sql
SET GLOBAL event_scheduler = ON/OFF
```

**案例**

创建这样一个 yearly_delete_stale_audit_row 事件，每年删除过期的（超过一年的）日志记录（stale adj. 陈腐的；不新鲜的）

```sql
DELIMITER $$

CREATE EVENT yearly_delete_stale_audit_row

-- 设定事件的执行计划：
ON SCHEDULE  
    EVERY 1 YEAR [STARTS '2019-01-01'] [ENDS '2029-01-01']    

-- 主体部分：（注意 DO 关键字）
DO BEGIN
    DELETE FROM payments_audit
    WHERE action_date < NOW() - INTERVAL 1 YEAR;
END$$

DELIMITER ;
```

关键点：

1. 命名：用时间间隔（频率）开头，可以方便之后分类检索，时间间隔（频率）包括 【once】/hourly/daily/monthly/yearly 等等

2. 执行计划：

- 规律性周期性执行用 EVERY 关键字，可以是 EVERY 1 HOUR / EVERY 2 DAY 等等
- 若只执行一次就用 AT 关键字，如：`AT '2019-05-01'`
- 开始 STARTS 和结束 ENDS 时间都是可选的

另外：

`NOW() - INTERVAL 1 YEAR` 等效于 `DATE_ADD(NOW(), INTERVAL -1 YEAR)` 或 `DATE_SUB(NOW(), INTERVAL 1 YEAR)`，但感觉不用DATEADD/DATESUB函数，直接相加减（但INTERVAL关键字还是要用）还简单直白点

**小结**

查看和开启/关闭事件调度器（event_scheduler）：

```sql
SHOW VARIABLES LIKE 'event%';
SET GLOBAL event_scheduler = ON/OFF
```

创建事件：

```sql
……
CREATE EVENT 以频率打头的命名
ON SCHEDULE
    EVERY 时间间隔 / AT 特定时间 [STARTS 开始时间][ENDS 结束时间]
DO BEGIN
……
END$$
……
```

------

### 查看、删除和更改事件

Viewing, Dropping and Altering Events (2:04)

**导航**

上节课讲的是创建事件，即“增”，这节课讲如何对事件进行“查、删、改”，说来说去其实任何对象都是这四种操作

查（SHOW）和删（DROP）和之前的类似：

```sql
SHOW EVENTS 
-- 可看到各个数据库的事件
SHOW EVENTS [LIKE 'yearly%'];  
-- 之前命名以时间间隔开头这里才能这样筛选
DROP EVENT IF EXISTS yearly_delete_stale_audit_row;
```

“改” 要特殊一些，这里首次用到 ALTER 关键字，而且有两种用法：

1. 如果要修改事件内容（包括执行计划和主体内容），直接把 ALTER 当 CREATE 用（或者说更像是REPLACE）直接重建语句
2. 暂时地启用或停用事件（用 DISABLE 和 ENABLE 关键字）

```sql
ALTER EVENT yearly_delete_stale_audit_row DISABLE/ENABLE
```

------

## 事务

事务（trasaction）是**完成一个完整事件的一系列SQL语句**。这一组SQL语句是**一条船上的蚂蚱**，要不然都成功，要不然都失败，如果一部分执行成功一部分执行失败那成功的那一部分就会复原（revert）以保持数据的一致性。

**ACID 特性**

事务有四大特性，总结为 ACID（刚好是英文单词“酸的”）：

1. Atomicity 原子性，即整体性，不可拆分行（unbreakable），所有语句必须都执行成功事务才算完成，否则只要有语句执行失败，已执行的语句也会被复原
2. Consistency 一致性，指的是通过事务我们的数据库将永远保持一致性状态，比如不会出现没有完整订单项目的订单
3. Isolation 隔离性，指事务间是相互隔离互不影响的，尤其是需要访问相同数据时。具体而言，如果多个事务要修改相同数据，该数据会被锁定，每次只能被一个事务有权修改，其它事务必须等这个事务执行结束后才能进行
4. Durability 持久性，指的是一旦事务执行完毕，这种修改就是永久的，任何停电或系统死机都不会影响这种数据修改

### 创建事务

Creating Transactions (5:11)

**准备**

先用 create-databases.sql 恢复一下数据库

**案例**

创建一个事务来储存订单及其订单项目（为了简化，这个订单只有一个项目）

用 START TRANSACTION 来开始创建事务，用 COMMIT 来关闭事务

```sql
USE sql_store;

START TRANSACTION;

INSERT INTO orders (customer_id, order_date, status)
VALUES (1, '2019-01-01', 1);
-- 只需明确声明并插入这三个非自增必须（不可为空）字段

INSERT INTO order_items 
-- 所有字段都是必须的，就不必申明了
VALUES (last_insert_id(), 1, 2, 3);

COMMIT;
```

执行，会看到最新的订单和订单项目记录

当 MySQL 看到上面这样的事务语句组，会把所有这些更改写入数据库，如果有任何一个更改失败，会自动撤销之前的修改，这种情况被称为事务被退回（回滚）（is rolled back）

为了模拟退回的情况，可以用 Ctrl + Enter 逐条执行语句，执行一半，即录入了订单但还没录入订单项目时断开连接（模拟客户端或服务器崩溃或断网之类的情况），重连后会发现订单和订单项目都没有录入

**手动退回**

多数时候是用上面的 `START TRANSACTION + COMMIT` 来创建事务，但当我们想先进行一下事务里语句的测试/错误检查并因此想在执行结束后手动退回时，可以将最后的 `COMMIT;` 换成 `ROLLBACK;`，这会退回事务并撤销所有的更改

**autocommit**

？ 我们执行的每一个语句（可以是增删查改 SELECT、INSERT、UPDATE 或 DELETE 语句），就算没有 START TRANSACTION + COMMIT，也都会被 MySQL 包装（wrap）成事务并在没有错误的前提下自动提交，这个过程由一个叫做 autocommit 的系统变量控制，默认开启

因为有 autocommit 的存在，当事务只有一个语句时，用不用 START TRANSACTION + COMMIT 都一样，但要将多个语句作为一个事务时就必须要加 START TRANSACTION + COMMIT 来手动包装了

```sql
SHOW VARIABLES LIKE 'autocommit';
```

**小结**

```sql
START TRANSACTION;

……;

COMMIT / ROLLBACK;

SHOW VARIABLES LIKE 'autocommit';
```

### 并发和锁定

Concurrency and Locking (4:07)

**并发**

之前都只有一个用户访问数据，现实中常出现多个用户访问相同数据的情况，这被称为“并发”（concurrency），当一个用户企图修改另一个用户正在检索或修改的数据时，并发会成为一个问题

**导航**

本节介绍默认情况下MySQL是如何处理并发问题的，接下来几节课将介绍如何最小化并发问题

**案例**

假设要通过如下事务语句给1号顾客的积分增加10分

```sql
USE sql_store;
START TRANSACTION;
UPDATE customers
SET points = points + 10
WHERE customer_id = 1;
COMMIT;
```

现在有两个会话（注意是两个链接（connection），而不是同一个会话下的两个SQL标签，这两个链接相当于是在模拟两个用户）都要执行这段语句，用 Ctrl+Enter 逐句执行， 当第一个执行到UPDATE 而还没有 COMMIT 提交时，转到第二个会话，执行到UPDATE语句时会出现旋转指针表示在等待执行（若等的时间太久会超时而放弃执行），这时跳回第一个对话 COMMIT 提交，第二个会话的 UDDATE 才不再转圈而得以执行，最后将第二段对话的事务也COMMIT提交，此时刷新顾客表会发现1号顾客的积分多了20分

**上锁**

所以，可以看到，当一个事务修改一行或多行时，会给这些行上锁，这些锁会阻止其他事务修改这些行，直到前一个事务完成（不管是提交还是退回）为止，由于上述MySQL默认状态下的锁定行为，多数时候不需要担心并发问题，但在一些特殊情况下，默认行为不足以满足你应用里的特定场景，这时你可以修改默认行为，这正是我们接下要学习的

**导航**

我们接下来会学习常见并发问题以及如何解决他们

### 并发问题

Concurrency Problems (7:25)

现在已经知道什么是并发了，我们来看看它带来的常见问题：

**1. Lost Updates 丢失更新**

例如，当事务A要更新john的所在州而事务B要更新john的积分时，若两个事务都读取了john的记录，在A跟新了州且尚未提交时，B更新了积分，那后执行的B的更新会覆盖先执行的A的更新，州的更新将会丢失。

解决方法就是前面说的锁定机制，锁定会防止多个事务同时更新同一个数据，必须一个完成的再执行另一个

**2. Dirty Reads 脏读**

例如，事务A将某顾客的积分从10分增加为20分，但在提交前就被事务B读取了，事务B按照这个尚未提交的顾客积分确定了折扣数额，可之后事务A被退回了，所以该顾客的积分其实仍然是10分，因此事务B等于是读取了一个数据库中从未提交的数据并以此做决定，这被称作为脏读

解决办法是设定事务的隔离等级，例如让一个事务无法看见其它事务尚未提交的更新数据，这个下节课会学习。标准SQL有四个隔离等级，比如，我们可以把事务B设为 READ COMMITED 等级，它将只能读取提交后的数据

积分提交完之后，B事务依此做决定，如果之后积分再修改，这就不是我们考虑的问题了，我们只需要保证B事务读取的是提交后的数据就行了

**3. Non-repeating Reads 不可重复读取 （或 Inconsistent Read 不一致读取）**

上面的隔离能保证只读取提交过的数据，但有时会发生一个事务读取同一个数据两次但两次结果不一致的情况

例如，事务A的语句里需要读取两次某顾客的积分数据，读取第一次时是10分，此时事务B把该积分更新为0分并提交，然后事务A第二次读取积分为0分，这就发生了不可重复读取 或 不一致读取

一种说法是，我们应该总是依照最新的数据做决定，所以这不是个问题。在商务场景中，我们一般不用担心这个问题

另一种说法是，我们应该保持数据一致性，以事务A在开始执行时的数据初始状态为依据来做决定，如果这是我们想要的话，就要增加事务A的隔离等级，让它在执行过程中看不见其它事务的数据更改（即便是提交过的），SQL有个标准隔离等级叫 Repeatable Read 可重复读取，可以保证读取的数据是可重复和一致的，无论过程中其它事务对数据做了何种更改，读取到的都是数据的**初始状态**

**4. Phantom Reads 幻读 （n. 幽灵；幻影，幻觉）**

最后一个并发问题是幻读

例如，事务A要查询所有积分超过10的顾客并向他们发送带折扣码的E-mail，查询后执行结束前，事务B更新了（可能是增删改）数据，然后多了一个满足条件的顾客，事务A执行结束后就会有这么一个满足条件的顾客没有收到折扣码，这就是幻读，Phantom是幽灵的意思，这种突然出现的数据就像幽灵一样，我们在查询中错过了它因为它是在我们查询语句后才更新的

解决办法取决于想解决的商业问题具体是什么样的以及把这个顾客包括进事务中有多重要

我们总可以再次执行事务A来让这顾客包含进去

但如果确保我们总是包含了最新的所有满足条件的顾客是至关重要的，我们就要保证查询过程中没有任何其他可能影响查询结果的事务在进行，为此，我们建立另一个隔离等级叫 Serializable 序列化，它让事务能够知晓是否有其它事务正在进行可能影响查询结果的数据更改，并会等待这些事务执行完毕后再执行，这是最高的隔离等级，为我们提供了最高的操作确定性。但 Serializable 序列化 等级是有代价的，当用户和并发增加时，等待的时间会变长，系统会变慢，所以这个隔离等级会影响性能和可扩展性，出于这个原因，我们只要在避免幻读确实必要的情形下才使用这个隔离等级

**导航**

这里只是先总体介绍，之后的课程会详细讲解每个并发问题以及如何用相应的隔离等级来解决它们

### 事务隔离级别

Transaction Isolation Levels (5:42)

**总结：并发问题与隔离等级**

> 其实我觉得这个表里其它都好理解，最需要记忆的是解决 丢失更新 问题

![img](高级查询.assets/v2-a11c4204c4167ec84b1b832ef4b8a0c1_1440w.webp)

Transaction Isolation Levels

**四个并发问题：**

1. Lost Updates 丢失更新：两个事务更新同一行，最后提交的事务将覆盖先前所做的更改
2. Dirty Reads 脏读：读取了未提交的数据
3. Non-repeating Reads 不可重复读取 （或 Inconsistent Read 不一致读取）：在事务中读取了相同的数据两次，但得到了不同的结果
4. Phantom Reads 幻读：在查询中缺失了一行或多行，因为另一个事务正在修改数据而我们没有意识到事务的修改，我们就像遇见了鬼或者幽灵

**为了解决这些问题，我们有四个标准的事务隔离等级：**

1. Read Uncommitted 读取未提交：无法解决任何一个问题，因为事务间并没有任何隔离，他们甚至可以读取彼此未提交的更改
2. Read Committed 读取已提交：给予事务一定的隔离，这样我们只能读取已提交的数据，这防止了Dirty Reads 脏读，但在这个级别下，事务仍可能读取同个内容两次而得到不同的结果，因为另一个事务可能在两次读取之间更新并提交了数据，也就是它不能防止Non-repeating Reads 不可重复读取 （或 Inconsistent Read 不一致读取）
3. Repeatable Read 可重复读取：在这一级别下，我们可以确信不同的读取会返回相同的结果，即便数据在这期间被更改和提交
4. Serializable 序列化：可以防止以上所有问题，这一级别还能防止幻读，如果数据在我们执行过程中改变了，我们的事务会等待以获取最新的数据，但这很明显会给服务器增加负担，因为管理等待的事务需要消耗额外的储存和CPU资源

**并发问题 VS 性能和可扩展性：**

更低的隔离级别更容易并发，会有更多用户能在相同时间接触到相同数据，但也因此会有更多的并发问题，另一方面因为用以隔离的锁定更少，性能会更高

相反，更高的隔离等级限制了并发并减少了并发问题，但代价是性能和可扩展性的降低，因为我们需要更多的锁定和资源

MySQL的默认等级是 Repeatable Read 可重复读取，它可以防止除幻读外的所有并发问题并且比序列化更快，多数情况下应该保持这个默认等级。

如果对于某个特定的事务，防止幻读至关重要，可以改为 Serializable 序列化

对于某些对数据一致性要求不高的批量报告或者对于数据很少更新的情况，同时又想获得更好性能时，可考虑前两种等级

总的来说，一般保持默认隔离等级，只在特别需要时才做改变

**设定隔离等级的方法**

读取隔离等级

```sql
SHOW VARIABLES LIKE 'transaction_isolation';
--？为什么我的客户端没有这个变量？
```

显示默认隔离等级为 'REPEATABLE READ'

改变隔离等级：

```sql
SET [SESSION]/[GLOBAL] TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

默认设定的是下一次事务的隔离等级，加上 SESSION 就是设置本次会话（链接）之后所有事务的隔离等级，加上 GLOBAL 就是设置之后所有对话的所有事务的隔离等级

如果你是个应用开发人员，你的应用内有一个功能或函数可以链接数据库来执行某一事务（可能是利用对象关系映射或是直接连接MySQL），你就可以连接数据库，用 SESSION 关键词设置本次链接的事务的隔离等级，然后执行事务，最后断开连接，这样数据库的其它事务就不会受影响

**导航**

接下来讲逐一讲解各个隔离级别

### 读取未提交隔离级别

READ UNCOMMITTED Isolation Level (3:26)

**小结**

主要通过模拟脏读来表明 Read Uncommitted（读取未提交）是最低的隔离等级并会遇到所有并发问题

**案例**

建立链接1和链接2，模拟用户1和用户2，分别执行如下语句：

链接1：

查询顾客1的积分，用于之后的商业决策（如确定折扣等级）

注意里面的 SELECT 查询语句虽然没被 START TRANSACTION + COMMIT 包裹，但由于 autucommit，MySQL会把执行的每一条没错误的语句包装在事务中并自动提交，所以这个查询语句也是一个事务，隔离等级为上一句设定的 READ UNCOMMITTED（读取未提交）

```sql
USE sql_store;
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
SELECT points
FROM customers
WHERE customer_id = 1
```

链接2：

建立事务，将顾客1的积分（由原本的2293）改为20

```sql
USE sql_store;
START TRANSACTION;
UPDATE customers
SET points = 20
WHERE customer_id = 1;
ROLLBACK;
```

模拟过程:

链接1将下一次事务（感觉是针对本对话的下一次事务）的隔离等级设定为 READ UNCOMMITTED 读取未提交

→ 链接2执行了更新但尚未提交

→ 链接1执行了查询，得到结果为尚未提交的数据，即查询结果为20分而非原本的2293分

→ 链接2的更新事务被中断退回（可能是手动退回也可能是因故障中断）

这样我们的对话1就使用了一个数据库中从未存在过的值，这就是脏读问题，总之，READ UNCOMMITTED 读取未提交 是最低的隔离等级，在这一级别我们会遇到所有的并发问题

### 读取已提交隔离级别

READ COMMITTED Isolation Level (3:01)

**小结**

Read Committed 读取已提交 等级只会读取别人已提交的数据，所以不会发生脏读，但因为能够读取到执行过程中别人已提交的更改，所以还是会发生不可重复读取（不一致读取）的问题

另外（Mosh没讲，自己从第5节的表里想到的），因为这一等级对于在执行更改型的事务语句时不会锁定正在操作的行，所以同时执行的更改型事务可能发生后面的覆盖前面的情况，所以也不能避免更新丢失的问题

**案例1：不会发生脏读**

就是把上一节链接1的设置隔离级别语句改为 READ COMMITTED 读取已提交 等级，就会发现链接1不会读取到链接2未提交的更改，只有当改为20分的事务提交以后才能被链接1的查询语句读取到

**案例2：可能会发生不可重复读取（不一致读取）**

虽然不会存在脏读，但会出现其他的并发问题，如 Non-repeating Reads 不可重复读取，即在一个事务中你会两次读取相同的内容，但每次都得到不同的值

为模拟该问题，将顾客1的分数还原为2293，将上面的连接1里的语句变为两次相同的查询（查询1号顾客的积分），连接2里的UPDATE语句不变，还是将1号顾客的积分（由原本的2293）更改为20

```sql
USE sql_store;
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
START TRANSACTION;
SELECT points FROM customers WHERE customer_id = 1;
SELECT points FROM customers WHERE customer_id = 1;
COMMIT;
```

注意虽然案例1里已经执行过一次 `SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;` 但这里还是要再执行一次，因为该语句是设定（本对话内）【下一次（next）】事务的隔离等级，如果这里不执行，事务就会恢复为MySQL默认隔离等级，即 Repeatable Read 可重复读取

还有因为这里事务里有两个语句，所以必须手动添加 START TRANSACTION + COMMIT 包装成一个事务，否则autocommit会把它们分别包装形成 两个事务

模拟过程：

再次设定隔离等级为 READ COMMITTED，启动事务，执行第一次查询，得到分数为2293

→ 执行链接2的 UPDATE 语句并提交

→ 再执行链接1的第二次查询，得到分数为20，同一个事务里的两次查询得到不同的结果，发生了 Non-repeating Reads 不可重复读取 （或 Inconsistent Read 不一致读取）

**导航**

为了解决 Non-repeating Reads 不可重复读取 的问题，我们需要提高隔离等级，这正是接下来要学习的

### 重复读取隔离级别

REPEATABLE READ Isolation Level (3:29)

**小结**

在这一默认级别上，不仅只会读取已提交的更改，而且同一个事物内读取会始终保持一致性，但因为可能会忽视正在进行但未提交的可能影响查询结果的更改而漏掉一些结果，即发生幻读

Mosh没讲但从第3节以及第5节的表格可以看得出来，这个默认级别还能避免更新丢失问题

之前说了MySQL默认等级正是 REPEATABLE READ（重复读取）而且MySQL默认会在执行事务内的增删改语句时锁定相关行，所以可以判断 **REPEATABLE READ（重复读取）正是通过执行修改语句时锁定相关行来避免更新丢失问题的**（不过执行查询语句时应该不是通过锁定而只是是通过记忆原始值来保证一致读取的，因为查询语句中途并不会阻止别人更改相关行）

**案例1：不会发生不可重复读取（不一致读取）**

（注意，先要将上一节最后的事务COMMIT提交了，才能执行新的，设定下一次事务隔离等级的语句）

此案例和上一个案例完全一样，只是把隔离等级的设定语句改为了 REPEATABLE READ 可重复读取，然后发现两次查询中途别人把积分从2293改为20不会影响两次查询的结果，都是初始状态的20分，不会发生不可重复读取（不一致读取）

**案例2：可能发生幻读**

但这一级别还是会发生幻读的问题，一个模拟情形如下：

用户1：查询在 'VA' 州的顾客

```sql
USE sql_store;
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
START TRANSACTION;
SELECT * FROM customers WHERE state = 'VA';
SELECT points FROM customers WHERE customer_id = 1;
COMMIT;
```

用户2：将1号顾客所在州更改为 'VA'

```sql
USE sql_store;
START TRANSACTION;
UPDATE customers
SET state = 'VA'
WHERE customer_id = 1;
COMMIT;
```

假设customer表中原原本只有2号顾客在维州（'VA'）

→ 用户2现在正要将1号顾客也改为VA州，已执行UPDATE语句但还没有提交，所以这个更改技术上讲还在内存里
→ 此时用户1查询身处VA州的顾客，只会查到2号顾客
→ 用户2提交更改
→ 若1号用户未提交，再执行一次事务中的查询语句会还是只有2号顾客，因为在 REPEATABLE READ 可重复读取 隔离级别，我们的读取会保持一致性
→ 若1号用户提交后再执行一次查询，会得到1号和2号两个顾客的结果，我们之前的查询遗漏了2号顾客，这被称作为幻读

简单讲就是在这一等级下1号用户的事务只顾读取当前已提交的数据，不能察觉现在正在进行但还未提交的可能对查询结果造成影响的更改，导致遗漏这些新的“幽灵”结果

下节课讲如何用序列化隔离级别来解决幻读问题

### 序列化隔离级别

SERIALIZABLE Isolation Level (2:18)

**案例**

和上面那个案例一摸一样，只是把用户1事务的隔离等级设置为 SERIALIZABLE 序列化，模拟场景如下：

→ 用户2现在正要将1号顾客也改为VA州，已执行UPDATE语句但还没有提交，所以这个更改技术上讲还在内存里
→ 此时用户1查询身处VA州的顾客，会**察觉**到用户2的事务正在进行，因而会出现旋转指针等待用户2的完成
→ 用户2提交更改
→ 用户1的查询语句执行并返回最新结果：顾客1和顾客2

**区别**

我感觉 REPEATABLE READ（重复读取）和 SERIALIZABLE（序列化）的区别在于，前者是修改时自己优先（锁定相关行）查询时自以为是（记忆相关行），后者修改时可能是一样的（不确定），但查询时若**察觉**到别人在进行的更改可能对自己的查询结果有影响会让别人优先——“你要改你先来，你改完了我再查”

**小结**

SERIALIZABLE（序列化）是最高隔离等级，它等于是把系统变成了一个单用户系统，事务只能一个接一个依次进行，所以所有并发问题（更新丢失、脏读、不一致读取、幻读）都从从根本上解决了，但用户和事务越多等待时间就会越漫长，所以，只对那些避免幻读至关重要的事务使用这个隔离等级。默认的可重复读取等级对大多数场景都有效，最好保持这个默认等级，除非你知道你在干什么（Stick to that, unless you know what you are doing）

### 死锁

Deadlocks (6:11)

**小结**

不管什么隔离等级，**事务里的增删改语句都会锁定相关行**（我怎么觉得前两个等级不会呢？不然也不会有更新丢失的问题了……），如果两个同时在进行的事务分别锁定了对方下一步要使用的行，就会发生死锁，死锁不能完全避免但有一些方法能减少其发生的可能性

**案例**

用户1：将1号顾客的州改为'VA'，再将1号订单的状态改为1

```sql
USE sql_store;
START TRANSACTION;
UPDATE customers SET state = 'VA' WHERE customer_id = 1;
UPDATE orders SET status = 1 WHERE order_id = 1;
COMMIT;
```

用户2：和用户1完全相同的两次更改，只是顺序颠倒

```sql
USE sql_store;
START TRANSACTION;
UPDATE orders SET status = 1 WHERE order_id = 1;
UPDATE customers SET state = 'VA' WHERE customer_id = 1;
COMMIT;
```

模拟场景：

用户1和2均执行完各自的第一个更改
→ 用户2执行第二个更改，出现旋转指针
→ 用户1执行第二个更改，出现死锁，报错：Error Code: 1213. Deadlock found ……

**缓解方法**

死锁如果只是偶尔发生一般不是什么问题，重新尝试或提醒用户重新尝试即可，死锁不可能完全避免，但有一些方法可以最小化其发生的概率：

1. 注意语句顺序：如果检测到两个事务总是发生死锁，检查它们的代码，这些事务可能是储存过程的一部分，看一下事务里的语句顺序，如果这些事务以相反的顺序更新记录，就很可能出现死锁，为了减少死锁，我们在**更新多条记录时可以遵循相同的顺序**
2. 尽量让你的事务小一些，持续时间短一些，这样就不太容易和其他事务相冲突
3. 如果你的事务要操作非常大的表，运行时间可能会很长，冲突的风险就会很高，看看能不能让这样的事物避开高峰期运行，以避开大量活跃用户

## 数据类型

![MySQL 常见字段类型总结](SQL进阶学习记录.assets/summary-of-mysql-field-types.png)

MySQL的数据分为以下几个大类：

1. String Types 字符串类型
2. Numeric Types 数字类型
3. Date and Time Types 日期和时间类型
4. Blog Types 存放二进制的数据类型
5. Spatial Types 存放地理数据的类型

### 字符串类型

**最常用的两个字符串类型**

1. `CHAR()` 固定长度的字符串，如州（'CA', 'NY', ……）就是 CHAR(2)
2. `VARCHAR()` 可变字符串

- Mosh习惯用 VARCHAR(50) 来记录用户名和密码这样的短文本 以及 用 VARCHAR(255) 来记录像地址这样较长一些的文本，保持这样的习惯能够简化数据库设计，不必每次都想每一列该用多长的 VARCHAR
- VARCHAR 最多能储存 64KB, 也就是最多约 65k 个字符（如果都是英文即每个字母只占一字节的话），超出部分会被截断

字符串类型也可以用来储存邮编，电话号码这样的特殊的数字型数据，因为它们不会用来做数学运算而且常常包含‘-’或括号等分隔符号

**储存较大文本的两个类型**

1. `MEDIUMTEXT` 最大储存16MB（约16百万的英文字符），适合储存JSON对象，CS视图字符串，中短长度的书籍
2. `LONGTEXT` 最大储存4GB，适合储存书籍和以年记的日志

**还有两个用的少一些的**

1. `TINYTEXT` 最大储存 255 Bytes
2. `TEXT` 最大储存 64KB，最大储存长度和 VARCHAR 一样，但最好用 VARCHAR，因为 VARCHAR 可以使用索引（之后会讲，索引可以提高查询速度）

**国际字符**

所有这些字符串类型都支持国际字符，其中：

- 英文字符占1个字节
- 欧洲和中东语言字符占2个字节
- 像中日这样的亚洲语言的字符占3个字节

所以，如果一列数据的类型为 CHAR(10)，MySQL会预留30字节给那一列的值

**导航**

下节课讲整数

### 整数类型

Integer Types (2:52)

我们用整数类型来保存没有小数的数字，MySQL里共有5种常用的整数类型，它们的区别在于占用的空间和能记录的数字范围

| 整数类型  | 占用储存 | 记录的数字范围 |
| --------- | -------- | -------------- |
| TINYINT   | 1B       | [-128,127]     |
| SMALLINT  | 2B       | [-32K,32K]     |
| MEDIUMINT | 3B       | [-8M,8M]       |
| INT       | 4B       | [-2B,2B]       |
| BIGINT    | 8B       | [-9Z,9Z]       |

**属性1. 不带符号 UNSIGNED**
这些整数可以选择不带符号，加上 `UNSIGNED` 则只储存非负数
如最常用的 `UNSIGNED TINYINT`，占用空间和 `TINYINT` 一样也是1B，但表示的数字范围不是 [-128-127] 而是 [0-255]，适合储存像年龄这样的数据，可以防止意外输入负数

**属性2. 填零 ZEROFILL**
整数类型的另一个属性是填零（Zerofill），主要用于当你需要给数字前面添加零让它们位数保持一致时
我们用**括号表示显示位数**，如 `INT(4)` => 0001，注意这**只影响MySQL如何显示数字而不影响如何保存数字**

**方法**

不用强行去记，谷歌 mysql integer types 即可查阅

**注意**

如果试图存入超出范围的数字，MySQL会抛出异常 'The value is out of range'

**最佳实践**

总是使用能满足你需求的最小整数类型，如储存人的年龄用 UNSIGNED TINYINT 就足够了，至少可见的未来内没人能活过255岁
数据需要在磁盘和内存间传输，虽然不同类型间只有几个字节的差异，但数据量大了以后对空间和查询效率的影响就很大了，所以在数据量较大时，有意识地分配每一字节，保持数据最小化是很有必要的。

------

### 定点数类型和浮点数类型

Fixedpoint and Floatingpoint Types (1:42)

这节主要讲储存小数的数据类型，有定点数和浮点数两种类型

**Fixedpoint Types 定点数类型**

`DECIMAL(p, s)` 两个参数分别指定最大的有效数字位数和小数点后小数位数（小数位数固定）
如：DECIMAL(9, 2) => 1234567.89 总共最多9位，小数点后两位，整数部分最多7位

`DECIMAL` 还有几个别名：`DEC` / `NUMERIC` / `FIXED`，最好就使用 `DECIMAL` 以保持一致性，但其它几个也要眼熟，别人用了要认识

**Floatingpoint Types 浮点数类型**

进行科学计算，要计算特别大或特别小的数时，就会用到浮点数类型，浮点数不是精确值而是近似值，这也正是它能表示更大范围数值的原因
具体有两个类型：

- `FLOAT` 浮点数类型，占用4B
- `DOUBLE` 双精度浮点数，占用8B，显然能比前者储存更大范围的数值

**小结**

如果需要记录**精确**的数字，比如货币金额，就是用 `DECIMAL` 类型
如果要进行科学计算，要处理很大或很小的数据，而且精确值不重要的话，就用 `FLOAT` 或 `DOUBLE`

------

### 布尔类型

Boolean Types (0:46)

有时我们需要储存 是/否 型数据，如 “这个博客是否发布了？”，这里我们就要用到布林值，来表示真或假

MySQL里有个数据类型叫 `BOOL` / `BOOLEAN`

**案例**

```sql
UPDATE posts 
SET is_published = TRUE / FALSE
或
SET is_published = 1 / 0
```

**注意**

布林值其实本质上就是 微整数 `TINYINT` 的另一种表现形式，TRUE / FALSE 实质上就是 1 / 0，但 Mosh 个人觉得写成 TRUE / FALSE 表意更清楚

------

### 枚举和集合类型

Enum and Set Types (3:36)

enumeration n. 枚举

有时我们希望某个字段从固定的一系列值中取值，我们就可以用到 `ENUM()` 和 `SET()` 类型，前者是取一个值，后者是取多个值

**ENUM()**

从固定一系列值中取一个值

**案例**

例如，我们希望 sql_store.products（产品表）里多一个size（尺码）字段，取值为 small/medium/large 中的一个，可以打开产品表的设计模式，添加size列，数据类型设置为 `ENUM('small','medium','large')`，然后apply

则产品表会增加一个尺码列，可将其中的值设为small/medium/large(大小写无所谓)，但若设为其他值会报错

**SET()**

SET和ENUM类似，区别是，SET是从固定一系列值中取多个值而非一个值

**注意**

讲解 ENUM 和 SET 只是为了眼熟，**最好不要用这两个数据类型**，问题很多：

1. 修改可选的值（如想增加一个'extra large'）会重建整个表，耗费资源
2. 想查询可选值的列表或者想用可选值当作一个下拉列表都会比较麻烦
3.  难以在其它表里复用，其它地方要用只有重建相同的列，之后想修改就要多处修改，又会很麻烦

**最佳实践**

像这种某个字段是从固定的一系列值中取值的情况，不应该使用 ENUM 和 SET 而应该用这一系列的可选值另外建一个 “查询表” (lookup table)

例如，上面案例中，应该为尺码另外专门建一个 size表（可选尺码表）

又如，sql_invoicing 里为支付方式另外专门建了一个 payment_methods 可选支付方式表

这样就解决了上面的所有问题，既方便查询可选值的列表，也方便作为下拉选项，也方便复用和更改

**导航**

下一章设计数据库讲里讲 normalization（标准化/归一化）时会更详细地讲解这个问题

### 日期和时间类型

Date and Time Types (0:44)

MySQL 有4种储存日期事件的类型：

1. `DATE` 有日期没时间
2. `TIME` 有时间没日期
3. DATETIME` 包含日期和时间` 
4. `TIMESTAMP` 时间戳，常用来记录一行数据的的插入或最后更新时间

最后两个的区别是：

- TIMESTAMP 占4B，最晚记录2038年，被称为“2038年问题”
- DATETIME 占8B，如果要储存超过2038年的日期时间，就要用 DATETIME

另外，还有一个 `YEAR` 类型专门储存四位的年份

### 二进制大对象类型

Blob Types (1:17)

我们用 `Blob` 类型来储存大的二进制数据，包括PDF，图像，视频等等几乎所有的二进制的文件
具体来说，MySQL里共有4种 Blob 类型，它们的区别在于可储存的最大文件大小：

| 占用储存    | 最大可储存 |
| ----------- | ---------- |
| TINYBOLB    | 255B       |
| BLOB        | 65KB       |
| MEDIUM BLOB | 16MB       |
| LONG BLOB   | 4GB        |

**注意**

通常应该将二进制文件存放在数据库之外，关系型数据库是设计来专门处理结构化关系型数据而非二进制文件的

如果将文件储存在数据库内，会有如下问题：

1. 数据库的大小将迅速增长
2. 备份会很慢
3. 性能问题，因为从数据库中读取图片永远比直接从文件系统读取慢
4. 需要额外的读写图像的代码

所以，尽量别用数据库来存文件，除非这样做确实有必要而且上面这些问题已经被考虑到了

### JSON类型

JSON Type (10:24)

**背景：关于JSON**

- MySQL还可以储存 JSON 文件，JSON 是 JavaScript Object Notation（JavaScript 对象标记法）的简称
- 简单讲，JSON 是一种在互联网上储存和传播数据的简便格式（Lightweight format for storing and transferring data over the Internet）
- JSON 在网络和移动应用中被大量使用，多数时候你的手机应用向后端传输数据都是使用 JSON 格式

语法结构：

```json
{
    "key1": value1,
    "key2": value2,
    ……
}
```

- JSON 用大括号{}表示一个对象，里面有多对键值对
- 键 key 必须用引号包裹（而且似乎必须是双引号，不能用单引号）
- 值 value 可以是数值，布林值，数组，文本， 甚至另一个对象（形成嵌套 JSON 对象）

**案例**

用 sql_store 数据库，在 products 商品表里，在设计模式下新增一列 properties，设定为 JSON 类型，注意在Workbench里，要将 Eidt-Preferences-Modeling-MySQL-Default Target MySQL Version 设定为 8.0 以上，不然设定 JSON 类型会报错 （我Workbench 里 Default Target MySQL Version 确实设定为 8.0 以上，但我电脑里的 MySQL 装的是 5.7 版本，似乎也没影响之后操作都很顺利，为什么？）

这里的 properties 记录每件产品附加的独特属性，注意这里每件产品的独特属性是不一样的，如衣服是颜色和尺码，而电视机是的重量和尺寸，把所有可能的属性都作为不同的列添加进表是很糟糕的设计，因为每个商品都只能用到所有这些属性的一部分（一个子集），相反，通过增加一列 JSON 类型的 properties 列，我们可以利用 JSON 里的键值对很方便的储存每个商品独特的属性

现在我们已经有了一个 JSON 类型的列，接下来从 **增** **删** **改** **查** 各角度来看看如何操作使用 JSON 类型的列，注意这里的 增删查改 主要针对的是 properties 列里的特定键值对，即如何 增删查改 某些特定的具体属性

**增**

给1号商品增加一系列属性，有两种方法

*法1：*

用单引号包裹（注意不能是双引号），里面用 JSON 的标准格式：

- 双引号包裹键 key（注意不能是单引号）
- 值 value 可以是数、数组、甚至另一个用 `{}` 包裹的JSON对象
- 键值对间用逗号隔开

```sql
USE sql_store;
UPDATE products
SET properties = '
{
    "dimensions": [1, 2, 3], 
    "weight": 10,
    "manufacturer": {"name": "sony"}
}
'
WHERE product_id = 1;
```

*法2：*

也可以用 MySQL 里的一些针对 JSON 的内置函数来创建商品属性：

```sql
UPDATE products
SET properties = JSON_OBJECT(
    'weight', 10,
    -- 注意用函数的话，键值对中间是逗号而非冒号
    'dimensions', JSON_ARRAY(1, 2, 3),
    'manufacturer', JSON_OBJECT('name', 'sony')
)
WHERE product_id = 1;
```

两个方法是等效的

**查**

现在来讲如何查询 JSON 对象里的特定键值对，这是将某一列设为 JSON 对象的优势所在，如果 properties 列是字符串类型如 VARCHAR 等，是很难获取特定的键值对的

有两种方法：

*法1 :*

使用 `JSON_EXTRACT(JSON对象, '路径')` 函数，其中：

- 第1参数指明 JSON 对象
- 第2参数是用单引号包裹的路径，路径中 `$` 表示当前对象，点操作符 `.` 表示对象的属性

```sql
SELECT product_id, JSON_EXTRACT(properties, '$.weight') AS weight
FROM products
WHERE product_id = 1;
```

法2

更简便的方法，使用列路径操作符 `->` 和 `->>`，后者可以去掉结果外层的引号

用法是：`JSON对象 -> '路径'`

```sql
SELECT properties -> '$.weight' AS weight
FROM products
WHERE product_id = 1;
-- 结果为：10

SELECT properties -> '$.dimensions' 
……
-- 结果为：[1, 2, 3]

SELECT properties -> '$.dimensions[0]' 
-- 用中括号索引切片，且序号从0开始，与Python同
……
-- 结果为：1

SELECT properties -> '$.manufacturer'
……
-- 结果为：{"name": "sony"}

SELECT properties -> '$.manufacturer.name'
……
-- 结果为："sony"

SELECT properties ->> '$.manufacturer.name'
……
-- 结果为：sony
```

通过路径操作符来获取 JSON 对象的特定属性不仅可以用在 SELECT 选择语句中，也可以用在 WHERE 筛选语句中，如：
筛选出制造商名称为 sony 的产品：

```sql
SELECT 
    product_id, 
    properties ->> '$.manufacturer.name' AS manufacturer_name
FROM products
WHERE properties ->/->> '$.manufacturer.name' = 'sony'
```

结果为：

| product_id | manufacturer_name |
| ---------- | ----------------- |
| 1          | sony              |

Mosh说最后这个查询的 WHERE 条件语句里用路径获取制作商名字时必须用双箭头 ->> 才能去掉结果的双引号，才能使得比较运算成立并最终查找出符合条件的1号产品，但实验发现用单箭头 -> 也可以，但另一方面在 SELECT 选择语句中用单双箭头确实会使得显示的结果带或不带双引号，所以综合来看，单双箭头应该是只影响路径结果 "sony" 是否【显示】外层的引号，但不会改变其实质，所以不会影响其比较运算结果，即单双箭头得出的sony都是 = 'sony' 的

**改**

如果我们是要重新设置整个 JSON 对象就用前面 **增** 里讲到的 `JSON_OBJECT()` 函数，但如果是想修改已有 JSON 对象里的一些属性，就要用 `JSON_SET()` 函数

```sql
USE sql_store;
UPDATE products
SET properties = JSON_SET(
    properties,
    '$.weight', 20,  -- 修改weight属性
    '$.age', 10  -- 增加age属性
)
WHERE product_id = 1;
```

注意 `JSON_SET()` 是选择已有的 JSON 对象并修改部分属性然后返回修改后新的 JSON 对象，所以其第1参数是要修改的 JSON 对象，并且可以用

```sql
SET porperties = JSON_SET(properties, ……)
```

的语法结构来实现对 properties 的修改

**删**

可以用 `JSON_REMOVE()` 函数实现对已有 JSON 对象特性属性的删除，原理和 `JSON_SET()` 一样

```sql
USE sql_store;
UPDATE products
SET properties = JSON_REMOVE(
    properties,
    '$.weight',
    '$.age'
)
WHERE product_id = 1;
```

**小结**

增：利用标准格式或利用 `JSON_OBJECT`, `JSON_ARRAY` 等函数

查：`JSON_EXTRACT` 或 `->/-->`，注意表达路径时单引号、 `$` 和 `.` 的使用

改：`JSON_SET`，注意其原理

删：`JSON_REMOVE`，原理同上