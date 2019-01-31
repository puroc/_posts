---
title: 数据库知识点-DML-数据操纵
date: 2019-01-30 15:23:41
tags:
categories:
---
[文档链接](http://www.postgres.cn/docs/10/dml.html)
## 1、插入数据
向如下产品表插入数据

```
CREATE TABLE products (
    product_no integer,
    name text,
    price numeric
);
```
* 插入一行数据，数据的值是按照这些列在表中出现的顺序列出的。
```
INSERT INTO products VALUES (1, 'Cheese', 9.99);
```

* 指定列的顺序进行插入

```
INSERT INTO products (product_no, name, price) VALUES (1, 'Cheese', 9.99);
INSERT INTO products (name, price, product_no) VALUES ('Cheese', 9.99, 1);
```
* 插入数据时使用缺省值

以下第二种形式是PostgreSQL的一个扩展。它从使用给出的值从左开始填充列，有多少个给出的列值就填充多少个列，其他列的将使用缺省值。

```
INSERT INTO products (product_no, name) VALUES (1, 'Cheese');
INSERT INTO products VALUES (1, 'Cheese');
```
* 一次插入多行

```
INSERT INTO products (product_no, name, price) VALUES
    (1, 'Cheese', 9.99),
    (2, 'Bread', 1.99),
    (3, 'Milk', 2.99);
```
* 也可以插入查询的结果（可能没有行、一行或多行）

```
INSERT INTO products (product_no, name, price)
  SELECT product_no, name, price FROM new_products
    WHERE release_date = 'today';
```
## 2、更新数据

```
UPDATE products SET price = 10 WHERE price = 5;
```
新的列值可以是任意标量表达式， 而不仅仅是常量。

```
UPDATE products SET price = price * 1.10;
```
一次更新多列

```
UPDATE mytable SET a = 5, b = 3, c = 1 WHERE a > 0;
```
## 3、删除数据

```
DELETE FROM products WHERE price = 10;
```
## 4、从修改的行中返回数据
 RETURNING可以返回分配给新行的ID
 
```
CREATE TABLE users (firstname text, lastname text, id serial primary key);

INSERT INTO users (firstname, lastname) VALUES ('Joe', 'Cool') RETURNING id;
```
在UPDATE中，可用于RETURNING的数据是被修改行的新内容

```
UPDATE products SET price = price * 1.10
  WHERE price <= 99.99
  RETURNING name, price AS new_price;
```
在DELETE中，可用于RETURNING的数据是删除行的内容

```
DELETE FROM products
  WHERE obsoletion_date = 'today'
  RETURNING *;
```

