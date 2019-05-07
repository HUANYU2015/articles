# oracle 高级查询

* 分组查询：求每个部门员工的平均工资
* 多表查询：员工信息、员工部门信息 联合查询
* 子查询：嵌套查询，如 select 语句中嵌套另一个 select 语句

## 分组查询

### 分组函数的概念

分组函数作用域一组数据，并对一组数据返回一个值

### 分组函数的使用

$示例$

``` sql
select [column,] group function(column), ...
   from table
   [where condition]
   [group by column]
   [order column]
;
```

$常用的分组函数$

* avg
* sum
* min
* max
* count 计数函数
* wm_concat 拼接  -> 行转列

$avg 函数和 sum 函数$

* 获取员工的平均工资和工资总额

``` sql
select avg(sal), sum(sal) from emp;
```

$min 函数和 max 函数$

* 获取员工工资最大值与最小值

``` sql
select max(sal), min(sal) from emp;
```

$count 函数$

* 获取员工的总人数

``` sql
select count(*) from emp;
```

* 或者使用 `distinct` 关键字用来去除重复的记录

* 获取员工部门数

``` sql
select count(distinct deptno) from emp;
```

$vm_concat 函数$

* 行转列

``` sql
select deptno, wm_concat(ename) from emp group by deptno;
-- 执行结果
--      10
-- Clark，King，Miller
--     20
-- Smith，Ford，Adams，Scott，Jones
```

![屏幕快照 2019-03-07 20.48.51](/assets/屏幕快照%202019-03-07%2020.48.51.png)

$示例：统计员工的平均工资或平均奖金$

``` sql
-- 1.count(*)得出的是全部列数
select sum(comm) / count(*) ...
-- 2.count(comm)可以自动过滤掉空值，只统计 comm 不为空的行数记录，这也是分组函数都有的特性
select sum(comm) / count(comm)...
-- 3.与2相同
select avg(comm)...
```

> 🦋注意：如果统计逻辑是将全部员工都纳入统计范围，由于分组函数会自动过滤空值，为此为了保证计算逻辑的合理性，需要使用 `nvl 函数`将空值转为0 ，而将空值保留。修改SQL 命令如下

``` sql
select sum(comm) / count(nvl(comm, 0)) from emp;
select avg(nvl(comm, 0)) from emp;
```

### 使用 group by 子句进行数据分组

$单列分组：按部门计算所属员工平均工资$

``` sql
select deptno, avg(sal) from emp group by deptno;
```

语法要求：在 select 列表中所有未包含在分组函数中的列都应该包含在 group by 子句中，示例命令如下。反之则无此限制。

``` sql
select col1, col2, col3, 分组函数(colx)
from table
group by col1, col2, col3, col4;
```

$多列分组：按部门及员工职位计算所属员工工资总额$

``` sql
select deptno, job, sum(sal)
from emp
group by deptno, job
order by deptno;
```

![Xnip2019-03-12_17-30-10](/assets/Xnip2019-03-12_17-30-10.png)

### 使用 having 子句过滤分组结果集

$having 子句的作用：依据限定条件过滤分组结果集$

$语法结构$

``` sql
select column, groub_function
from table
[where condition]
[group by group_by_expression]
[having group_condition]
[order by column];
```

$示例：使用 having 子句实现获取员工平均工资大于2000的部门$

``` sql
select deptno, avg(sal)
from emp
group by deptno
having avg(sal) > 2000;
```

> **where 与 having 的区别**
>
> having 子句中可以使用分组函数（组函数或多行函数），但是在 where 子句中不能出现分组函数（组函数或多行函数）。
>
> **where 和 having 可以通用的情况**
>
> 如查看10号部门的平均工资，既可以使用 where 子句也可以使用 having 子句，但是从 SQL 优化的角度考虑（如果其后又 group 子句，则会降低分组对象的量，从而提升命令执行效率），**`尽量使用 where 子句`**。
> 🦋注意：参照上面的语法结构，**`注意 where 子句及 having 子句相对与 group 子句的书写位置`**。

### 在分组查询中使用 order by 子句

