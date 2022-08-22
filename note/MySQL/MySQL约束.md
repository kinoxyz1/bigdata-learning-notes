








---

# 一、Docker 安装 MySQL8 
```bash
# 安装运行 docker
$ docker pull mysql:8.0
$ docker run -itd \
    --name mysql8 \
    -v /Users/kino/docker_volume/mysql/conf:/etc/mysql/conf.d \
    -v /Users/kino/docker_volume/mysql/logs:/logs \
    -v /Users/kino/docker_volume/mysql/data:/var/lib/mysql \
    -e MYSQL_ROOT_PASSWORD=123456 \
    mysql:8.0

mysql> create database kinodb;
```


# 二、非空约束
限定某个字段/某列的值不允许为空

## 2.1 建表时创建非空约束
```mysql
-- 创建表时添加非空约束
CREATE TABLE test1(
    id INT, 
    name VARCHAR(200) NOT NULL,
    age INT NOT NULL,
    tel CHAR(11) NOT NULL
);

-- 插入数据
INSERT INTO test1 VALUES(1, NULL, 18, 11223344556);  -- ERROR 1048 (23000): Column 'name' cannot be null
INSERT INTO test1 VALUES(1, 'kino1', NULL, 11223344556);  -- ERROR 1048 (23000): Column 'age' cannot be null
INSERT INTO test1 VALUES(1, 'kino1', 18, NULL);  -- ERROR 1048 (23000): Column 'tel' cannot be null
INSERT INTO test1 VALUES(1, 'kino1', 18, 11223344556);  -- Query OK, 1 row affected (0.02 sec)
```

## 2.2  修改表结构添加非空约束
```mysql
ALTER TABLE test1 MODIFY sex VARCHAR(30) NOT NULL; 
```

## 2.3 删除非空约束
```mysql
ALTER TABLE test1 MODIFY sex int(10) NULL; #去掉not null，相当于修改某个非注解字段，该字段允许为空
或者
ALTER TABLE test1 MODIFY sex int(10); #去掉not null，相当于修改某个非注解字段，该字段允许为空
```

# 三、唯一约束
用来限制某个字段/某列的值不能重复。

MySQL 会给唯一约束的列上默认创建一个唯一索引。

## 3.1 建表时添加唯一约束
```mysql
-- 两种建表语法
create table 表名称(
字段名 数据类型,
字段名 数据类型 unique,
字段名 数据类型 unique key,
字段名 数据类型
);

or

create table 表名称(
字段名 数据类型,
字段名 数据类型,
字段名 数据类型,
[constraint 约束名] unique key(字段名)
);

-- 示例
create table test2(
    id INT NOT NULL,
    name VARCHAR(200),
    password VARCHAR(200),
    CONSTRAINT uk_name_pwd UNIQUE(name, password)
);
insert into test2 values(1, 'kino1', '123456');
insert into test2 values(2, 'kino1', '123456'); -- ERROR 1062 (23000): Duplicate entry 'kino1-123456' for key 'test2.uk_name_pwd'
insert into test2 values(2, 'kino2', '123456'); -- Query OK, 1 row affected (0.03 sec)
```


## 3.2 建表后添加唯一约束
```mysql
-- 字段列表中如果是一个字段，表示该列的值唯一。如果是两个或更多个字段，那么复合唯一，即多个字段的组合是唯一的 
-- 方式1：
ALTER TABLE test2 ADD UNIQUE KEY uk_name_pwd1 (name, password)

#方式2：
ALTER TABLE test2 MODIFY id INT unique;
```

## 3.3 删除唯一约束
```mysql
-- 查看都有哪些约束
SELECT * FROM information_schema.table_constraints WHERE table_name = '表名';

-- 可以通过 show index from 表名; 查询表的索引

-- 删除唯一约束
alter table test2 drop index uk_name_pwd1;
```

# 四、主键约束
用来唯一标识表中的一行记录。

主键约束相当于 非空约束 + 唯一约束 的组合, 主键约束的列不允许重复，也不允许为空。

