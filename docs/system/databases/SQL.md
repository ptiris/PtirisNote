# SQL 基本语法与使用

!!! tip "Example 提示"
    在本节的例子中我们主要的情景建立在如下的银行例子上

    - branch(branch-name, branch-city, assets) 
    - customer(customer-name, customer-street, customer-city) 
    - account(account-number, branch-name, balance) 
    - loan(loan-number, branch-name, amount) 
    - depositor(customer-name, account-number) 
    - borrower(customer-name, loan-number)

    ![](./assets/1.png)

## Data Definition Language 

DDL 在数据库中是用于定义数据库结构与数据库对象(schema,domain,integrity constraints,physical storage,indices,view)的语言。在DDL中我们主要通过 ``` CREATE DROP ALTER ```三个关键字来增/删/改。

### Domain Types

Domain 用于限定数据库中元素的“范围”

- char: 定长的字符串（用户指定）
- varchar: 不定长的字符串（用户指定最大长度）
- int: 整数（具体的位数由运行的机器限制）
- smallint: 小整数（具体的位数由运行的机器限制）
- numeric(p,d): 固定小数点的实数，由用户指定精度 p 和 小数位数 d 
- real,double precision: 浮点数，与IEEE标准相同
- float(n)：固定精度浮点数
- Null valus: 空值
- date/time/timestamp: 日期(2025-3-3)、时间(18:45:02)、时间戳(2025-3-3 18:45:02)

