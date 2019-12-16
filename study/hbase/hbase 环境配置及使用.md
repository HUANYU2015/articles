# hbase 环境配置及使用

## HBASE 单机安装及配置

由于 HBASE 依赖 Hadoop，他配套发布了 Hadoop jar文件，位于 HBASE 的lib 目录下。该 jar 包仅仅适用于 HBASE 的独立模式。所以该模式下，可以暂且先不安装 Hadoop。

$安装步骤$

``` sh
tar -zxvf hbase-0.90.4.tar.gz
cd hbase-0.90.4
```

$设置相关配置文件$

* **hbase-site.xml**

    通过在该文件中配置 `hbase.rootdir` ，来设定 HBASE 写入数据的目录位置

    ``` xml
    <!--  单机配置  -->
    <?xml version="1.0"?>  
    <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>  
    <configuration>
        <property>  
            <name>hbase.rootdir</name>  
            <value>file:///DIRECTORY/hbase</value>  
        </property>  
     </configuration>  
    ```

    将 DIRECTORY 替换成你期望写文件的目录. 默认 hbase.rootdir 是指向 /tmp/hbase-${user.name} ，也就说你会在重启后丢失数据(重启的时候操作系统会清理/tmp目录)

$启动 hbase$

``` sh
./bin/start-hbase.sh
```

现在你运行的是单机模式的Hbaes。所以的服务都运行在一个JVM上，包括Hbase和Zookeeper。Hbase的日志放在logs目录,当你启动出问题的时候，可以检查这个日志。

$使用 shell 连接 hbase$

``` sh
./bin/start-hbase.sh
```

现在你运行的是单机模式的Hbaes。所以的服务都运行在一个JVM上，包括Hbase和Zookeeper。Hbase的日志放在logs目录,当你启动出问题的时候，可以检查这个日志。

输入 help 然后 <RETURN> 可以看到一列shell命令。这里的帮助很详细，要注意的是表名，行和列需要加引号。

$创建表$

创建一个名为 test 的表，这个表只有一个column family 为 cf。可以列出所有的表来检查创建情况，然后插入些值。

``` sql
$ ./bin/hbase shell
HBase Shell; enter 'help<RETURN>' for list of supported commands.
Type "exit<RETURN>" to leave the HBase Shell
Version: 0.90.0, r1001068, Fri Sep 24 13:55:42 PDT 2010

hbase(main):001:0>

hbase(main):003:0> create 'test', 'cf'
0 row(s) in 1.2200 seconds
hbase(main):003:0> list 'table'
test
1 row(s) in 0.0550 seconds
hbase(main):004:0> put 'test', 'row1', 'cf:a', 'value1'
0 row(s) in 0.0560 seconds
hbase(main):005:0> put 'test', 'row2', 'cf:b', 'value2'
0 row(s) in 0.0370 seconds
hbase(main):006:0> put 'test', 'row3', 'cf:c', 'value3'
0 row(s) in 0.0450 seconds
```

以上我们分别插入了3行。第一个行Row-key为row1, 列为 cf:a， 值是 value1。

$检查插入情况$

Scan这个表，操作如下

``` sql
hbase(main):007:0> scan 'test'
ROW        COLUMN+CELL
row1       column=cf:a, timestamp=1288380727188, value=value1
row2       column=cf:b, timestamp=1288380738440, value=value2
row3       column=cf:c, timestamp=1288380747365, value=value3
3 row(s) in 0.0590 seconds
```

Get一行，操作如下

``` sql
hbase(main):008:0> get 'test', 'row1'
COLUMN      CELL
cf:a        timestamp=1288380727188, value=value1
1 row(s) in 0.0400 seconds
disable 再 drop 这张表，可以清除你刚刚的操作

hbase(main):012:0> disable 'test'
0 row(s) in 1.0930 seconds
hbase(main):013:0> drop 'test'
0 row(s) in 0.0770 seconds
```

关闭shell

``` sql
hbase(main):014:0> exit
```