## 4.1 创建表时添加主键约束
```mysql
create table test3(
    id INT primary key,
    name varchar(200) primary key,
    age int
);
or
create table test3(
    id INT,
    name varchar(200),
    age int,
    constraint pk_id_name primary key (id, name)
);
```
## 4.2 建表后增加主键约束
```mysql
alter table test3 add primary key pk_id_name(id, name);
```

## 4.3 删除主键约束
```mysql
-- 删除主键约束不需要指定约束名称, 因为一个表只能有一个主键，删除主键约束之后，非空约束还存在。
alter table test3 drop primary key;
```


# 五、自增列
某个字段的值自增。

1. 一个表最多只能有一个自增长列
2. 当需要产生唯一标识符或顺序值时，可设置自增长
3. 自增长列约束的列必须是键列（主键列，唯一键列）
4. 自增约束的列的数据类型必须是整数类型
5. 如果自增列指定了 0 和 null，会在当前最大值的基础上自增；如果自增列手动指定了具体值，直接赋值为具体值。

## 5.1 建表时添加自增列
```mysql
-- 两种语法
create table 表名称(
    字段名 数据类型 primary key auto_increment,
    字段名 数据类型 unique key not null,
    字段名 数据类型 unique key,
    字段名 数据类型 not null default 默认值,
);
create table 表名称(
    字段名 数据类型 default 默认值 ,
    字段名 数据类型 unique key auto_increment,
    字段名 数据类型 not null default 默认值,,
    primary key(字段名)
);

-- 示例
create table test4(
    id int primary key auto_increment,
    name varchar(200),
    age int
);

mysql> insert into test4(name, age) values('kino1', 18);
Query OK, 1 row affected (0.03 sec)

mysql> insert into test4(name, age) values('kino2', 19);
Query OK, 1 row affected (0.03 sec)

mysql> select * from test4;
+----+-------+------+
| id | name  | age  |
+----+-------+------+
|  1 | kino1 |   18 |
|  2 | kino2 |   19 |
+----+-------+------+
2 rows in set (0.00 sec)
```

## 5.2 建表后添加自增列
```mysql
-- 示例
create table test5(
      id int primary key,
      name varchar(200),
      age int
);
alter table test5 modify id int auto_increment;
```

## 5.3 删除自增约束
```mysql
-- 删除 auto_increment 关键字相当于删除
alter table test5 modify id int;
```

## 5.4 MySQL8新特新-自增变量的持久化
在 MySQL8 之前，自增主键 AUTO_INCREMENT 的值如果大于 max(primary key)+1, 在 MySQL 重启后，会重置 AUTO_INCREMENT=max(primary key)+1, 这种现象在某些情况下会导致业务主键冲突或者其他难以发现的问题。下面通过案例来对比不同的版本中自增变量是否持久化。

### 5.4.1 在MySQL5.7版本中
```mysql
# docker run -itd \
#     --name kinomysql \
#     -v /home/kino/docker_volume/mysql/conf:/etc/mysql/conf.d \
#     -v /home/kino/docker_volume/mysql/logs:/logs \
#     -v /home/kino/docker_volume/mysql/data:/var/lib/mysql \
#     -e MYSQL_ROOT_PASSWORD=123456 \
#     mysql:5.7

-- 创建表
create table test_inc(id int primary key auto_increment);

-- 写入数据
insert into test_inc values(0),(0),(0),(0);

-- 查询数据
select * from test_inc;
+----+
| id |
+----+
|  1 |
|  2 |
|  3 |
|  4 |
+----+

-- 删除 id 为 4 的记录
delete from test_inc where id = 4;

-- 插入一个空值
insert into test_inc values(0);
    
-- 再次查询
select * from test_inc;
+----+
| id |
+----+
|  1 |
|  2 |
|  3 |
|  5 |
+----+

-- 可以看到这里再次插入自增值的时候并没有再次引用id=4, 而是分配了id=5,再次删除id=5
delete from test_inc where id = 5;

-- 重启数据库, 再次插入自增记录
insert into test_inc values(0);
+----+
| id |
+----+
|  1 |
|  2 |
|  3 |
|  4 |
+----+
```
此时 id=4 被复用了，按照重启前的逻辑，此时分配的应该是id=6。 出现这种情况是因为主键自增没有持久化。在 MySQL5.7 系统中，对于自增主键的分配规则，是由 InnoDB 数据字典内部一个 `计数器` 来决定的，而该计数器只在 `内存中维护`, 并不会持久化到磁盘中。在数据库重启时，该计数器会被初始化。

