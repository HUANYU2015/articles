# docker 中 SQLserver 连接

## 基本步骤

1.打开 docker

2.使用下面的 command 运行SQL server 镜像

``` bash
sudo docker run -e 'ACCEPT_EULA=Y' -e 'SA_PASSWORD=<Huanyu221@>' \
-p 1433:1433 --name sql1 \
-d mcr.microsoft.com/mssql/server:2017-latest
```

或

```bash
sudo docker run -e 'ACCEPT_EULA=Y' -e 'MSSQL_SA_PASSWORD=<YourStrong!Passw0rd>' \
   --name 'sql1' -p 1401:1433 \
   -v sql1data:/var/opt/mssql \
   -d mcr.microsoft.com/mssql/server:2017-latest
```

可选`-v sql1data:/var/opt/mssql`参数创建一个名为的数据卷容器**sql1ddata**。 这用于保存 SQL Server 创建的数据。

参数说明
![Xnip2019-07-08_16-22-41](/assets/Xnip2019-07-08_16-22-41.png)

> 注意：我的 `--name` 参数值为 `sqlserver1`

3.检查运行状态

``` bash
sudo docker ps -a
```

其中 status 一项显示 `up`，则表示 SQL server 启动成功。

## 更改 SA密码

``` bash
sudo docker exec -it sql1 /opt/mssql-tools/bin/sqlcmd \
   -S localhost -U SA -P '<YourStrong!Passw0rd>' \
   -Q 'ALTER LOGIN SA WITH PASSWORD="<YourNewStrong!Passw0rd>"'
```

## 数据库连接

### 内部连接

1.使用 docker exec -it 命令在运行的容器内部启动交互式 Bash Shell。 在下面的示例中，sql1 是在创建容器时由 --name 参数指定的名称

``` bash
sudo docker exec -it sql1 "bash"
```

2.一旦位于容器内部，使用 sqlcmd 进行本地连接。 默认情况下，sqlcmd 不在路径之中，因此需要指定`完整路径`

``` bash
/opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P '<YourNewStrong!Passw0rd>'
/opt/mssql-tools/bin/sqlcmd -S localhost -U SA
```

> 提示：可以省略命令行上提示要输入的密码

3.如果成功，应会显示 sqlcmd 命令提示符：1>

### 外部连接

#### 一般工具简单连接：

``` bash
jdbc:sqlserver://localhost:1433
```

用户名：SA
密码：Huanyu221@

#### 通用方式连接

可以从支持 SQL 连接的任何 Linux、Windows 或 macOS 外部工具连接到 Docker 计算机上的 SQL Server 实例。
以下步骤在容器外使用 sqlcmd 连接在容器中运行的 SQL Server。 这些步骤假定你已在容器外安装了 SQL Server 命令行工具。 该原理同样适用时使用其他工具，但连接的过程是唯一的每个工具。

1.查找承载容器的计算机的 IP 地址。 在 Linux 上，使用 ifconfig 或 ip addr。在 Windows 上，使用 ipconfig。
2.对于此示例中，安装sqlcmd在客户端计算机上的工具。 有关详细信息，请参阅在 Windows 上安装 sqlcmd或在 Linux 上安装 sqlcmd。
3.运行 sqlcmd，指定 IP 地址和映射容器中的端口 1433 的端口。 在此示例中，这是同一个端口，1433，在主机上。 如果主机计算机上指定其他映射的端口，你将在此处使用它。

``` bash
sqlcmd -S <ip_address>,1433 -U SA -P '<YourNewStrong!Passw0rd>'
```

## 删除容器

如果想删除本教程中使用的 SQL Server 容器，请运行以下命令：

```bash
sudo docker stop sql1
sudo docker rm sql1
```

> 注意：停止并永久删除容器会删除容器中的所有 SQL Server 数据。 如果你需要保留数据，请在容器外`创建并复制备份文件`或使用`容器数据暂留技术`。

## 持久化数据库数据

即使您使用了`docker stop` 和`docker start`命令重启了容器，您的SQL server 配置修改以及数据库文件在容器中都是持久化的而不会被改变。然而，如果您使用`docker rm`命令将容器删除（`remove`），包括 SQL server 和您的数据库文件在内的任何处于容器中文件资料都会被删除。下面将讲解如何使用**数据卷（data volumes）**来持久化您的数据库文件，即使关联的容器被删除。

