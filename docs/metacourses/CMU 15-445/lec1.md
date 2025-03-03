# Relational Model and Algebra

## Database System

> Database: Organized **collection** of inter-related **data** 

> Database Manage System: systems that manage data bases

数据库是存在内部关联的数据的集合（e.g. Excel Table），而数据库系统是管理数据库的工具（e.g. MySQL,MiniSQL...）这两者有着本质的区别。

!!! example

    假如我们现在有一个音乐软件的数据库，用于记录作者和专辑:

    - Information about **Artists**
    - What **Albums** those Artists released

    一个很自然的想法是我们直接用 磁盘文件来记录这些信息 (e.g. CSV File)，例如，我们会用逗号来分割各个项。
    > Artist(name, year, country)
    ```CSV
    "Wu-Tang Clan",1992,"USA"
    "Notorious BIG",1992,"USA"
    "GZA",1990,"USA"
    ```

    >Album(name, artist, year)
    ```CSV
    "Enter the Wu-Tang","Wu-Tang Clan",1993
    "St.Ides Mix Tape","Wu-Tang Clan",1994
    "Liquid Swords","GZA",1990
    ```

    显然，这样的 FLAT FILES 有很多的缺点，比如：查询时必须展开遍历所有的信息和条目
    > 假如我们想要在 Artist 中查找 GZA 这一项的信息
    ```python
    for line in file.readlines():
    record = parse(line)
    if record[0] == "GZA":
    print(int(record[1]))
    ```


    - 在这个例子中 FLAT FILES 的一些问题：
        + 数据完整性
            - 我们如何确定对于同一张专辑的作者是同一个呢？（e.g. 春日影）
            - 如果有人拿非法的字符串来修改专辑年份呢？
            - 如果一张专辑有多个作者参与呢？(e.g. 春日影)
            - 如果我们删除了某个有专辑的作者呢？(e.g. ok这次是库来库西)
        + 数据库的实现
            - 你该如何查找一个特定的专辑呢？
            - 当我们创建一个新的app时，我们可以使用同一个数据库吗？如果我们有多个设备同时访问呢？
            - 如果存在两个进程同时写入数据呢？
        + 数据的维护
            - 当在修改一个数据时，程序崩溃了怎么办？
            - 当一个数据库被反复访问时，我们想复制该怎么办？

## DBSM & DATA MODELS
DATA BASE MANAGEMENT SYSTEM（DBSM）可以使应用对数据进行存储和分析。

一个DBSM应该支持: **definition**,**creation**, **querying**, **update**,  **administration**

Data Models 数据模型是一组用于描述数据库中数据的**模型**。Schema 是在给定 Data Models 的情况下对一组特定数据的描述。

常见的 Data Models:

- **Relational** 
- Key/Value
- Graph
- Document / XML / Object
- Wide-Column / Column-family
- Array / Matrix / Vectors
- Hierarchical
- Network
- Multi-Value
    
对于绝大部分的数据库而言，都使用关系（Relation），来作为 Data Model ，而NoSQL的数据库则使用Key/Value、Graph、Document / XML / Object。机器学习使用的是 Array / Matrix / Vectors，而最后的三种则较为过时与罕见。

## RELATIONAL MODEL

Relational Model 通过关系来抽象数据，来避免额外的开销。其关键在于：

- 通过简单的数据结构来储存数据库
- 物理层面的储存由 DBMS 的具体实现来决定
- 在应用层访问、修改数据时，由DBMS来决定具体的最佳的策略

构建一个关系数据模型：0

- 我们该如何定义其结构？关系中包含了哪些内容？
- 我们该如何确保数据库的内容的合法性？
- 我们该如何编程实现查找和删除数据库的内容？

!!!quote
    A relation is an **unordered** set that contain the relationship of **attributes** that represent entities.
    
    A tuple is a set of **attribute** values (also known as its domain) in the relation.
    
    ——《离散数学及其应用》

tips : 通常而言，数据值都是标量。但同时 NULL 可以作为任意元组的值。

**Primary Key**: 可以 **唯一** 的确定一个 Tuple。当一张表格没有一个 Primary Key 的时候，部分 DBMS 会自动的生成一个 Primary Key。

??? example
    <center>
    Artist(id,name,year,country)

    |id|name|year|country|
    |:--:|:--:|:--:|:--:|
    |101|"Wu-Tang Clan|1992|"USA"|
    |102|"Notorious BIG|1992|"USA"|
    |103|"GZA|1990|"USA"|

    </center>
    例如，在上面的例子中，id与每一个tuple就是一一对应的。

**Foreign key**: 每个数据表都是由关系来连系彼此的关系，主键（Primary Key）会放在另一个数据表，当做属性以建立彼此的关系，而这个属性就是Foreign Key。

??? example
    继续考虑我们之前选用的例子：
    <center>
    Album(id,name,artist,year)

    |id|name|artist|year|
    |:--:|:--:|:--:|:--:|
    |11|"Enter the Wu-Tang"|101|1993|
    |12|"St.Ides Mix Tape"|102|1994|
    |13|"Liquid Swords"|103|1995|
    </center>

    此时，我们将 artist 的 Primary Key 与 Album 的 Primary Key 放在一起,的 attribute 这样就叫做Foreign Key

    <center>
    Album(id,name,artist,year)

    |artist_id|album_id|
    |:--:|:--:|
    |101|11|
    |102|12|
    |103|13|

    </center>

**Constraints**:数据库系统的规范和约定。可以是用户指定的，也可能是系统所规定的。

## DATA MANIPULATION LANGUAGES

