# oracle 连接

## 步骤

1、登录docker；

``` bash
sudo docker login
```

2、查找docker hub中oracle可用镜像；

``` bash
docker search oracle
```

3、拉取docker镜像到本地。可通过docker image ls查看已拉取到本地的镜像。

``` bash
docker pull sath89/oracle-12c
docker pull alexeiled/docker-oracle-xe-11g
```

阿里云镜像加速：
https://pnw95f56.mirror.aliyuncs.com

4.利用 Oracle 镜像生成 Oracle 容器

``` bash
docker run -h "oraclehost" --name "oracle" -d -p 1521:1521 sath89/oracle-12c
docker run -h "oraclehost" --name "oracle" -d -p 49160:22 -p 49161:1521 -p 49162:8080 alexeiled/docker-oracle-xe-11g
docker run -h "oraclehost" --name "oracle" -d -v /Users/shuyunfeng/Development/Library/docker/oracle/dump:/usr/home -p 49160:22 -p 49161:1521 -p 49162:8080 alexeiled/docker-oracle-xe-11g

```

-h "oracle"：指定容器的hostname为oracle
--name "oracle"：将容器命名为oracle
-d：在后台运行
-p: 端口映射，格式为：主机(宿主)端口:容器端口

5.使用 docker exec -it 命令在运行的容器内部启动交互式 Bash Shell

``` sql
docker exec -it 884939696c52 /bin/bash
```

其中aa110cf626d1为您的 container ID，可以通过 `docker ps -a` 查看获取。

6.关闭或开启容器

``` bash
# 开启容器
docker start container_name
# 关闭容器
docker stop container_name
```

## oracle 操作

1.登录

``` bash
sqlplus sys as sysdba/oracle
sqlplus system/oracle
```

``` sql
root@oraclehost:~# sqlplus system/oracle

SQL*Plus: Release 11.2.0.2.0 Production on Fri Jul 12 11:34:16 2019

Copyright (c) 1982, 2011, Oracle.  All rights reserved.

ERROR:
ORA-28002: the password will expire within 7 days



Connected to:
Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production

SQL> 
```

需要修改密码：

``` sql
alter user system identified by 123456;
```

2.创建用户

``` sql
create user hnny identified by hnny;
```

3.授权用户

``` sql
grant dba to hnny;
```

4.展示当前 Oracle 连接用户

``` sql
show user;
```

5.连接其他用户

``` sql
conn username/password
```

6.检查DBA权限的用户

``` sql
select * from dba_role_privs where granted_role='DBA';
```

7.复制文件到 Oracle 容器并加载文件到Oracle数据库

``` bash
# 1.进入 Oracle 容器内部交互式 bash shell
docker exec -it 884939696c52 /bin/bash
# 2.创建项目目录，用以存放引入文件
mkdir /data/dbsource
# 3.退出 Oracle 容器颞部交互式 bash shell
exit
# 4.复制本地文件至 Oracle 容器对应目录
docker cp /Users/shuyunfeng/Development/Library/docker/oracle/dump/2019-07-10.dmp 884939696c52:/data/dbsource/
# 5.进入 bash shell
docker exec -it 884939696c52 /bin/bash
# 6.将数据库文件引入对应账户
imp hnny/hnny@hnnydd file=/data/dbsource/2019-07-10.dmp buffer=640000 ignore=y full=y
imp hnny/hnny file=/data/dbsource/2019-07-10.dmp buffer=640000 ignore=y full=y
imp hnny/hnny file=/data/dbsource/2019-07-10.dmp ignore=y full=y

imp hnny/hnny@hnnydd fromuser=hnny touser=hnny file=/data/dbsource/2019-07-10.dmp buffer=640000 ignore=y full=y
```

8.删除用户

``` sql
drop user username [cascade];
```

cascade表示级联删除用户的所有对象，删用户时，一起删除该用户的对象

``` sql
select * from dba_objects o where o.owner = 'hnny' and o.object_type = 'FUNCTION';

select text from user_source where name='fn_getlastday';
```

补充学习：
https://juejin.im/entry/58fb250044d9040069dcecad