








```sql
-- 建表
create table student_scores(
id int comment '主键',
studentId int comment 'student 编号',
language int comment '语文',
math int comment '数学',
english int comment '英语',
classId string comment '教室编号',
departmentId string comment '部门编号' 
);
-- 写入数据
insert into table student_scores values 
  (1,111,68,69,90,'class1','department1'),
  (2,112,73,80,96,'class1','department1'),
  (3,113,90,74,75,'class1','department1'),
  (4,114,89,94,93,'class1','department1'),
  (5,115,99,93,89,'class1','department1'),
  (6,121,96,74,79,'class2','department1'),
  (7,122,89,86,85,'class2','department1'),
  (8,123,70,78,61,'class2','department1'),
  (9,124,76,70,76,'class2','department1'),
  (10,211,89,93,60,'class1','department2'),
  (11,212,76,83,75,'class1','department2'),
  (12,213,71,94,90,'class1','department2'),
  (13,214,94,94,66,'class1','department2'),
  (14,215,84,82,73,'class1','department2'),
  (15,216,85,74,93,'class1','department2'),
  (16,221,77,99,61,'class2','department2'),
  (17,222,80,78,96,'class2','department2'),
  (18,223,79,74,96,'class2','department2'),
  (19,224,75,80,78,'class2','department2'),
  (20,225,82,85,63,'class2','department2');
```

# 聚合开窗函数

```bash
### CURRENT ROW: 当前行
### n PRECEDING: 往前 n 行
### n FOLLOWING: 往后 n 行
### UNBOUNDED: 起点 
### UNBOUNDED PRECEDING: 从前面的起点
### UNBOUNDED FOLLOWING: 表示到后面的终点
### LAG(col,n): 往前第n行数据
### LEAD(col,n): 往后第n行数据
### NTILE(n)：把有序分区中的行分发到指定数据的组中，各个组有编号，编号从1开始，对于每一行，NTILE返回此行所属的组的编号。注意：n必须为int类型。
```





## count开窗函数
```sql
select 
  id,
  studentId,
  language,
  math,
  english,
  classId,
  departmentId,
  # 以符合条件的所有行作为窗口
  count(language) over() as count,
  # 以按classId分组的所有行作为窗口
  count(math) over(partition by classId) as count,
  # 以按classId分组、按math排序的所有行作为窗口 TODO
  count(math) over(partition by classid order by math) as count,
  # 以按classId分组、按math排序、按 当前行+往前1行+往后2行的行作为窗口
  count(math) over(partition by classid order by math rows between 1 preceding and 2 following) count1
from student_scores;
```

## sum开窗函数

```sql
# 以符合条件的所有行作为窗口
select 
  id,
  studentId,
  language,
  math,
  english,
  classId,
  departmentId,
  sum(math) over() as sum1,
  # 以按classId分组的所有行作为窗口
  sum(math) over(partition by classid) as sum1,
  # 以按classId分组、按math排序后、按到当前行(含当前行)的所有行作为窗口
  sum(math) over(partition by classid order by id) as sum1,
  # 以按classId分组、按math排序后、按当前行+往前1行+往后2行的行作为窗口
  sum(math) over(partition by classid order by math rows between 1 preceding and 2 following) as sum1
from student_scores;
```

## min开窗函数

```sql
select 
  id,
  studentId,
  language,
  math,
  english,
  classId,
  departmentId,
  # 以符合条件的所有行作为窗口
  min(math) over() as m1,
  # 以按classId分组的所有行作为窗口
  min(math) over(partition by classid) as m2,
  # 以按classId分组、按math排序后、按到当前行(含当前行)的所有行作为窗口
  min(math) over(partition by classid order by math rows between current row and unbounded following) as m3,
  # 以按classId分组、按math排序后、按当前行+往前1行+往后2行的行作为窗口
  min(math) over(partition by classid order by math rows between 1 preceding and 2 following) as m4
from student_scores;
```

## max 开窗函数

