# docker 安装 MySQL

``` bash
docker pull mysql
docker run -h "mysql" --name "mysql" -p 59160:22 -p 59161:3306 -p 59162:8080 -e MYSQL_ROOT_PASSWORD=123456 -d mysql
docker exec -it f541ef907452 /bash
docker exec -it mysql bash
```

``` bash

#登录mysql
mysql -u root -p

# 设置权限（为root分配权限，以便可以远程连接）
grant all PRIVILEGES on *.* to root@'%' WITH GRANT OPTION;

# 由于Mysql5.6以上的版本修改了Password算法，这里需要更新密码算法，便于使用Navicat连接
ALTER user 'root'@'%' IDENTIFIED BY '123456' PASSWORD EXPIRE NEVER;

ALTER user 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';

FLUSH PRIVILEGES;
```

https://www.runoob.com/docker/docker-install-mysql.html
https://www.jb51.net/article/142822.htm
https://blog.csdn.net/xsj34567/article/details/80940238