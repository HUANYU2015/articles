## 使用nginx设置远程代理激活JRebel
- mac中nginx配置文件位置-/usr/local/etc/nginx
- 远程代理URL-http://idea.lanyus.com:80
- 注意：Mac 本机上 NGINX 安装位置位于 /usr/local/etc/nginx 中，NGINX 命令在 /usr/local/bin/nginx 中，sudo nginx -t 等命令需在后一个目录下执行，即就是 /usr/local/bin/ 目录下。


1. 在nginx目录下的nginx.conf中添加`include vhost/*.conf;`，将vhost目录下以.conf结尾的文件纳入nginx.conf中，
2. 新建vhost文件夹，进入该文件夹后，新建*.conf，并在其中键入以下代码。
```xml
server {
  listen 8888;

  location / {
        proxy_pass http://idea.lanyus.com:80;
  }
}

```
3.打开jrebel激活页面，选择连接connect license service,在两个输入框分别填入
http://127.0.0.1:8888/guid
任意邮箱地址
其中guid，可使用在线生成器生成后附加在远程代理url后面。guid 在线生成器地址https://www.guidgen.com/。
激活后的图像如下所示
![0253a7c107280d4512c3146c2dedaf71.png](evernotecid://7AE89F49-5D9C-4AD3-84E8-076F65637FC8/appyinxiangcom/5470235/ENResource/p25)


Mac 查看端口占用情况
``` bash
lsof -i tcp:port
```

如果已经占用，则查看其 PID
使用
``` bash
kill pid
```
杀死进程