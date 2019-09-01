# oracle 基础讲解

参考：https://blog.csdn.net/u013412772/article/details/52733050

$用户$

![Xnip2019-07-09_14-41-37](/assets/Xnip2019-07-09_14-41-37.png)

![Xnip2019-07-09_14-41-58](/assets/Xnip2019-07-09_14-41-58.png)

## 角色和权限

每个Oracle用户都有一个名字和口令,并拥有一些由其创建的表、视图和其他资源。Oracle角色（role）就是一组权限（privilege） (或者是每个用户根据其状态和条件所需的访问类型)。用户可以给角色授予或赋予指定的权限，然后将角色赋给相应的用户。一个用户也可以直接给其他用户授 权。

数据库系统权限（Database System Privilege）允许用户执行特定的命令集。例如，CREATE TABLE权限允许用户创建表，GRANT ANY PRIVILEGE 权限允许用户授予任何系统权限。

数据库对象权限（Database Object Privilege）使得用户能够对各个对象进行某些操作。例如DELETE权限允许用户删除表或视图的行，SELECT权限允许用户通过select从 表、视图、序列（sequences）或快照 （snapshots）中查询信息。

$1) 3种标准角色$

oracle为了兼容以前的版本，提供了三种标准的角色（role）：CONNECT、RESOURCE和DBA。

1. CONNECT Role(连接角色)
   1. 临时用户，特别是那些不需要建表的用户，通常只赋予他们CONNECT role。CONNECT是使用Oracle的简单权限，这种权限只有在对其他用户的表有访问权时，包括select、insert、update和delete等，才会变得有意义。拥有CONNECT role的用户还能够创建表、视图、序列（sequence）、簇（cluster）、同义词（synonym ）、会话（session）和与其他数据库的链（link）。

2. RESOURCE Role(资源角色)
   1. 更可靠和正式的数据库用户可以授予RESOURCE role。RESOURCE提供给用户另外的权限以创建他们自己的表、序列、过程（procedure）、触发器（trigger）、索引（index）和簇（cluster）。

3. DBA Role(数据库管理员角色)
   1. DBA role拥有所有的系统权限----包括无限制的空间限额和给其他用户授予各种权限的能力。SYSTEM由DBA用户拥有。下面介绍一些DBA经常使用的典型权限。

A.grant（授权）命令

下面对刚才创建的用户user01授权，命令如下：

``` sql
grant connect, resource to user01;
```

B.revoke（撤消）权限

已授予的权限可以撤消。例如撤消（1）中的授权，命令如下：

``` sql
revoke connect, resource from user01;
```

一个具有DBA角色的用户可以撤消任何别的用户甚至别的DBA的CONNECT、RESOURCE 和DBA的其他权限。当然，这样是很危险的，因此，除非真正需要，DBA权限不应随便授予那些不是很重要的一般用户。

撤消一个用户的所有权限，并不意味着从Oracle中删除了这个用户，也不会破坏用户创建的任何表；只是简单禁止其对这些表的访问。其他要访问这些表的用户可以象以前那样地访问这些表。

$2) 创建角色$

除了前面讲到的三种系统角色—-CONNECT、RESOURCE和DBA，用户还可以在Oracle创建自己的role。用户创建的role可以由 表或系统权限或两者的组合构成。为了创建role，用户必须具有CREATE ROLE系统权限。下面给出一个create role命令的实例：

``` sql
create role STUDENT;
```

这条命令创建了一个名为STUDENT的role。

一旦创建了一个role，用户就可以给他授权。给role授权的grant命令的语法与对对用户的语法相同。在给role授权时，在grant命令的to子句中要使用role的名称，如下所示：

``` sql
grant select on CLASS to STUDENT;
```

现在，拥有STUDENT角色的所有用户都具有对CLASS表的select权限。

$3) 删除角色$

   要删除角色，可以使用drop role命令，如下所示：

``` sql
drop role STUDENT;
```

### 授予用户权限

``` sql
-- 简介版
grant 权限列表 to username;

--综合版
grant privilege [, privilege...] to username [, user| role, public...] [with admin option];

-- 可选项 public            所有用户
-- 可选项 with admin option 使用户同样具有分配权限的权利，可将此权限授予别人
```

