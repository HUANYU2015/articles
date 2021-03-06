# 五层模型

应用层-传输层-网络层-数据链路层-物理层

## 物理层

**物理层负责通过相关介质，比如光纤、电缆或者双绞线等介质将两台电脑连接起来，让后通过高低电频传送 0、1 这样的电信号**

## 数据链路层

### 以太网协议

以太网协议规定，一组电信号构成一个数据包，我们把这个数据包称为**帧**，每一个帧由标头（head）和数据（data）两部分构成

**用来指定0、1 数据列的传输规则**

### Mac 地址

连入网络的每一个计算机都会有网卡接口，每一个网卡都会有一个唯一的地址，这个地址就叫做 MAC 地址。计算机之间的数据传送，就是通过 MAC 地址来唯一寻找、传送的

**地址**

### 广播与 ARP 协议

#### 广播📢

在同一个**子网**中，计算机 A 要向计算机 B 发送一个数据包，**这个数据包会包含接收者的 MAC 地址。当发送时，计算机 A 是通过广播的方式发送的**，这时同一个子网中的计算机 C, D 也会收到这个数据包的，然后收到这个数据包的计算机，会把数据包的 MAC 地址取出来，与自身的 MAC 地址对比，如果两者相同，则接受这个数据包，否则就丢弃这个数据包。这种发送方式我们称之为广播,就像我们平时在广场上通过广播的形式呼叫某个人一样

#### ARP 协议

ARP 协议解决计算机 A(数据返送方)获取计算机 B(数据接收方)的 MAC 地址的问题

## 网络层

上面我们有说到**子网**这个关键词，实际上我们所处的网络，是由无数个子网络构成的。**广播的时候，也只有同一个子网里面的计算机能够收到**。

假如没有子网这种划分的话，计算机 A 通过广播的方式发一个数据包给计算机 B , 其他所有计算机也都能收到这个数据包，然后进行对比再舍弃。世界上有那么多它计算机，每一台计算机都能收到其他所有计算机的数据包，那就不得了了。那还不得奔溃。  因此产生了子网这么一个东西。

那么问题来了，我们如何区分哪些 MAC 地址是属于同一个子网的呢？假如是同一个子网，那我们就用广播的形式把数据传送给对方，如果不是同一个子网的，我们就会把数据发给网关，让网关进行转发。

为了解决这个问题，于是，有了 IP 协议。

### IP 协议

**区分两台计算机是否处于同一子网之中**

IP协议，它所定义的地址，我们称之为IP地址。IP协议有两种版本，一种是 IPv4,另一种是 IPv6。不过我们目前大多数用的还是 IPv4，我们现在也只讨论 IPv4 这个版本的协议。

这个 IP 地址由 **32 位**的二进制数组成，我们一般把它分成**4段**的十进制表示，地址范围为0.0.0.0~255.255.255.255。（**`0000000.00000000.00000000.00000000~11111111.11111111.11111111.11111111`**）

每一台想要联网的计算机都会有一个IP地址。这个IP地址被分为两部分，前面一部分代表**网络部分**，后面一部分代表**主机部分**。并且网络部分和主机部分所占用的二进制位数是不固定的。

假如两台计算机的网络部分是一模一样的，我们就说这两台计算机是处于同一个子网中。例如 192.168.43.1 和 192.168.43.2, 假如这两个 IP 地址的网络部分为 24 位，主机部分为 8 位。那么他们的网络部分都为 192.168.43，所以他们处于同一个子网中。

可是问题来了，你怎么知道网络部分是占几位，主机部分又是占几位呢？也就是说，单单从两台计算机的IP地址，我们是无法判断他们的是否处于同一个子网中的。

这就隐身除了另一个关键词----**子网掩码**。子网掩码和 IP 地址一样也是 32 位进制数，**不过他的网络部分规定全部为 1，主机部分规定全部为 0**。也就是说，假如上面那两个IP地址的网络部分为 24 位，主机部分为 8 位的话，那他们的子网掩码都为 11111111.11111111.11111111.00000000，即255.255.255.0。