在 MySQL8 中重试上面的测试步骤,最后的结果为:
```mysql
mysql> select * from test_inc;
+----+
| id |
+----+
|  1 |
|  2 |
|  3 |
|  6 |
+----+
```

# 六、foreign key 约束
限定某个表的某个字段的引用完整性。

比如：员工表的员工所在部门的选择，必须在部门表能找到对应的部分。

## 6.1 建表时添加外键约束
```mysql
-- 主表
create table dept(
    id int primary key auto_increment,
    name varchar(200)
);

-- 从表
create table emp(
    id int primary key auto_increment,
    name varchar(200),
    deptid int, 
    foreign key (deptid) references dept(id) -- 在从表中指定外键约束
); 
```

## 6.2 建表后添加外键约束
```mysql
-- 语法
ALTER TABLE 从表名 ADD [CONSTRAINT 约束名] FOREIGN KEY (从表的字段) REFERENCES 主表名(被引用字段) [on update xx][on delete xx];

-- 实例
alter table emp1 add constraint emp_dept_id foreign key (deptid) references dept(id)
```

## 6.3 约束等级
- Cascade方式 ：在父表上update/delete记录时，同步update/delete掉子表的匹配记录
- Set null方式 ：在父表上update/delete记录时，将子表上匹配记录的列设为null，但是要注意子表的外键列不能为not null
- No action方式 ：如果子表中有匹配的记录，则不允许对父表对应候选键进行update/delete操作
- Restrict方式 ：同no action， 都是立即检查外键约束
- Set default方式 （在可视化工具SQLyog中可能显示空白）：父表有变更时，子表将外键列设置 成一个默认的值，但Innodb不能识别。

如果没有指定等级，就相当于Restrict方式。

对于外键约束，最好是采用: `ON UPDATE CASCADE ON DELETE RESTRICT` 的方式。

### 6.3.1 演示1：on update cascade on delete set null
```mysql
create table dept(
    did int primary key, #部门编号
    dname varchar(50) #部门名称
);
create table emp(
    eid int primary key, #员工编号
    ename varchar(5), #员工姓名
    deptid int, #员工所在的部门
    foreign key (deptid) references dept(did) on update cascade on delete set null
    -- 把修改操作设置为级联修改等级，把删除操作设置为set null等级
);

insert into dept values(1003, '咨询部');
insert into emp values(1,'张三',1001); #在添加这条记录时，要求部门表有1001部门
insert into emp values(2,'李四',1001);
insert into emp values(3,'王五',1002);

#修改主表成功，从表也跟着修改，修改了主表被引用的字段1002为1004，从表的引用字段就跟着修改为1004了
mysql> update dept set did = 1004 where did = 1002;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1 Changed: 1 Warnings: 0
mysql> select * from dept;
+------+--------+
| did | dname |
+------+--------+
| 1001 | 教学部 |
| 1003 | 咨询部 |
| 1004 | 财务部 | #原来是1002，修改为1004
+------+--------+
3 rows in set (0.00 sec)

mysql> select * from emp;
+-----+-------+--------+
| eid | ename | deptid |
+-----+-------+--------+
| 1 | 张三 | 1001 |
| 2 | 李四 | 1001 |
| 3 | 王五 | 1004 | #原来是1002，跟着修改为1004
+-----+-------+--------+
3 rows in set (0.00 sec)

#删除主表的记录成功，从表对应的字段的值被修改为null
mysql> delete from dept where did = 1001;
Query OK, 1 row affected (0.01 sec)
mysql> select * from dept;
+------+--------+
| did | dname | #记录1001部门被删除了
+------+--------+
| 1003 | 咨询部 |
| 1004 | 财务部 |
+------+--------+
2 rows in set (0.00 sec)
mysql> select * from emp;
+-----+-------+--------+
| eid | ename | deptid |
| 3   | 王五   | 1004   |
+-----+-------+--------+
3 rows in set (0.00 sec)
```

