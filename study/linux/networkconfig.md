# linux 网络配置

## 配置方法选择

1. ifconfig 命令临时配置 IP 地址
2. setup 工具永久配置 IP 地址
3. 修改网络配置文件
4. 图像界面配置 IP 地址

### ifconfig 命令

查看与配置网络状态的命令

``` bash
# 临时设置 eth0 网卡的 IP 地址与子网掩码
ifconfig eth0 192.168.0.200 netmask 255.255.255.0
```

$示例$

``` bash
hadoop@centos-7 ~]$ ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.211.55.8  netmask 255.255.255.0  broadcast 10.211.55.255
        inet6 fdb2:2c26:f4e4:0:d5a2:332c:f355:9ca0  prefixlen 64  scopeid 0x0<global>
        inet6 fe80::d41a:a6de:73c6:70fd  prefixlen 64  scopeid 0x20<link>
        ether `00:1c:42:e3:0f:54(MAC 地址或 HWaddr)`  txqueuelen 1000  (Ethernet)
        RX packets 307  bytes 34950 (34.1 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 238  bytes 31040 (30.3 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536（`无用，只代表本机，无法实现网络连接`）
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 100  bytes 11232 (10.9 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 100  bytes 11232 (10.9 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

virbr0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 192.168.122.1  netmask 255.255.255.0  broadcast 192.168.122.255
        ether 52:54:00:b9:2f:bb  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

### setup Redhat专有命令工具

### 修改网络配置文件

* 网卡信息文件-/etc/sysconfig/network-scripts/ifcfg-eth0
* 主机名文件-/etc/sysconfig/network
* dns 配置文件-/etc/resolv.conf

$查看 eth0 网卡的配置文件 `ifcfg-eth0`$

``` bash
vim /etc/sysconfig/network-scripts/ifcfg-eth0
```

``` bash
# 网卡设备名称
DEVICE=eth0
# 是否自动获取 IP（可选 none、static、dhcp），
# 要想自动获取 IP，需要局域网内有 DHCP 服务器，
BOOTPROTO=dhcp
# Mac 地址
HWADDR=00:1c:42:75:17:eb
# 是否随网络服务启动，eth0 生效，默认为 no，一般需要修改为 yes
ONBOOT=yes
# 网络环境为以太网
TYPE=Ethernet
# 不允许非 root 用户控制此网卡
USERCTL=no
# 注意：如果BOOTPROTO设置为 dhcp,且可以自动获取 IP，则后面的诸项可以不设置，会从 DHCP 服务器上自动获取


# 是否可以由 network manager 图形管理工具托管
# 只有 Linux 是以 GUI 形式安装的才可以
NM_CONTROLLER=no
# 唯一识别码，centos6 之后，复制系统镜像并安装后，需要修改该项，
UUID=ad169432-7822-4c84-beff-b4bd41b3606f
# IP 地址，子网掩码，网关，DNS
IPADDR=10.101.0.144
NETMASK=255.255.252.0
GATEWAY=10.101.0.1
DNS1=172.16.172.82
DNS2=172.16.172.83

IPV4_FAILURE_FATAL=no
# ipv6已经启用
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
DEFROUTE=yes
PROXY_METHOD=none
BROWSER_ONLY=no
NAME=eth0
```

$查看主机名文件`/etc/sysconfig/network`$

``` bash
vim /etc/sysconfig/network
```

``` bash
# Created by anaconda
# 必须为 yes
NETWORKING=yes
# Linux 的主机名可以重复
# 更改后重启 network 无效的，必须要重启系统
# 此外还有临时修改方法，可以使用 **hostname 新的主机名** 进行修改
# 修改完成后，直接使用 **hostname** 命令进行查看
HOSTNAME=hadoop000
```

$dns 配置文件`/etc/resolv.conf`$

``` bash
vim /etc/resolv.conf
```

``` bash
search localdomain shared
# dns 名称服务器
nameserver 10.211.55.1
nameserver fe80::21c:42ff:fe00:18%eth0

```

## centos7 修改 hostname

由于前面通过修改主机名文件`/etc/sysconfig/network`，来修改 hostname 无效，所以有下面的方法。

在CentOS7中，有三种定义的主机名:静态的（static）、瞬态的（transient）、灵活的（pretty）。
“静态”主机名也称为内核主机名，是系统在启动时从/etc/hostname自动初始化的主机名。“瞬态”主机名是在系统运行时临时分配的主机名，例如，通过DHCP或mDNS服务器分配。静态主机名和瞬态主机名都遵从作为互联网域名同样的字符限制规则。而另一方面，“灵活”主机名则允许使用自由形式（包括特殊/空白字符）的主机名，以展示给终端用户。

* 临时修改 transient hostname   `hostname 你的主机名`
* 永久修改 transient hostname   `hostnamectl set-hostname 你的主机名 --transient`
* 永久修改 hostname             通过修改`/etc/hostname` 文件

hostname 查看方式：使用 **`hostname`**（只能查看 transient hostname）或者使用 **`hostnamectl status`**（可以查看所有 hostname 信息及系统信息）

``` bash
[root@Geeklp201 ~]# hostnamectl   #查看一下当前主机名的情况
   Static hostname: Geeklp201
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 77efa27de81d470883b5bb0ed04f468c
           Boot ID: fa62bd1c0f5e4e53a0691fb97971594f
    Virtualization: vmware
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-693.el7.x86_64
      Architecture: x86-64
