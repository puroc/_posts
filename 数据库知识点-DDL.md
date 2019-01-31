---
title: 数据库知识点-DDL
date: 2019-01-30 15:22:46
tags: 数据库
categories: 数据库知识点
---
[全文目录](https://puroc.github.io/blog/2019/01/30/数据库知识点-目录/)
## 1、数据库管理
### 1.1、创建数据库
#### 语法
[文档连接](http://www.postgres.cn/docs/10/sql-createtable.html)
#### 例子：

```
CREATE DATABASE lusiadas;
```
```
CREATE DATABASE music2
    LC_COLLATE 'sv_SE.iso885915' LC_CTYPE 'sv_SE.iso885915'
    ENCODING LATIN9
    TEMPLATE template0;
```
### 1.2、修改数据库
#### 语法
[文档连接](http://www.postgres.cn/docs/10/sql-alterdatabase.html)
#### 例子
```
ALTER DATABASE test SET enable_indexscan TO off;
```

### 1.3、删除数据库
#### 语法

```
DROP DATABASE [ IF EXISTS ] name
```
#### 参数说明
IF EXISTS：删除数据库时，如果数据库不存在，也不会抛出错误
#### 例子
```
DROP DATABASE IF EXISTS music2;
```
## 2、表管理
### 2.1、创建表
#### 语法
[文档连接](http://www.postgres.cn/docs/10/sql-createtable.html)
#### 例子
[文档连接](http://www.postgres.cn/docs/10/ddl.html)
##### 1、默认值
在一个表定义中，默认值被列在列的数据类型之后。例如：

```
CREATE TABLE products (
    product_no integer,
    name text,
    price numeric DEFAULT 9.99
);
```
默认值可以是一个表达式，它将在任何需要插入默认值的时候被实时计算（不是表创建时）。一个常见的例子是为一个timestamp列指定默认值为CURRENT_TIMESTAMP，这样它将得到行被插入时的时间。另一个常见的例子是为每一行生成一个“序列号” 。这在PostgreSQL可以按照如下方式实现：这里nextval()是函数

```
CREATE TABLE products (
    product_no integer DEFAULT nextval('products_product_no_seq'),
    ...
);
```
##### 2、检查约束
检查约束有列约束和表约束两种用法。列约束例子如下，如你所见，约束定义就和默认值定义一样跟在数据类型之后。

```
CREATE TABLE products (
    product_no integer,
    name text,
    price numeric CHECK (price > 0)
);
```
表约束例子如下，其中第三个约束使用了一种新语法。它并没有依附在一个特定的列，而是作为一个独立的项出现在逗号分隔的列表中，这就是表检查约束。

```
CREATE TABLE products (
    product_no integer,
    name text,
    price numeric CHECK (price > 0),
    discounted_price numeric CHECK (discounted_price > 0),
    CHECK (price > discounted_price)
);
```
##### 3、非空约束
非空约束指定一个列中不会有空值

```
CREATE TABLE products (
    product_no integer NOT NULL,
    name text NOT NULL,
    price numeric
);
```
一个列可以有多于一个的约束，只需要将这些约束一个接一个写出，约束的顺序没有关系。

```
CREATE TABLE products (
    product_no integer NOT NULL,
    name text NOT NULL,
    price numeric NOT NULL CHECK (price > 0)
);
```
##### 4、唯一约束
唯一约束保证\在一列中或者一组列中保存的数据在表中所有行间是唯一的

```
CREATE TABLE products (
    product_no integer UNIQUE,
    name text,
    price numeric
);
```
要为一组列定义一个唯一约束，把它写作一个表级约束，列名用逗号分隔

```
CREATE TABLE example (
    a integer,
    b integer,
    c integer,
    UNIQUE (a, c)
);
```
**注意：
1）如果表中有超过一行在约束所包括列上的值相同，将会违反唯一约束。但是在这种比较中，两个空值被认为是不同的。
2）增加一个唯一约束会在约束中列出的列或列组上自动创建一个唯一B-tree索引**
##### 5、主键
一个主键约束表示可以用作表中行的唯一标识符的一个列或者一组列。这要求那些值都是唯一的并且非空

```
CREATE TABLE products (
    product_no integer PRIMARY KEY,
    name text,
    price numeric
);
```
主键也可以包含多于一个列

```
CREATE TABLE example (
    a integer,
    b integer,
    c integer,
    PRIMARY KEY (a, c)
);
```
**注意：
1) 增加一个主键将自动在主键中列出的列或列组上创建一个唯一B-tree索引**

##### 6、外键
一个外键约束指定一列（或一组列）中的值必须匹配出现在另一个表中某些行的值。
例如我们有一个使用过多次的产品表：

```
CREATE TABLE products (
    product_no integer PRIMARY KEY,
    name text,
    price numeric
);
```
让我们假设我们还有一个存储这些产品订单的表。我们希望保证订单表中只包含真正存在的产品的订单。因此我们在订单表中定义一个引用产品表的外键约束，现在就不可能创建包含不存在于产品表中的product_no值（非空）的订单。在这种情况下，订单表是引用表而产品表是被引用表。

```
CREATE TABLE orders (
    order_id integer PRIMARY KEY,
    product_no integer REFERENCES products (product_no),
    quantity integer
);
```
一个外键也可以约束和引用一组列。照例，它需要被写成表约束的形式。当然被约束列的数量和类型应该匹配被引用列的数量和类型。下面是一个例子：

```
CREATE TABLE t1 (
  a integer PRIMARY KEY,
  b integer,
  c integer,
  FOREIGN KEY (b, c) REFERENCES other_table (c1, c2)
);
```
我们知道外键不允许创建与任何产品都不相关的订单。但如果一个产品在一个引用它的订单创建之后被移除会发生什么？SQL允许我们处理这种情况。直观上，我们有几种选项：

* 不允许删除一个被引用的产品

* 同时也删除引用产品的订单

* 其他？

为了说明这些，让我们在上面的多对多关系例子中实现下面的策略：当某人希望移除一个仍然被一个订单引用（通过order_items）的产品时 ，我们组织它。如果某人移除一个订单，订单项也同时被移除：

```
CREATE TABLE products (
    product_no integer PRIMARY KEY,
    name text,
    price numeric
);

CREATE TABLE orders (
    order_id integer PRIMARY KEY,
    shipping_address text,
    ...
);

CREATE TABLE order_items (
    product_no integer REFERENCES products ON DELETE RESTRICT,
    order_id integer REFERENCES orders ON DELETE CASCADE,
    quantity integer,
    PRIMARY KEY (product_no, order_id)
);
```
限制删除或者级联删除是两种最常见的选项。RESTRICT阻止删除一个被引用的行。NO ACTION表示在约束被检察时如果有任何引用行存在，则会抛出一个错误，这是我们没有指定任何东西时的默认行为（这两种选择的本质不同在于NO ACTION允许检查被推迟到事务的最后，而RESTRICT则不会）。CASCADE指定当一个被引用行被删除后，引用它的行也应该被自动删除。还有其他两种选项：SET NULL和SET DEFAULT。这些将导致在被引用行被删除后，引用行中的引用列被置为空值或它们的默认值。注意这些并不会是我们免于遵守任何约束。例如，如果一个动作指定了SET DEFAULT，但是默认值不满足外键约束，操作将会失败。

与ON DELETE相似，同样有ON UPDATE可以用在一个被引用列被修改（更新）的情况，可选的动作相同。在这种情况下，CASCADE意味着被引用列的更新值应该被复制到引用行中。

正常情况下，如果一个引用行的任意一个引用列都为空，则它不需要满足外键约束。如果在外键定义中加入了MATCH FULL，一个引用行只有在它的所有引用列为空时才不需要满足外键约束（因此空和非空值的混合肯定会导致MATCH FULL约束失败）。如果不希望引用行能够避开外键约束，将引用行声明为NOT NULL。

一个外键所引用的列必须是一个主键或者被唯一约束所限制。这意味着被引用列总是拥有一个索引（位于主键或唯一约束之下的索引），因此在其上进行的一个引用行是否匹配的检查将会很高效。由于从被引用表中DELETE一行或者UPDATE一个被引用列将要求对引用表进行扫描以得到匹配旧值的行，在引用列上建立合适的索引也会大有益处。由于这种做法并不是必须的，而且创建索引也有很多种选择，所以外键约束的定义并不会自动在引用列上创建索引。
### 2.2、修改表
#### 语法
[文档连接](http://www.postgres.cn/docs/10/sql-altertable.html)
#### 例子
要向一个表增加一个类型为varchar的列：

```
ALTER TABLE distributors ADD COLUMN address varchar(30);
```
要从表中删除一列：

```
ALTER TABLE distributors DROP COLUMN address RESTRICT;
```
要在一个操作中更改两个现有列的类型：

```
ALTER TABLE distributors
    ALTER COLUMN address TYPE varchar(80),
    ALTER COLUMN name TYPE varchar(100);
```
### 2.3、删除表
#### 语法
[文档连接](http://www.postgres.cn/docs/10/sql-droptable.html)
#### 例子

```
DROP TABLE IF EXISTS films, distributors;
```
## 3、数据类型
[文档连接](http://www.postgres.cn/docs/10/datatype.html)
### 1、数字类型
![](https://puroc521blog.oss-cn-beijing.aliyuncs.com/20190131104815.png)
### 2、字符类型
![](https://puroc521blog.oss-cn-beijing.aliyuncs.com/20190131105254.png)
### 3、日期时间类型
![](https://puroc521blog.oss-cn-beijing.aliyuncs.com/20190131105903.png)
### 4、JSON类型
### 5、数组类型