关于这些 Type 的实现，你可以在 [CMU 15/445 Database Storage](https://15445.courses.cs.cmu.edu/fall2023/slides/04-storage2.pdf "真得是仙品") 中查阅更多的细节（底层的实现与考量）

这里我们额外讨论一下 NULL这种特殊的类型。NULL一般用于只带未知的值或者值不存在。

- 任何对于NULL的算术运算将会返还NULL
- 任何对于NULL的比较运算将会返还NULL
- 对于NULL的逻辑运算如下：
    + OR:  
           - (unknown or true) = true 
           - (unknown or false) = unknown 
           - (unknown or unknown) = unknown 
    + AND:  
           * (true and unknown) = unknown 
           * (false and unknown) = false 
           * (unknown and unknown) = unknown 
    + NOT: (not unknown) = unknown

我们可以通过 `IS NULL` 来检查是否是空值（这里不用等号，因为 NULL = ？ 没有意义）

但是，聚合函数会直接忽略掉所有的空值

### Tables

我们可以通过 ```#!sql CREATE Table name (Attr1 Domain1 Constraint1, ...)``` 来创建不同的表格。

其中 Constraint 包含:

- Not null 非空
- Primary key 指定这个属性为主键(当然主键一定非空)
- Check (P) 要求填入表格的值符合谓语P

```sql
CREATE TABLE branch2
(
    branch_name char(20) primary key,
    branch_city char(30) not null, 
    assets      integer, 
    check (assets >= 0)
); 
```

我们可以通过 ```#!sql DROP Table name``` 来删除表格。
```sql
DROP TABLE branch2;
DROP TABLE bank.branch2;   #删除bank数据库中的branch2表格
```

我们可以通过 ```#!sql ALTER Table name (Attr1 Domain1 Constraint1, ...)``` 来为表格增加属性；```#!sql DROP Table name Attr1``` 来为表格删除属性。我们可以通过 ```#!sql ALTER TABLE branch MODIFY (Attr1 Domain1 Constraint1, ...)``` 来修改表格的属性。

### Index

Index索引是通过在部分属性建立数据结构（通常是 HashTable 与 B+Tree）的方法来增快部分数据的查询速度(1)。
{   .annotate   }

1.  [CMU 15/445 Hash Table Index](../../metacourses/CMU%2015-445/lec5.md) 和 [CMU 15/445 B+Tree Index](../../metacourses/CMU%2015-445/lec6.md)

我们可以通过 `CREATE` 和 `DROP` 来增加和删除 INDEX

```sql
CREATE INDEX b_index ON branch (branch_name); 
CREATE INDEX cust_strt_city_index ON customer (customer_city, customer_street); 
DROP INDEX b_index; 

```

## Data Manipulation Language

### Select Clause

从某种意义而言 Select 语句相当于关系代数中的 $\prod_{attr}$。

我们可以直接直接通过属性的名字来进行选择（选择的内容可以是某个属性，也可以是经过运算的结果，可以是某些函数...）；可以通过 DISTINCT 关键字来对选择的属性进行去重；而关键字 * 则代表所有属性。
```sql
SELECT branch_name 
FROM loan ;

SELECT distinct branch_name 
FROM loan ;
```

### From Clause
FROM Clause可以表明在查询中涉及的关系（这个关系可以是单张表格、多张表格、甚至是另一个查询运算的结果）。当FROM选定多个表格时，可以表示它们的笛卡尔积，例如```#!sql SELECT * FROM borrower, loan ``` 即为 $borrower \times loan$

!!! example 
    Find the customer name, loan number and loan amount of all customers having a loan at the Perryridge branch. 

    ```sql
    SELECT customer_name, borrower.loan_number, amount 
    FROM   borrower, loan 
    WHERE  borrower.loan_number = loan.loan_number and 
                branch_name = ‘Perryridge’ 
    ```

### Rename Option

我们可以通过 `AS` 关键字对表格、属性取一个别名。重命名的操作可以出现在多个地方：可以出现在 Select 中为属性重命名；可以在 From 中为表格重命名...其中，在From从句的命名尤为重要，它可以用于在求一个表格与自身的笛卡尔积中消除歧义。

```sql
SELECT customer_name, borrower.loan_number as loan_id, amount  
    FROM borrower, loan 
    WHERE borrower.loan_number = loan.loan_number;

SELECT customer_name, T.loan_number, S.amount 
   FROM borrower as T, loan as S 
   WHERE T.loan_number = S.loan_number ;

SELECT distinct T.branch_name 
    FROM branch as T, branch as S 
    WHERE T.assets > S.assets and S.branch_city = ‘Brooklyn’ ;
```

### String Option
由于不同的DBMS实现的问题，在字符串上的操作有着很大的不同。仅在字符串的拼接上，就出现了 `+``||` `strcat` 等多种不同的方式。但总体而言他们都指代着同一种功能。所以我们以课件上介绍的为准。

SQL提供了字符串匹配的运算符，用于字符串的查找(需要通过 LIKE 关键字进行):

- '%' 可以匹配任意的子串 

- '_' 可以匹配任意的字符

SQL中可以通过` ||` 进行字符串的连接；可以通过 `LOWER` 和 `UPPER` 进行大小写的转换；可以通过 `SUBSTR(pos,len)` 进行子串的裁剪；可以通过 `LEN` 来查看字符串的长度。

### Order By
我们可以通过 ORDER 关键字来实现排序。我们所指定的类型需要具有可排序的性质。
我们可以根据多个属性进行排序，即当当前属性相同时，按照下一属性进行排序；同时可以在属性后加上关键字 `DESC` 来表明该属性按照降序排序。 
```sql
SELECT distinct customer_name 
	FROM borrower A, loan B 
		WHERE A.loan_number = B.loan_number and 
			 branch_name = ‘Perryridge’ 
	ORDER BY customer_name；


SELECT * FROM customer 
    ORDER BY customer_city, customer_street desc, customer_name 
```

### Set Operations

在 SQL 中，我们可以用 `UNION` 来执行 $\cup$ 操作；可以用 `INTERSECT` 来执行 $\cap$ 操作；可以使用 `EXCEPT` 来执行 `-` 操作。但由于集合的性质，这三个操作保证一定会消除相同的元组的复制。当我们想要保留所有的元组的复制时，我们需要加上` ALL `关键字来保证其不会消除复制。

在这里同样存在“方言”的问题：Oracle通过 MINUS 来实现 EXCEPT；SQL Server 仅支持 UNION。

```sql
(SELECT customer_name FROM depositor) 
UNION 
(SELECT customer_name FROM borrower) ;

(SELECT customer_name FROM depositor) 
INTERSECT 
(SELECT customer_name FROM borrower) ;
```

### Aggregate Functions

我们可以通过 Aggregate Functions 对表格中的元素进行函数操作:

- avg(col): average value 
- min(col): minimum value 
- max(col): maximum value 
- sum(col): sum of values 
- count(col): number of values 

!!! warning
    一个非常重要的点是：当我们的 SELECT 中同时选中了返回单一值的对列的 Aggregate Function 和 某一列属性时，需要通过 GROUP BY 来使得查询有意义；这是因为返回单一值与某一列的组合是没有意义的。

    ```sql
    SELECT branch_name, avg(balance) avg_bal 
      FROM account 
      GROUP BY branch_name 

    SELECT branch_name, count(customer_name) tot_num 
      FROM depositor, account 
      WHERE depositor.account_number=account.account_number 
      GROUP BY branch_name 
    ```

GROUP BY 返回的值同样可以作为我们的选择条件之一，此时这个条件需要放到 HAVING 从句中。所有的聚合函数的结果的筛选都需要放入 Having 中，因为 Having 从句代表对分组后的结果进行筛选；而 Where 语句代表对分组前的数据进行筛选。

### Conclusion

事实上，我们执行一个 SELECT 查询的顺序为：

```#!sql FROM → WHERE → GROUP (AGGREGATE) → HAVING → SELECT → DISTINCT → ORDER BY ```

## 