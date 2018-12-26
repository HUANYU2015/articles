## SQL Plus登录账户/密码：
* system/Huanyu221
* hnny/123456
## 创建新用户：
create user ${username} identified by ${passowrd};
## 授予用户权限：
grant privilageName[dba, resources...] to ${username};
## 删除用户及用户下的所有对象：
drop user ${username} cascade; 
## 导出数据：
exp ${username}/${password}@${sid} file=E:/file.dmp owner=${username};
## 导入数据：
* imp ${username}/${password} buffer=64000 file=D:/file.dmp full=y;
* imp ${usernameOut}/${passwordOut} buffer=64000 file=D:/file.dmp from user=${usernameOut} touser=${usernameInput};