### 挂载主机目录作为数据卷

这种方式是将您主机上的一个目录挂载在容器中作为**数据卷**（The first option is to mount a directory on your host as a data volumn in your container）。为了实现这样的目的，可以使用`docker run` 命令外加`-v <host directory>:/var/opt/mssql` 标签。这允许在容器执行之间恢复数据。

``` bash
docker run -e 'ACCEPT_EULA=Y' -e 'MSSQL_SA_PASSWORD=<YourStrong!Passw0rd>' -p 1433:1433 -v <host directory>:/var/opt/mssql -d mcr.microsoft.com/mssql/server:2017-latest
```

这项技术可以让您从 docker 外的主机上分享和查看文件。

> 注意：
> 
> Host volume mapping for Docker on Mac with the SQL Server on Linux image is not supported at this time.可以使用数据卷容器`data volume containers` 替代。这个限制特定于`/var/opt/mssql`目录。从挂载目录读取数据是可行的（`works fine`）。例如，您可以在 Mac 上使用 `-v` 来挂载一个主机目录，并从存在于主机上的一个 `.bak` 文件恢复备份。

> IMPORTAT
> 
> Host volume mapping for Docker on Mac with the SQL Server on Linux image is not supported at this time. Use data volume containers instead. This restriction is specific to the /var/opt/mssql directory. Reading from a mounted directory works fine. For example, you can mount a host directory using -v on Mac and restore a backup from a .bak file that resides on the host.

### 使用数据卷容器

第二个选择是使用数据卷容器。您可以通过指定一个卷名称（`volumn name`）而不是`-v`参数外加一个主机目录的形式创建一个数据卷容器。下面的示例创建了一个名为 sqlvolume 的可分享的数据卷（`data volume`）。**-v sqlvolume:/var/opt/mssql**

``` bash
docker run -e 'ACCEPT_EULA=Y' -e 'MSSQL_SA_PASSWORD=<YourStrong!Passw0rd>' -p 1433:1433 -v sqlvolume:/var/opt/mssql -d mcr.microsoft.com/mssql/server:2017-latest
```

Even if you stop and remove this container, the data volume persists. You can view it with the `docker volume ls` command.

``` bash
docker volumn ls
```

If you then create another container with the same volume name, the new container uses the same SQL Server data conntained inn the volume.

To remove a data volume containner, use the `docker volumn rm` command.

> ⚠️警告
> 
> If you delete the data volumn container, any SQL Server data in the conntainer is permanenntly deleted.

### 备份和还原

除这些容器技术外，还可使用 SQL Server 标准备份和还原技术。 可通过备份文件来保护数据，或将数据移动至其他 SQL Server 实例。 有关详细信息，请参阅[SQL Server 备份和还原 Linux 上的数据库][1]。

> ⚠️警告
> 
> 如要创建备份，请确保在容器外部创建或复制备份文件。 否则，一旦删除容器，也将随之删除备份文件。

## 使用持久化数据

除了进行数据库备份（`database backups`）以保护您的数据之外，您还可以使用数据卷容器（`data volume containers`）。如果前面您已经使用`-v sql1data:/var/opt/mssql` 参数创建了 **sql1** 容器，那么名为**sql1data**的数据卷容器即使在 sql1 容器被移除后仍然保留`/var/opt/mssql`数据，具体命令详见[基本步骤](#基本步骤)。
以下步骤在完全删除 sql1 容器后，用持久化数据创建一个新容器 sql2。

1.Stop sql1 容器

``` bash
sudo docker stop sql1
```

2.Remove sql1 容器，这并不会删除之前创建的包含持久化数据的 sql1data 数据卷容器。

``` bash
sudo docker rm sql1
```

3.创建一个新的容器 sql2，并且重用 sql1data数据卷容器

``` bash
sudo docker run -e 'ACCEPT_EULA=Y' -e 'MSSQL_SA_PASSWORD=<YourStrong!Passw0rd>' \
   --name 'sql2' -e 'MSSQL_PID=Developer' -p 1401:1433 \
   -v sql1data:/var/opt/mssql -d mcr.microsoft.com/mssql/server:2017-latest
```


[1]:https://docs.microsoft.com/zh-cn/sql/linux/sql-server-linux-backup-and-restore-database?view=sql-server-2017



