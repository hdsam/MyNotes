# MySQL的使用

[TOC]



## 一 、配置文件及目录说明

1. PID文件所在位置

```java
/var/run/mysqld/mysqld.pid`
```

执行：`ps -ef|grep mysql`可以看到

![image-20210328143345696](C:\Users\Yegenyao\AppData\Roaming\Typora\typora-user-images\image-20210328143345696.png)

其中`--pid-file=指明了mysql启动的pid文件所在位置。

2. MySQL数据文件所在位置:

   ```
   /var/lib/mysql
   ```

查看文件：

![image-20210328143924447](C:\Users\Yegenyao\AppData\Roaming\Typora\typora-user-images\image-20210328143924447.png)



3. MySQL安装目录：

```java
/var/lib/mysql
```

4. MySQL配置目录：

```
/usr/share/mysql
```

5. 命令目录：

```
/usr/bin/
```

6. MySQL配置文件：

```bas
/etc/my.cnf
```

设置编码：

```bash
mysql> show variables like '%char%';
+--------------------------------------+----------------------------+
| Variable_name                        | Value                      |
+--------------------------------------+----------------------------+
| character_set_client                 | utf8                       |
| character_set_connection             | utf8                       |
| character_set_database               | utf8                       |
| character_set_filesystem             | binary                     |
| character_set_results                | utf8                       |
| character_set_server                 | utf8                       |
| character_set_system                 | utf8                       |
| character_sets_dir                   | /usr/share/mysql/charsets/ |
| validate_password_special_char_count | 1                          |
+--------------------------------------+----------------------------+
9 rows in set (0.00 sec)
```

若要修改编码，则修改`/etc/my.cnf`文件，然后重启。

## 二、MySQL逻辑分层

1. **连接层 ：**提供与客户端连接的服务。

2. **服务层：**提供各种用户使用的接口；提供SQL优化器（MySQL Query Optimizer）。
3. **引擎层：** 提供了各种存储数据的方式（InnoDB、MyISAM）。
4. **存储层：**存储数据

查询支持的引擎：

```mysql
show engines;
```

查询默认的引擎：

```mysql
 show variables like '%storage_ngine%';
```

指定数据建库对象的引擎：

```sql
create table tb(

	id int(4) auto_increment,

	name varchar(5),

	dept varchar(5),

	primary key(id)

)ENGINE = MyISAM AUTO_INCREMENT = 1 DEFAULT CHARSET=utf8;
```

## 三、SQL优化

原因：性能低、执行时间太长、等待时间太长、SQL语句欠佳、索引失效、参数配置不正确。

SQL解析过程：

​	join  --> on --> --> where -->group by --> having -->  select  distinct --> order by 

索引：帮助提高sql查询的效率。MySQL中常用索引数据结构有（B+树，Hash树）

索引的弊端：

1. 索引本身需要占用空间，可以放在内存或硬盘上
2. 不是所有情况均使用：a. 少量数据不适用；b. 频繁更新的数据；c. 很少使用的字段；
3. 索引会降低增删改的效率，增删改数据时，需要同步地维护索引文件。

## 四、索引使用

### 索引分类及创建

1. **单列索引**： 针对一个字段的普通索引

```sql
create index dept_index on tb(dept);
```

或：

```mysql
alter table tb add index dept_index(dept);
```

2. **唯一索引：**某个字段值在该列中只能是唯一的

```mysql
create unique index name_index on tb(name);
```

或：

```mysql
alter table tb add unique index name_index(name);
```

3. **复合索引**：多个列构成的索引

```sql
create index dept_name_index on tb(dept,name);
```

### **删除索引，则可以执行如下语句：**

```mysql
drop index dept_index on tb;
```

或:

```mysql
alter table tb drop index dept_index;
```



## SQL explain 命令释义

`explain`可以解释sql的执行过程，如：

![image-20210328212912944](C:\Users\Yegenyao\AppData\Roaming\Typora\typora-user-images\image-20210328212912944.png)

下面来解释下explain命令中各个字段的说明：

`id`：执行命令，id越大，则越先执行；id相同，则在上面的先执行。

`select_type`：查询的类型,主要有以下几种：

- SIMPLE: 简单查询

- PRIMARY : 主查询， 包含子查询的主查询
- SUBQUERY: 子查询。
- DERIVED: 衍生查询（使用到了临时表）
  - 在from 后面查的是一个子查询t1， 则这个子查询t1就是一个衍生表
  - 在from子句中，如果t1 union t2， 则t1则是衍生表。
- UNION RESULT : 表明哪些表存在union

`table`: 表示查询的表名

`type`:索引类型