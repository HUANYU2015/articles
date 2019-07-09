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
docker exec -it aa110cf626d1 /bin/bash
```

其中aa110cf626d1为您的 container ID，可以通过 `docker ps -a` 查看获取。

## oracle 操作

1.登录

``` bash
sqlplus sys as sysdba/oracle
sqlplus system/oracle
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
docker exec -it 7d9ae715b6f4 /bin/bash
# 2.创建项目目录，用以存放引入文件
mkdir /data/ordbfile
# 3.退出 Oracle 容器颞部交互式 bash shell
exit
# 4.复制本地文件至 Oracle 容器对应目录
docker cp /Users/shuyunfeng/Development/Library/docker/oracle/dump/hnny.dmp 7d9ae715b6f4:/data/ordbfile/
# 5.将数据库文件引入对应账户
imp hnny/hnny file=/data/ordbfile/hnny.dmp ignore=y full=y
```