### 收回用户权限

``` sql
-- 简洁版
revoke 权限列表 from username;

-- 综合版
revoke {privilege | role} from {user_name | role_name | public}
```

## 角色

一个角色是多个权限的集合

### 系统预定义角色

* connect 连接
* resource 访问资源权限，访问表、序列，不包括create session
* dba 拥有所有权限

### 自定义角色

``` sql
create role 角色名
grant 权限列表|角色列表 to 角色名
```

### 常见的系统权限

|代码|作用描述|
|:----|:----|
|CREATE SESSION|创建会话|
|CREATE SEQUENCE|创建序列|
|CREATE SYNONYM|创建同名对象|
|CREATE TABLE|在用户模式中创建表|
|CREATE ANY TABLE|在任何模式中创建表|
|DROP TABLE|在用户模式中删除表|
|DROP ANY TABLE|在任何模式中删除表|
|CREATE PROCEDURE|创建存储过程|
|EXECUTE ANY PROCEDURE|执行任何模式的存储过程|
|CREATE USER|创建用户|
|DROP USER|删除用户|

## 权限总结

1.使用create user语句创建用户，alter user语句修改用户，其语法大致相同
    
    drop user username [CASCADE] 会删除用户所拥有的所有对象及数据

2.系统权限允许用户在数据库中执行特定的操作，如执行DDL语句。
    
    with admin option 使得该用户具有将自身获得的权限授予其它用户的功能

   但收回系统权限时，不会从其它帐户级联取消曾被授予的相同权限

3.对象权限允许用户对数据库对象执行特定的操作，如执行DML语句。
    
    with grant option 使得该用户具有将自身获得的对象权限授予其它用户的功能

   但收回对象权限时，会从其它帐户级联取消曾被授予的相同权限

4.系统权限与对象权限授予时的语法差异为对象权限使用了ON object_name 子句

5.PUBLIC 为所有的用户

6.ALL：对象权限中的所有对象权限

## 关键 SQL 命令讲解

### inner join、right join 与 inner join

left join(左联接)返回包括左表中的所有记录和右表中联结字段相等的记录
right join(右联结)返回包括右表中的所有记录和左表中联结字段相等的记录
inner join(等值连接)只返回两个表中联结字段相等的行
outer join（共有只展示一次）1表独有的+2 表独有的+两个表共有的（不重复计数）

示例如下图;

![Xnip2019-07-15_17-27-56](/assets/Xnip2019-07-15_17-27-56.png)

![Xnip2019-07-15_17-29-08](/assets/Xnip2019-07-15_17-29-08.png)

![Xnip2019-07-15_17-30-16](/assets/Xnip2019-07-15_17-30-16.png)

https://www.cnblogs.com/pcjim/articles/799302.html

### union 与 union all

为了将两个 select 语句的结果作为一个整体显示出来，就需要用到 union 或者 union all 关键字。两者的区别如下：

union: 对两个结果集进行并集操作，会排除多个结果集中的重复行，同时进行默认规则的排序；
union all: 对两个结果集进行并集操作，包括重复行，不进行排序。

其他，intersect 与 Minus

Intersect: 对两个结果集进行交集操作，会排除多个结果集中的重复行，同时进行默认规则的排序；

Minus: 对两个结构集进行差操作，不包括重复行，同时进行默认规则的排序。

此外，我们没有必要在每一个 select 结果集中使用 order by子句来进行排序，我们可以在最后使用一条order by来对整个结果进行排序。例如：

例如：

``` sql
select employee_id, job_id from employees
union
select employee_id, job_id from job_history
order by 、、、
```

> 注意
>
> union 和 union all 都可以将多个结果集合并，而不仅仅是两个，你可以将多个结果集串起
> 来。 使用union和union all必须保证各个select 集合的结果有**`相同个数的列`**，并
> 且每个`列类型是一样的`。但`列名则不一定需要相同`，oracle会将`第一个`结果的列名作为
> 结果集的列名。

### with as 、、、select、、、


