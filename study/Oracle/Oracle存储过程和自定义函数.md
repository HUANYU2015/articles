# Oracle存储过程和自定义函数

## 存储过程

使用 create procedure 命令建立存储过程和存储函数。

语法：

---
create [or replace] procedure 过程名(参数列表)
as
plsql子程序体;

---

> 🦋注意：其中，as 后跟说明部分，相当于 plsql 中的 declare 部分，如无说明部分，**as 也不能省略**。

### 创建无参存储过程

``` sql
-- 第一个存储过程：打印 Hello world
create or replace procedure sayhelloworld
as
-- 说明部分（此处无，但 as 不能省略）
begin
   dbms_output.put_line('hello world');
end;
/
```

### 调用无参存储过程

* 使用 `execute sayhelloworld();` 调用；
* 将存储过程置于其他 plsql 子程序体中，如：

``` sql
begin
   sayhelloworld();
   sayhelloworld();
end;
/
```

### 创建含参存储过程

$为指定的员工，张100元钱的工资；并且打印涨前和涨后的薪水$

``` sql
create or replace procedure raisesalary(eno in number)
as
   psal emp.sal%type
begin
   -- 得到员工原来的工资
   select sal into psal from emp where empno = eno;

   -- 执行给员工张100元的工资
   update emp set sal = sal + 100 where empno = eno;

   -- 注意：考虑事务的单一原则，即谁调用存储过程或存储函数，谁来做提交或回滚，故一般不在存储过程或存储函数中执行 commit 和 rollback（虽然可行）

   -- 打印原始工资和涨后工资
   dbms_output.put_line('涨前工资：' || psal || ', 涨后工资：' || (psal + 100));
end;
/
```

### 调用含参存储过程

``` sql
begin
   raisesalary(7839);
   raisesalary(7866);
   commit;
end;
/
```

$调试存储过程$

不推荐远程调试，

1. 编译以进行调试
2. 通过管理员获取 debug 权限

## 存储函数

* 函数（Function）为一命名的存储程序，可带参数，并返回一计算值
* 存储函数和存储过程的结构类似，但必须有一个 return 子句，用于返回函数值

使用 create FUNCTION 命令建立存储过程和存储函数。

语法：

---

create [or replace] function 函数名(参数列表)
return 函数值类型
as
plsql子程序体;

---

> 🦋注意：其中，as 后跟说明部分，相当于 plsql 中的 declare 部分，如无说明部分，**as 也不能省略**。

## 创建存储函数

$存储函数：查询某位员工的年收入$

```sql
create or replace function queryempincome(eno in number)
return number
as
   -- 定义员工的薪水和奖金
   psal emp.sal%type;
   pcomm emp.comm%type;
begin
   -- 得到员工的月薪和奖金
   select sal, comm into psal, pcomm from emp where empno = eno;
   -- 直接返回员工年收入
   return psal * 12 + nvl(pcomm, 0);
end;
/
```

## 调用存储函数

可以远程调用

``` sql
begin
   queryempincome(7839);
end;
/
```

## in 和 out 参数

$输入参数 or 输出参数$

* 输入参数用 `in` 表明；
* 输出参数用 `out` 表明；
* 输入输出参数用 `in/out` 表明。

* 一般来讲，存储过程和存储函数的区别在与存储函数可以有一个返回值；而存储过程没有返回值
* 存储过程和存储函数都可以通过 `out` 参数指定一个或多个输出参数，我们可以利用 out 参数，在存储过程和存储函数中实现返回多个值，**即就是存储过程可以通过 out 参数来实现返回值**
  
原则：如果只有一个返回值，用存储函数，否则，使用存储过程。

$示例一：查询某个员工的姓名、月薪和职位$

``` sql
-- out 参数：查询某个员工的姓名、月薪和职位
create or replace procedure queryempinformation(eno in number,
                                                pename out varchar2,
                                                psal out number,
                                                pjob out varchar2)
as
begin
   -- 得到该员工的姓名、月薪和职位
   select ename, sal, empjob into pename, psal, pjob from emp where empno = eno;
end;
/
```

执行运行便可以完成调用上面的带 out 参数的存储过程。

$示例二：查询某个员工的所用信息$

> 查询某个员工的所用信息，out 参数过多，如何处理？
> 查询某个部门所有员工的所有信息，能否在 out 参数中返回一个集合？

## 在应用程序中访问存储过程和存储函数

connection -> statement -> PreparedStatement -> CallableStatement

CallableStatement 是用于执行 SQL 存储过程的接口，JDBC API 提供了一个存储过程 SQL 转移语法，该语法允许对所有 RDBMS 使用标准方式调用存储过程，此转移语法有一个包含结果参数的形式和一个不包含结果参数的形式，如果使用结果参数，则必须将其注册为 out 参数，其他参数可用于输入、输出或同时用于二者，参数是根据编号顺序引用的，第一个参数的编号是1。

``` sql
   -- 调用存储函数
   {?= call <procedure-name>[(<arg1>,<arg2>, ...)]}

   -- 调用存储过程
   {call <procedure-name>[(<arg1>,<arg2>, ...)]}
```

### 在应用程序中访问存储过程

$JDBCUtils$