[root@Geeklp201 ~]# hostnamectl set-hostname geeklp --static
[root@Geeklp201 ~]# hostnamectl status
   Static hostname: geeklp
   Pretty hostname: Geeklp201
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 77efa27de81d470883b5bb0ed04f468c
           Boot ID: fa62bd1c0f5e4e53a0691fb97971594f
    Virtualization: vmware
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-693.el7.x86_64
```

可以使还是用 nmcli 命令查看网络配置信息
networkManager 运行后，可以使用 `nmtui`(与 setup 工具相似) 工具进行网络配置

``` shell
# 使用下面的命令将 networkmanager 服务打开，
sudo systemctl start NetworkManager
```

$网络连接检查命令-ping netstat telnet$

* ping ip/域名 -c 3

``` bash
# 查看开了哪些端口
netstat -tlun 查看本机所开端口
telnet ip 80 查看 IP 主机 80 端口是否打开
telnet ip 远程连接
telnet 命令不安全，系统默认未安装
```

$路由追踪命令$

* traceroute [选项] IP或者域名
选项：
        -n 使用 IP，不使用域名，速度更快

``` bash
# 用来追踪一个网络节点传输本机所经过的所有路由节点，以及节点状态，
traceroute domainname
traceroute -n ip
```

$wget 命令$

* 下载命令

``` bash
wget ip
```

$tcpdump命令$

* tcpdump -i eth0 -nnX port 21
选项：
        -i   指定网卡接口
        -nn  将数据包中的域名与服务器准尉 IP 和端口
        -X   以十六进制和 ASCII 码显示数据包内容
        port 指定监听的端口

## 远程登录

### ssh 协议原理

$对称加密算法$

* 采用单钥匙密码系统的加密方法，同一个钥匙可以同时用作信息的加密和解密，这种加密方法称为对称加密，也成为单钥匙加密。

$非对称加密算法$

* asymmetric cryptographic algorithm 又名“公开钥匙加密算法”，非对称加密算法需要两个钥匙：公开密钥(publickey)和私有密钥(privatekey)

将对方公钥给己方。

SSH-**ssh 安全外壳协议使用的是非对称加密算法**

``` bash
Last login: Sun Apr 28 12:45:31 on ttys001
 ~  ssh yunyue@59.110.172.251
yunyue@59.110.172.251's password:
Last login: Fri Apr 26 17:16:02 2019 from 171.8.148.122

Welcome to Alibaba Cloud Elastic Compute Service !

[yunyue@iz2ze979d4i06ex9osgo9pz ~]$ ls -a
.              .bash_logout   bin     .gitconfig      .npm               .ssh
..             .bash_profile  .gem    .m2             npm-debug.log      .viminfo
.bash_history  .bashrc        .gemrc  .mysql_history  .oracle_jre_usage  .webstorm_nodejs_helpers
[yunyue@iz2ze979d4i06ex9osgo9pz ~]$ cd .ssh
[yunyue@iz2ze979d4i06ex9osgo9pz .ssh]$ ls
known_hosts
[yunyue@iz2ze979d4i06ex9osgo9pz .ssh]$ vim known_hosts
[yunyue@iz2ze979d4i06ex9osgo9pz .ssh]$ cat known_hosts
github.com,52.74.223.119 ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAq2A7hRGmdnm9tUDbO9IDSwBK6TbQa+PXYPCPy6rbTrTtw7PHkccKrpp0yVhp5HdEIcKr6pLlVDBfOLX9QUsyCOV0wzfjIJNlGEYsdlLJizHhbn2mUjvSAHQqZETYP81eFzLQNnPHt4EVVUh7VfDESU84KezmD5QlWpXLmvU31/yMf+Se8xhHTvKSCZIFImWwoG6mbUoWf9nzpIoaSjB+weqqUUmpaaasXVal72J+UX2B+2RPW3RcT0eOzQgqlJL3RKrTJvdsjE3JEAvGq3lGHSZXy28G3skua2SmVi/w4yCE6gbODqnTWlg7+wC604ydGXA8VJiS5ap43JXiUFFAaQ==
[yunyue@iz2ze979d4i06ex9osgo9pz .ssh]$
```

``` bash
#远程管理制定 Linux 服务器
ssh 用户名@ip
# 下载文件
scp [-r] 用户名@ip:文件路径 本地路径
# 上传文件
scp [-r] 本地文件 用户名@ip:上传路径
```

### secureCRT 远程管理工具