数据操纵语言（DML）用于描述数据库中存储、编辑信息的方式。

- Procedural: 准确的定义了我们查找等操作采用的策略。 <-Relational Algebra
- Non-Procedural: 只确定了所需的信息，没有指定我们的策略。（由DBMS决定）<-Relational Calculus

## RELATIONAL ALGEBRA

|Select|Projection|Union|Intersection|Difference|Product|Join|
|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
|$\sigma$|$\pi$|$\cup$|$\cap$|$-$|$\times$|$\Join$|

### SELECT
根据我们谓语(predicate)，从一个关系中选择一个元组的子集。（谓语就是我们的条件）

>使用的形式： $\sigma_{predicate}(R)$

??? example
    <center>

    R(a_id,b_id)

    |a_id|b_id|
    |:--:|:--:|
    |a1|101|
    |a2|102|
    |a2|103|
    |a3|104|

    </center>

    <center>

    $\sigma_{a\_id = \'{} a2 \'{} \land b\_id > 102} (R)$

    |a_id|b_id|
    |:--:|:--:|
    |a2|103|

    </center>
    而这里等价于
    ```MySQL
    SELECT * FROM R
        WHERE a_id = 'a2' AND b_id >  102
    ```

### PROJECTION

根据指定的属性，从元组中生成一个关系。可能涉及到：
- 重新排列属性
- 删除不需要的属性
- 对属性的值进行一些操作，来得到我们想要的属性

>使用的形式： $\pi_{A1,A2,\dots,An}(R)$

??? example
    <center>

    R(a_id,b_id)

    |a_id|b_id|
    |:--:|:--:|
    |a1|101|
    |a2|102|
    |a2|103|
    |a3|104|

    </center>

    <center>

    $\pi_{b\_id - 100,a\_id}(\sigma_{a\_id = \'{} a2 \'{} } (R))$

    |b\_id - 100|a_id|
    |:--:|:--:|
    |2|a2|
    |3|a2|

    </center>
    而这里等价于
    ```MySQL
    SELECT b_id-100, a_id
        FROM R WHERE a_id = 'a2';
    ```
### UNION & INTERSECTION

生成两个关系的并或交。

> 使用的形式： $R \bigcup S$ 或 $R \bigcap S$ 

### DIFFERENCE

生成一个新的关系，只包含在第一个关系中，但不在第二个关系中的元组。

> 使用的形式： $R - S$ 

??? example
    假如我们有关系 R 和 S:
    <center>

    R(a_id,b_id)

    |a_id|b_id|
    |:--:|:--:|
    |a1|101|
    |a2|102|
    |a2|103|

    </center>

    <center>

    
    S(a_id,b_id)

    |a_id|b_id|
    |:--:|:--:|
    |a3|103|
    |a4|104|
    |a5|105|

    </center>

    <center>

    
    R-S(a_id,b_id)

    |a_id|b_id|
    |:--:|:--:|
    |a1|101|
    |a2|102|

    </center>
    而这里等价于
    ```MySQL
    (SELECT * FROM R)
        EXCEPT
    (SELECT * FROM S);
    ```

### PRODUCT

从两个关系生成一个新的关系，de facto其包含原关系中元组所有的组合。

>使用的方式： $R \times S$

??? example
    假如我们有关系 R 和 S:
    <center>

    R(a_id,b_id)

    |a_id|b_id|
    |:--:|:--:|
    |a1|101|
    |a2|102|
    |a2|103|

    </center>

    <center>

    
    S(a_id,b_id)de facto
    |:--:|:--:|
    |a3|103|
    |a4|104|
    |a5|105|

    </center>

    <center>

    R $\times$ S

    |R.a_id |R.b_id |S.a_id |S.b_id|
    |:--:|:--:|:--:|:--:|
    |a1| 101| a3 |103|
    |a1| 101| a4 |104|
    |a1| 101| a5 |105|
    |a2| 102| a3 |103|
    |a2| 102| a4 |104|
    |a2| 102| a5 |105|
    |a3| 103| a3 |103|
    |a3| 103| a4 |104|
    |a3| 103| a5 |105|

    </center>

    ```MySQL
    SELECT * FROM R CROSS JOIN S;
    ```

### JOIN

生成一个新的关系，其中包含了前两个关系元组的组合，并且这些元组具有共有的属性值。

>使用的方法： (R $Join$ S)

??? example
    假如我们有关系 R 和 S:
    <center>

    R(a_id,b_id)

    |a_id|b_id|
    |:--:|:--:|
    |a1|101|
    |a2|102|
    |a2|103|

    </center>

    <center>

    
    S(a_id,b_id,val)

    |a_id|b_id|val|
    |:--:|:--:|:--:|
    |a3|103|xxx|
    |a4|104|yyy|
    |a5|105|zzz|

    </center>

    <center>

    R $\Join$ S

    |a_id |b_id |val|
    |:--:|:--:|:--:|
    |a3| 103|xxx|

    </center>

    ```MySQL
    SELECT * FROM R NATURAL JOIN S;
    ```

我们还有更多的关系算符：

- Rename $\rho$
- Assignment $R \leftarrow S$
- Duplicate Elimination $\delta$
- Aggregation $\gamma$
- Sorting $\Tau$
- Division $R \div S$

基于关系代数，我们可以对查询的过程进行方法的优化。例如 $S \Join(\sigma_{a\_id = 100}(R))$ 与 $\sigma_{a\_id = 100}(S\Join R)$ 在性能上将会有巨大的差异。DBMS 在关系代数的帮助下可以更好地优化这个过程（类似于编译器）。