```sql
select
  id,
  studentId,
  language,
  math,
  english,
  classId,
  departmentId,
  # 以符合条件的所有行作为窗口
  max(math) over() as m1,
  # 以按classId分组的所有行作为窗口
  max(math) over(partition by classid) as m2,
  # 以按classId分组、按math排序后、按到当前行(含当前行)的所有行作为窗口
  max(math) over(partition by classid order by math rows between current row and unbounded following) as m3,
  # 以按classId分组、按math排序后、按当前行+往前1行+往后2行的行作为窗口
  max(math) over(partition by classid order by math rows between 1 preceding and 2 following) as m4
from student_scores;
```

## avg 开窗函数

```sql
select 
  id,
  studentId,
  language,
  math,
  english,
  classId,
  departmentId,
  # 以符合条件的所有行作为窗口
  avg(math) over() as a1,
  # 以按classId分组的所有行作为窗口
  avg(math) over(partition by classid) as a2,
  # 以按classId分组、按math排序后、按到当前行(含当前行)的所有行作为窗口
  avg(math) over(partition by classid order by math rows between current row and unbounded following) as a3,
  # 以按classId分组、按math排序后、按当前行+往前1行+往后2行的行作为窗口
  avg(math) over(partition by classid order by math rows between 1 preceding and 2 following) as a4
from student_scores;
```

## first_value 开窗函数

```sql
select 
  id,
  studentId,
  language,
  math,
  english,
  classId,
  departmentId,
  # 以符合条件的所有行作为窗口
  first_value(math) over() as f1,
  # 以按classId分组的所有行作为窗口
  first_value(math) over(partition by classid) as f2,
  # 以按classId分组、按math排序后、按到当前行(含当前行)的所有行作为窗口
  first_value(math) over(partition by classid order by math rows between unbounded preceding and current row) as f3,
  # 以按classId分组、按math排序后、按当前行+往前1行+往后2行的行作为窗口
  first_value(math) over(partition by classid order by math rows between 1 preceding and 2 following) as f4
from student_scores;
```

## last_value 开窗函数

```sql
# 以符合条件的所有行作为窗口
# 以按classId分组的所有行作为窗口
# 以按classId分组、按math排序后、按到当前行(含当前行)的所有行作为窗口
# 以按classId分组、按math排序后、按当前行+往前1行+往后2行的行作为窗口
```

## lag 开窗函数

```sql
lag(col, n, default) 用于统计窗口内往上n个值

select 
  id,
  studentId,
  language,
  math,
  english,
  classId,
  departmentId,
  # 窗口内 往上取第二个 取不到时赋默认值60
  lag(math, 2, 60) over(partition by classid order by math) as l1,
  # 窗口内 往上取第二个 取不到时赋默认值NULL
  lag(math, 2) over(partition by classid order by math) as l2
from student_scores;
```

## lead 开窗函数

```sql
# 窗口内 往下取第二个 取不到时赋默认值60
# 窗口内 往下取第二个 取不到时赋默认值NULL
```



```sql
select
  math,
  lag(math, 1) over(order by math asc),
  ## 判断本行和上一行是否相等
  if(math=lag(math, 1, 0) over(order by math asc), "true", "false"),
  ## 本和和上一行的差值
  math-lag(math,1,0) over(order by math asc),
  ## 如果本行为奇数，取上一行，如果本行为偶数，取下一行
  ## if(math%2=0, lag(math, 1, 0) over(order by math asc), lead(math, 1, 0) over(order by math asc)) 
  if(math%2=0, true, false),
  case math%2=0 
    when true then lag(math, 1, 0) over(order by math asc)
    when false then lead(math, 1, 0) over(order by math asc) 
  end
from student_scores
order by math asc;
```











## cume_dist开窗函数

```sql
# 统计小于等于当前分数的人数占总人数的比例
# 统计大于等于当前分数的人数占总人数的比例
# 统计分区内小于等于当前分数的人数占总人数的比例
```

# 排序开窗函数

## rank开窗函数

```sql
# 对全部学生按数学分数排序 
# 对院系 按数学分数排序
# 对每个院系每个班级 按数学分数排序
```

## dense_rank开窗函数

```sql
# 对全部学生按数学分数排序 
# 对院系 按数学分数排序
# 对每个院系每个班级 按数学分数排序
```

## ntile开窗函数

```sql
# 对分区内的数据分成两组
# 对分区内的数据分成三组
```

## row_number开窗函数

```sql
# 对分区departmentid,classid内的数据按math排序
```

## percent_rank开窗函数

```sql

```









