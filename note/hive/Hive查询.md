* [一、 基本查询](#%E4%B8%80-%E5%9F%BA%E6%9C%AC%E6%9F%A5%E8%AF%A2)
  * [1\.1 全表和特定列查询](#11-%E5%85%A8%E8%A1%A8%E5%92%8C%E7%89%B9%E5%AE%9A%E5%88%97%E6%9F%A5%E8%AF%A2)
  * [1\.2 列别名](#12-%E5%88%97%E5%88%AB%E5%90%8D)
  * [1\.3 算术运算符](#13-%E7%AE%97%E6%9C%AF%E8%BF%90%E7%AE%97%E7%AC%A6)
  * [1\.4 常用函数](#14-%E5%B8%B8%E7%94%A8%E5%87%BD%E6%95%B0)
  * [1\.5 Limit 语句](#15-limit-%E8%AF%AD%E5%8F%A5)
* [二、Where 语句](#%E4%BA%8Cwhere-%E8%AF%AD%E5%8F%A5)
  * [2\.1 比较运算符](#21-%E6%AF%94%E8%BE%83%E8%BF%90%E7%AE%97%E7%AC%A6)
  * [2\.2 Like 和 Rlike](#22-like-%E5%92%8C-rlike)
  * [2\.3 逻辑运算符（and/or/not）](#23-%E9%80%BB%E8%BE%91%E8%BF%90%E7%AE%97%E7%AC%A6andornot)
* [三、分组](#%E4%B8%89%E5%88%86%E7%BB%84)
  * [3\.1 Group By 语句](#31-group-by-%E8%AF%AD%E5%8F%A5)
  * [3\.2 Having 语句](#32-having-%E8%AF%AD%E5%8F%A5)
* [四、Join语句](#%E5%9B%9Bjoin%E8%AF%AD%E5%8F%A5)
  * [4\.1 等值 Join](#41-%E7%AD%89%E5%80%BC-join)
  * [4\.2 表的别名](#42-%E8%A1%A8%E7%9A%84%E5%88%AB%E5%90%8D)
  * [4\.3 内连接](#43-%E5%86%85%E8%BF%9E%E6%8E%A5)
  * [4\.4 左外连接](#44-%E5%B7%A6%E5%A4%96%E8%BF%9E%E6%8E%A5)
  * [4\.5 右外连接](#45-%E5%8F%B3%E5%A4%96%E8%BF%9E%E6%8E%A5)
  * [4\.6 满外连接](#46-%E6%BB%A1%E5%A4%96%E8%BF%9E%E6%8E%A5)
  * [4\.7 多表查询](#47-%E5%A4%9A%E8%A1%A8%E6%9F%A5%E8%AF%A2)
  * [4\.8 笛卡尔积](#48-%E7%AC%9B%E5%8D%A1%E5%B0%94%E7%A7%AF)
  * [4\.9 连接谓词中不支持or](#49-%E8%BF%9E%E6%8E%A5%E8%B0%93%E8%AF%8D%E4%B8%AD%E4%B8%8D%E6%94%AF%E6%8C%81or)
* [五、 排序](#%E4%BA%94-%E6%8E%92%E5%BA%8F)
  * [5\.1 全局排序](#51-%E5%85%A8%E5%B1%80%E6%8E%92%E5%BA%8F)
  * [5\.2 按照别名排序](#52-%E6%8C%89%E7%85%A7%E5%88%AB%E5%90%8D%E6%8E%92%E5%BA%8F)
  * [5\.3 多个列排序](#53-%E5%A4%9A%E4%B8%AA%E5%88%97%E6%8E%92%E5%BA%8F)
  * [5\.4 每个 MapReduce 内部排序(Sort By)](#54-%E6%AF%8F%E4%B8%AA-mapreduce-%E5%86%85%E9%83%A8%E6%8E%92%E5%BA%8Fsort-by)
  * [5\.5 分区排序(Distribute By)](#55-%E5%88%86%E5%8C%BA%E6%8E%92%E5%BA%8Fdistribute-by)
  * [5\.6 Cluster By](#56-cluster-by)
* [六、分桶及抽样查询](#%E5%85%AD%E5%88%86%E6%A1%B6%E5%8F%8A%E6%8A%BD%E6%A0%B7%E6%9F%A5%E8%AF%A2)
  * [6\.1 分桶表数据存储](#61-%E5%88%86%E6%A1%B6%E8%A1%A8%E6%95%B0%E6%8D%AE%E5%AD%98%E5%82%A8)
  * [6\.2 分桶抽样查询](#62-%E5%88%86%E6%A1%B6%E6%8A%BD%E6%A0%B7%E6%9F%A5%E8%AF%A2)
* [七、其他常用查询函数](#%E4%B8%83%E5%85%B6%E4%BB%96%E5%B8%B8%E7%94%A8%E6%9F%A5%E8%AF%A2%E5%87%BD%E6%95%B0)
  * [7\.1 空字段赋值](#71-%E7%A9%BA%E5%AD%97%E6%AE%B5%E8%B5%8B%E5%80%BC)
  * [7\.2 CASE WHEN](#72-case-when)
  * [7\.3 行转列](#73-%E8%A1%8C%E8%BD%AC%E5%88%97)
  * [7\.4 列转行](#74-%E5%88%97%E8%BD%AC%E8%A1%8C)
  * [7\.5 窗口函数](#75-%E7%AA%97%E5%8F%A3%E5%87%BD%E6%95%B0)
  * [7\.6 Rank](#76-rank)

---

# 一、 基本查询
## 1.1 全表和特定列查询

## 1.2 列别名

## 1.3 算术运算符

## 1.4 常用函数

## 1.5 Limit 语句

# 二、Where 语句
## 2.1 比较运算符

## 2.2 Like 和 Rlike

## 2.3 逻辑运算符（and/or/not）

# 三、分组
## 3.1 Group By 语句

## 3.2 Having 语句

# 四、Join语句
## 4.1 等值 Join

## 4.2 表的别名

## 4.3 内连接

## 4.4 左外连接

## 4.5 右外连接

## 4.6 满外连接

## 4.7 多表查询

## 4.8 笛卡尔积

## 4.9 连接谓词中不支持or

# 五、 排序
## 5.1 全局排序

## 5.2 按照别名排序

## 5.3 多个列排序

## 5.4 每个 MapReduce 内部排序(Sort By)

## 5.5 分区排序(Distribute By)

## 5.6 Cluster By
# 六、分桶及抽样查询
## 6.1 分桶表数据存储
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='red'>**分区针对的是数据的存储路径；分桶针对的是数据文件。**</font>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;分区提供一个隔离数据和优化查询的便利方式。不过，并非所有的数据集都可形成合理的分区，特别是之前所提到过的要确定合适的划分大小这个疑虑。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;分桶是将数据集分解成更容易管理的若干部分的另一个技术。

1. 先创建分桶表，通过直接导入数据文件的方式
① 数据准备
	```java
	1001	ss1
	1002	ss2
	1003	ss3
	1004	ss4
	1005	ss5
	1006	ss6
	1007	ss7
	1008	ss8
	1009	ss9
	1010	ss10
	1011	ss11
	1012	ss12
	1013	ss13
	1014	ss14
	1015	ss15
	1016	ss16
	```
	② 创建分桶表
	```sql
	create table stu_buck(id int, name string)
	clustered by(id) 
	into 4 buckets
	row format delimited fields terminated by '\t';
	```
	③ 查看表结构
	```sql
	hive (default)> desc formatted stu_buck;
	Num Buckets:            4    
	```
	④ 导入数据到分桶表中
	```sql
	hive (default)> load data local inpath '/opt/module/datas/student.txt' into table stu_buck;
	```
	⑤ 查看创建的分桶表中是否分成4个桶
	**发现并没有分成4个桶。是什么原因呢？**

2. 创建分桶表时，数据通过子查询的方式导入
	① 先建一个普通的stu表
	```sql
	create table stu(id int, name string)
	row format delimited fields terminated by '\t';
	```
	② 向普通的stu表中导入数据
	```sql
	load data local inpath '/opt/module/datas/student.txt' into table stu;
	```
	③ 清空stu_buck表中数据
	```sql
	truncate table stu_buck;
	select * from stu_buck;
	```
	④ 导入数据到分桶表，通过子查询的方式
	```sql
	insert into table stu_buck
	select id, name from stu;
	```
	⑤ 发现还是只有一个分桶，如图所示

	⑥ 需要设置一个属性
	```sql
	hive (default)> set hive.enforce.bucketing=true;
	hive (default)> set mapreduce.job.reduces=-1;
	hive (default)> insert into table stu_buck
	select id, name from stu;
	```
	⑦ 查询分桶的数据
	```sql
	hive (default)> select * from stu_buck;
	OK
	stu_buck.id     stu_buck.name
	1004    ss4
	1008    ss8
	1012    ss12
	1016    ss16
	1001    ss1
	1005    ss5
	1009    ss9
	1013    ss13
	1002    ss2
	1006    ss6
	1010    ss10
	1014    ss14
	1003    ss3
	1007    ss7
	1011    ss11
	1015    ss15
	```
## 6.2 分桶抽样查询
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;对于非常大的数据集，有时用户需要使用的是一个具有代表性的查询结果而不是全部结果。Hive可以通过对表进行抽样来满足这个需求。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;查询表stu_buck中的数据。
```sql
hive (default)> select * from stu_buck tablesample(bucket 1 out of 4 on id);
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**注：tablesample是抽样语句，语法：TABLESAMPLE(BUCKET x OUT OF y) 。**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;y必须是table总bucket数的倍数或者因子。hive根据y的大小，决定抽样的比例。例如，table总共分了4份，当y=2时，抽取(4/2=)2个bucket的数据，当y=8时，抽取(4/8=)1/2个bucket的数据。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='red'>x表示从哪个bucket开始抽取，如果需要取多个分区，以后的分区号为当前分区号加上y</font>。例如，table总bucket数为4，tablesample(bucket 1 out of 2)，表示总共抽取（4/2=）2个bucket的数据，抽取第1(x)个和第3(x+y)个bucket的数据。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='red'>注意：x的值必须小于等于y的值，否则</font>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;FAILED: SemanticException [Error 10061]: Numerator should not be bigger than denominator in sample clause for table stu_buck



# 七、其他常用查询函数
## 7.1 空字段赋值
1. 函数说明
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;NVL：给值为NULL的数据赋值，它的格式是NVL( string1, replace_with)。它的功能是如果string1为NULL，则NVL函数返回replace_with的值，否则返回string1的值，如果两个参数都为NULL ，则返回NULL。

2. 数据准备：采用员工表
3.	查询：如果员工的comm为NULL，则用-1代替
	```sql
	hive (default)> select nvl(comm,-1) from emp;
	OK
	_c0
	20.0
	300.0
	500.0
	-1.0
	1400.0
	-1.0
	-1.0
	-1.0
	-1.0
	0.0
	-1.0
	-1.0
	-1.0
	-1.0
	```
4.	查询：如果员工的comm为NULL，则用领导id代替
```sql
hive (default)> select nvl(comm,mgr) from emp;
OK
_c0
20.0
300.0
500.0
7839.0
1400.0
7839.0
7839.0
7566.0
NULL
0.0
7788.0
7698.0
7566.0
```
## 7.2 CASE WHEN
1. 数据准备

	|  name| dept_id  | sex|
	|--|--| --|
	| 悟空 | A | 男
	| 沙僧 | B | 男
	| 八戒 | A | 男
	| 白骨精 | B | 女
	| 蜘蛛精 | B | 女
	| 狐狸精 | A | 女
2. 需求
	求出不同部门各男女各多少人

3. 创建本地 emp_sex.txt ， 导入数据
	```bash
	[kino@hadoop102 datas]$ vim emp_sex.txt
	悟空	A	男
	沙僧	B	男
	八戒	A	男
	白骨精	B	女
	蜘蛛精	B	女
	狐狸精	A	女
	```
4. 创建hive表并导入数据
	```sql
	create table emp_sex(
	name string, 
	dept_id string, 
	sex string) 
	row format delimited fields terminated by "\t";
	load data local inpath '/opt/module/datas/emp_sex.txt' into table emp_sex;
	```
5. 按需求查询数据
	```sql
	select 
	  dept_id,
	  sum(case sex when '男' then 1 else 0 end) male_count,
	  sum(case sex when '女' then 1 else 0 end) female_count
	from 
	  emp_sex
	group by
	  dept_id;
	```
## 7.3 行转列

## 7.4 列转行

## 7.5 窗口函数

## 7.6 Rank