那有了子网掩码，如何来判端IP地址是否处于同一个子网中呢。显然，**知道了子网掩码，相当于我们知道了网络部分是几位，主机部分是几位**。我们只需要把 IP 地址与它的子网掩码做与(and)运算，然后把各自的结果进行比较就行了，如果比较的结果相同，则代表是同一个子网，否则不是同一个子网。

例如，192.168.43.1和192.168.43.2的子码掩码都为255.255.255.0，把IP与子码掩码相与，可以得到他们都为192.168.43.0，进而他们处于同一个子网中。

### ARP 协议详解

有了上面面IP协议的知识，我们回来讲一下ARP协议。

有了两台计算机的IP地址与子网掩码，我们就可以判断出它们是否处于同一个子网之中了。

假如他们处于同一个子网之中，计算机A要给计算机B发送数据时。我们可以通过ARP协议来得到计算机B的MAC地址。

ARP协议**也是通过广播的形式**给同一个子网中的每台电脑发送一个数据包(当然，**这个数据包会包含接收方的IP地址**)。对方收到这个数据包之后，会取出IP地址与自身的对比，如果相同，则把自己的MAC地址回复给对方，否则就丢弃这个数据包。这样，计算机A就能知道计算机B的MAC地址了。

可能有人会问，知道了MAC地址之后，发送数据是通过广播的形式发送，询问对方的MAC地址也是通过广播的形式来发送，那其他计算机怎么知道你是要传送数据还是要询问MAC地址呢？其实在询问MAC地址的数据包中，在对方的MAC地址这一栏中，填的是一个**特殊的MAC地址**，其他计算机看到这个特殊的MAC地址之后，就能知道广播想干嘛了。

假如两台计算机的IP不是处于同一个子网之中，这个时候，我们就会把数据包发送给**网关**，然后让网关让我们进行转发传送

### DNS服务器

**将域名解析为 IP地址**

这里再说一个问题，我们是如何知道对方计算机的IP地址的呢？这个问题可能有人会觉得很白痴，心想，当然是计算机的操作者来进行输入了。这没错，当我们想要访问某个网站的时候，我们可以输入IP来进行访问，但是我相信绝大多数人是输入一个网址域名的，例如访问百度是输入 www.baidu.com 这个域名。其实当我们输入这个域名时，会有一个叫做DNS服务器的家伙来帮我们解析这个域名，然后返回这个域名对应的IP给我们的。

**因此，网络层的功能就是让我们在茫茫人海中，能够找到另一台计算机在哪里，是否属于同一个子网等。**

## 传输层

通过物理层、数据链路层以及网络层的互相磅柱，我们已经把数据成功从计算机 A传送到计算机 B 了，可是，计算机 B 中有各种各样的**应用程序**，计算机该怎么知道这些数据是给哪些应用程序的呢？

这个时候，**端口（port）**就派上用场了，也就是说，我们在从计算机 A 传数据给计算机 B 的时候，还需要指定一个端口，以供特定的应用程序来接收处理

也就是，**传输层的功能就是建立端口到端口的通信**。相比网络层的功能是**建立主机到主机的通信**。

也就是说，只有有了IP和端口，我们才能**进行准确着通信**。这个时候可能有人会说，我输入IP地址的时候并没有指定一个端口啊。其实呢，对于有些传输协议，已经有**设定了一些默认端口了**。例如http的传输默认端口是80，这些端口信息也会包含在数据包里的。

传输层最常见的两大协议是 **TCP** 协议和 **UDP** 协议，其中 TCP 协议与 UDP 最大的不同就是 **TCP 提供可靠的传输**，而 **UDP 提供的是不可靠传输**。

## 应用层

终于说到应用层了，应用层这一层最接近我们用户了。

虽然我们收到了传输层传来的数据，可是这些传过来的数据五花八门，**有html格式的，有mp4格式的，各种各样**。你确定你能看的懂？

因此我们需要**指定这些数据的格式规则**，收到后才好**解读渲染**。例如我们最常见的 Http 数据包中，就会指定该数据包是什么格式的文件了。

>> 参见书目：《计算机网络：自顶向下》