### 6.3.2 演示2：on update set null on delete cascade
```mysql
create table dept(
    did int primary key, #部门编号
    dname varchar(50) #部门名称
);
create table emp(
    eid int primary key, #员工编号
    ename varchar(5), #员工姓名
    deptid int, #员工所在的部门
    foreign key (deptid) references dept(did) on update set null on delete cascade
    #把修改操作设置为set null等级，把删除操作设置为级联删除等级
);

insert into dept values(1001,'教学部');
insert into dept values(1002, '财务部');
insert into dept values(1003, '咨询部');
insert into emp values(1,'张三',1001); #在添加这条记录时，要求部门表有1001部门
insert into emp values(2,'李四',1001);
insert into emp values(3,'王五',1002);

mysql> select * from dept;
+------+--------+
| did | dname |
+------+--------+
| 1001 | 教学部 |
| 1002 | 财务部 |
| 1003 | 咨询部 |
+------+--------+
3 rows in set (0.00 sec)
mysql> select * from emp;
+-----+-------+--------+
| eid | ename | deptid |
+-----+-------+--------+
| 1 | 张三 | 1001 |
| 2 | 李四 | 1001 |
| 3 | 王五 | 1002 |
+-----+-------+--------+
3 rows in set (0.00 sec)

#修改主表，从表对应的字段设置为null
mysql> update dept set did = 1004 where did = 1002;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1 Changed: 1 Warnings: 0
mysql> select * from dept;
+------+--------+
| did | dname |
| 1001 | 教学部 |
| 1003 | 咨询部 |
| 1004 | 财务部 | #原来did是1002
+------+--------+
3 rows in set (0.00 sec)
mysql> select * from emp;
+-----+-------+--------+
| eid | ename | deptid |
+-----+-------+--------+
| 1 | 张三 | 1001 |
| 2 | 李四 | 1001 |
| 3 | 王五 | NULL | #原来deptid是1002，因为部门表1002被修改了，1002没有对应的了，就设置为
null
+-----+-------+--------+
3 rows in set (0.00 sec)

#删除主表的记录成功，主表的1001行被删除了，从表相应的记录也被删除了
mysql> delete from dept where did=1001;
Query OK, 1 row affected (0.00 sec)
mysql> select * from dept;
+------+--------+
| did | dname | #部门表中1001部门被删除
+------+--------+
| 1003 | 咨询部 |
| 1004 | 财务部 |
+------+--------+
2 rows in set (0.00 sec)
mysql> select * from emp;
+-----+-------+--------+
| eid | ename | deptid |#原来1001部门的员工也被删除了
+-----+-------+--------+
| 3 | 王五 | NULL |
+-----+-------+--------+
1 row in set (0.00 sec)
```