$示例：求每个部门的平均工资，要求显示：部门号，部门的平均工资，并且按照工资的升序排列$

可以按照：列、别名、表达式、序号进行排序

* 按照表达式排序

``` sql
select deptno, avg(sal)
from emp
group by deptno
order by avg(sal);
```

* 按照别名排序

``` sql
select deptno, avg(sal) 平均工资
from emp
group by deptno
order by 平均工资;
```

* 按照序号排序

``` sql
select deptno, avg(sal) 平均工资
from emp
group by deptno
order by 2;
```

> 🦋注意：order by 的项必须是 `select-list` 表达式的数目
>
> order by 子句默认使用升序进行排列，如需降序排列，可在其后添加 `desc` 关键字，或使用 `a` 命令，即就是 `append` 命令，在已经执行的排序语句后附加，代码如下：

``` sql
-- 注意两字符之间至少空两格
a  desc
```

> 其他简洁命令：`ed (edit)`, `/`

### 分组函数的嵌套

$示例：求部门平均工资的最大值$

``` sql
select max(avg(sal))
from emp
group by deptno;
```

### group by 子句的增强

$group by 子句的增强语法$

``` sql
group by rollup(a, b)
```

等价于下面三句

``` sql
group by a, b
group by a
group by null;
```

![1201552536440](/assets/1201552536440.jpg)

$分步执行$

* 按部门、不同职位，统计工资总额

``` sql
select deptno, job, sum(sal)
from emp
group by deptno, job;
```

* 按部门统计工资总额

``` sql
select deptno, sum(sal)
from emp
group by deptno;
```

* 统计工资总额

``` sql
select sum(sal) from emp
```

$一步执行：group by 子句的增强$

``` sql
select deptno, job, sum(sal)
from emp
group by rollup(deptno, job);
```

执行结果如下图

![屏幕快照 2019-03-14 12.16.10](/assets/屏幕快照%202019-03-14%2012.16.10.png)

再次执行下面的代码

``` sql
-- 相同的部门号只显示首行，不同的部门号跳过（间隔）两行
break on deptno skip 2;
```

执行结果如下图

![1211552537475](/assets/1211552537475.jpg)

再次执行下面的代码

``` sql
-- 设置每页显示30条记录（行）
set pagesize 30
```

执行结果如下图

![屏幕快照 2019-03-14 12.27.36](/assets/屏幕快照%202019-03-14%2012.27.36.png)

$最终完整代码$

``` sql
-- 设置每页显示30条记录（行）
set pagesize 30;
-- 相同的部门号只显示首行，不同的部门号跳过（间隔）两行
break on deptno skip 2;

select deptno, job, sum(sal)
from emp
group by rollup(deptno, job);
```

### sql plus 的报表功能

报表包括：标题、页码、别名
report.sql

``` sql
title col 15 '我的报表' col 35 sql.pno
col deptno heading 部门号
col job heading 职位
col sum(sal) heading 工资总额
break on deptno skip 1
```

``` sql
-- 读取 SQL 文件
get d:\temp\report.sql

-- 执行设置，即上面的SQL
@d:\temp\report.sql

-- 执行查询语句
select deptno, job, sum(sal) from emp group by rollup(deptno, job);

-- 设置一页显示的记录数
set pageSize 15;
```

执行结果如下所示：

![屏幕快照 2019-04-01 19.43.48](/assets/屏幕快照%202019-04-01%2019.43.48.png)

> 注意：可以将前面的 report.sql 设置添加在 select 语句中，这样就只在当前查询会话中有效，不具持久性。

## 多表查询

$什么是多表查询$

从多个表中查询数据，这些表使用外键相关联。

$笛卡尔集$

列数相加，行相乘

> 注意：为了避免使用笛卡尔集，可以在 where 条件中洪添加有效的连接条件；在实际运行环境下，应该避免使用笛卡尔全集，因为笛卡尔全集会包含错误记录。
>
> 连接条件的个数至少等于表的个数减一。

$连接$

* 等值连接
* 不等值连接
* 外链接
* 自连接
  
$层次查询$

## 子查询