``` java
public class JDBCUtils {
   private static String driver = "oracle.jdbc.OracleDriver";
   private static String driver = "jdbc:oracle:thin:@192.168.56.101.1521:orcl";
   private static String user = "scott";
   private static String password = "tiger";

   // 注册数据库驱动
   static {
      try {
         Class.forName(driver);
         // DriverManager.registerDriver(driver);
      } catch (ClassNotFoundException e) {
         throw new ExceptionInInitializerError(e);
      }
   }

   // 获取数据库连接
   public static Connection getConnection() {
      try {
         return DriverManager.getConnection(url, user, password)
      } catch (SQLException e) {
         e.printStackTrace();
      }
      return null;
   }

   // 用于释放数据库资源
   public static void release(Connection conn, Statement st, ResultSet rs) {
      if (rs != null) {
         try {
            rs.close();
         } catch (SQLException e) {
            e.printStackTrace();
         } finally {
            rs = null;
         }
      }
      if (st != null) {
         try {
            st.close();
         } catch (SQLException e) {
            e.printStackTrace();
         } finally {
            st = null;
         }
      }
      if (conn != null) {
         try {
            conn.close();
         } catch (SQLException e) {
            e.printStackTrace();
         } finally {
            conn = null;
         }
      }
   }
}
```

$调用存储过程$

``` java

import org.junit.Test;
import java.sql.Connection;
import oracle.jdbc.oracleTypes;
import demo.utils.JDBCUtils;

public class TestProcedure {

   // create or replace procedure queryempinformation(eno in number,
                                                // pename out varchar2,
                                                // psal out number,
                                                // pjob out varchar2)
   @Test
   public void testProcedure() {
      // 调用存储过程
      Sting sql = "{call queryempinformation(?, ?, ?, ?)}";
      Connection conn = null;
      CallableStatement call = null;
      try {
         // 得到一个数据库连接
         conn = JDBCUtils.getConnection();
         // 通过连接创建出 statement
         call = conn.prepareCall(sql);

         // 对于 in 参数，赋值
         call.setInt(1, 7839);

         // 对于 out 参数，申明变量类型
         call.registerOutParameter(2, OracleTypes.VARCHAR);
         call.registerOutParameter(3, OracleTypes.NUMBER);
         call.registerOutParameter(4, OracleTypes.VARCHAR);

         // 执行调用
         call.execute();

         // 取出结果
         String name = call.getString(2);
         double sal = call.getDouble(3);
         String job = call.getSting(4);
         System.out.println(name + "\t" + sal + "\t" + job);
      } catch (Exception e) {
         e.printStackTrace();
      } finally {
         JDBCUtils.release(conn, call, null)
      }
   }
}
```

### 在应用程序中访问存储函数

``` java
import org.junit.Test;
import java.sql.Connection;
import oracle.jdbc.oracleTypes;
import demo.utils.JDBCUtils;

public class TestFunction {

   @Test
   public void testFunction() {
      String sql = "{? = call queryempincome(?)}";
      Connection conn = null;
      CallableStatement call = null;
      try {
         // 得到一个数据库连接
         conn = JDBCUtils.getConnection();
         // 通过连接创建出 statement
         call = conn.prepareCall(sql);

         // 对于 out 参数，申明变量类型
         call.registerOutParameter(1, OracleTypes.NUMBER);

         // 对于 in 参数，赋值
         call.setInt(2, 7839);

         // 执行函数调用
         call.execute();

         // 取出年收入
         double income = call.getDouble(1);
         System.out.println("该员工的年收入为" + income);
      } catch (Exception e) {
         e.printStackTrace();
      } finally {
         JDBCUtils.release(conn, call, null);
      }
   }
}
```

## 在 out 参数中使用光标 包的使用

$包头结构$

``` sql
create or replace package mypackage as
   type empcursor is ref cursor;
   procedure queryEmpList(dno in number, empList out empcursor);
end mypackage;
```

$包体结构$

包体中需要实现包头中申明的所有方法

``` sql
create or replace package body mypackage as
   procedure queryEmpList(dno in number, empList out empcursor) as
   begin
      open empList for select * from emp where deptno = dno;
   end queryEmpList;
end mypackage;
```

$示例：查询某个部门的所有员工的全部信息$

包头

``` sql
create or replace package mypackage as
   -- todo enter package declarations(types, exceptions, methods etc) here
   type empcursor is ref cursor;
   procedure queryEmpList(dno in number, empList out empcursor);
   -- 可以再申明多个存储过程或存储函数
end mypackage;
```

包体

``` sql
create or replace package body mypackage as
   procedure queryEmpList(dno in number, empList out empcursor) as
   begin
      -- 注意区别直接在 plsql 中使用光标的方式
      open empList for select * from emp where deptno = dno;
   end queryEmpList;
end mypackage;
```

> 注意区别直接在 plsql 中使用光标的方式

使用 desc 查看表、视图、程序包的结构

![Xnip2019](/assets/Xnip2019.png)

### 在应用中访问包中的存储过程

> 注意：需要带上包名

``` java
import org.junit.Test;
import java.sql.Connection;
import oracle.jdbc.oracleTypes;
import demo.utils.JDBCUtils;

public class TestCursor {
   @Test
   public void testCursor() {
      String sql = "{call MYPACKAGE.queryEmpList(?, ?)}";
      Connection conn = null;
      CallableStatement call = null;
      ResultSet rs = null;
      try {
         // 得到数据库连接
         conn = JDBCUtils.getConnection();
         // 通过连接创建出 statement
         call = conn.prepareCall(sql);

         // 对于 in 参数，赋值，其中10为部门编号
         call.setInt(1, 10);

         // 对于 out 参数，申明变量类型
         call.registerOutParameter(2, OracleTypes.CURSOR);

         // 执行函数调用
         call.execute();

         // 取出年收入
         rs = ((OracleCallableStatement) call).getCursor(2)
         while (rs.next) {
            // 取出该员工的员工号、姓名和工作
            int empno = rs.getInt("empno");
            String name = rs.getString("ename");
            String job = rs.getString("empjob");
            System.out.println(empno + "\t" + name + "\t" + salary + "\t" + job);
         }
      } catch (Exception e) {
         e.printStackTrace();
      } finally {
         JDBCUtils.release(conn, call, null);
      }
   }
}
```