### 6.3.3 演示：on update cascade on delete cascade
```mysql
create table dept(
    did int primary key, #部门编号
    dname varchar(50) #部门名称
);
create table emp(
    eid int primary key, #员工编号
    ename varchar(5), #员工姓名
    deptid int, #员工所在的部门
    foreign key (deptid) references dept(did) on update cascade on delete cascade
    #把修改操作设置为级联修改等级，把删除操作也设置为级联删除等级
);

insert into dept values(1003, '咨询部');
insert into emp values(1,'张三',1001); #在添加这条记录时，要求部门表有1001部门
insert into emp values(2,'李四',1001);
insert into emp values(3,'王五',1002);

mysql> select * from dept;
+------+--------+
| did | dname |
+------+--------+
| 1001 | 教学部 |
| 1002 | 财务部 |
| 1003 | 咨询部 |
+------+--------+
3 rows in set (0.00 sec)
mysql> select * from emp;
+-----+-------+--------+
| eid | ename | deptid |
+-----+-------+--------+
| 1 | 张三 | 1001 |
| 2 | 李四 | 1001 |
| 3 | 王五 | 1002 |
+-----+-------+--------+
3 rows in set (0.00 sec)

#修改主表，从表对应的字段自动修改
mysql> update dept set did = 1004 where did = 1002;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1 Changed: 1 Warnings: 0
mysql> select * from dept;
+------+--------+
| did | dname |
+------+--------+
| 1001 | 教学部 |
| 1003 | 咨询部 |
| 1004 | 财务部 | #部门1002修改为1004
+------+--------+
3 rows in set (0.00 sec)
mysql> select * from emp;
+-----+-------+--------+
| eid | ename | deptid |
+-----+-------+--------+
| 1 | 张三 | 1001 |
| 2 | 李四 | 1001 |
| 3 | 王五 | 1004 | #级联修改
+-----+-------+--------+
3 rows in set (0.00 sec)

#删除主表的记录成功，主表的1001行被删除了，从表相应的记录也被删除了
mysql> delete from dept where did=1001;
Query OK, 1 row affected (0.00 sec)
+------+--------+
| did | dname | #1001部门被删除了
+------+--------+
| 1003 | 咨询部 |
| 1004 | 财务部 |
+------+--------+

mysql> select * from emp;
+-----+-------+--------+
| eid | ename | deptid | #1001部门的员工也被删除了
+-----+-------+--------+
| 3 | 王五 | 1004 |
+-----+-------+--------+
1 row in set (0.00 sec)
```


## 6.4 删除外键约束
语法
```mysql
(1)第一步先查看约束名和删除外键约束
SELECT * FROM information_schema.table_constraints WHERE table_name = '表名称';#查看某个表的约束名
ALTER TABLE 从表名 DROP FOREIGN KEY 外键约束名;

(2)第二步查看索引名和删除索引。（注意，只能手动删除）
SHOW INDEX FROM 表名称; #查看某个表的索引名
ALTER TABLE 从表名 DROP INDEX 索引名;
```
示例
```mysql
mysql> SELECT * FROM information_schema.table_constraints WHERE table_name = 'emp';
mysql> alter table emp drop foreign key emp_ibfk_1;
Query OK, 0 rows affected (0.02 sec)
Records: 0 Duplicates: 0 Warnings: 0

mysql> show index from emp;
mysql> alter table emp drop index deptid;
Query OK, 0 rows affected (0.01 sec)
Records: 0 Duplicates: 0 Warnings: 0
mysql> show index from emp;
```

# 七、default 约束
给某个字段/某列指定默认值，一旦设置默认值，在插入数据时，如果此字段没有显式赋值，则赋值为默认值。

## 7.1 建表时添加default 约束
```mysql
mysql> create table test6(
    id int primary key,
    email varchar(200) not null,
    gender char default '男',
    tel char(11) not null default ''
);

mysql> desc test6;
+--------+--------------+------+-----+---------+-------+
| Field  | Type         | Null | Key | Default | Extra |
+--------+--------------+------+-----+---------+-------+
| id     | int          | NO   | PRI | NULL    |       |
| email  | varchar(200) | NO   |     | NULL    |       |
| gender | char(1)      | YES  |     |         |       |
| tel    | char(11)     | NO   |     |         |       |
+--------+--------------+------+-----+---------+-------+
4 rows in set (0.02 sec)
```

## 7.2 建表后添加 default 约束
```mysql
-- 如果这个字段原来有非空约束，你还保留非空约束，那么在加默认值约束时，还得保留非空约束，否则非空约束就被删除了。
alter table test6 modify tel char(11) default '00000000';
-- 同理，在给某个字段加非空约束也一样，如果这个字段原来没有默认值约束，你想保留，也要在 modify 语句中保留默认值约束，否则就删除了
alter table test6 modify tel char(11) default '00000000' not null;
```

## 7.3 删除 default 约束
```mysql
alter table 表名称 modify 字段名 数据类型; -- 删除默认值约束，也不保留非空约束
alter table 表名称 modify 字段名 数据类型 not null; -- 删除默认值约束，保留非空约束 
```
