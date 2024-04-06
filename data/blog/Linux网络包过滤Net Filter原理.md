---
tags: [linux, network, 网络, netfilter, 包过滤, nat, snat, dnat, iptables, ipvs, ipset]
date: 2018-08-05
title: Linux网络包过滤Net Filter原理
---

[网络包过滤 net filter](https://netfilter.org/)是Linux网络协议栈中hook的实现，内核协议栈在处理网络包的时候，会回调hook，通过net filter提供的hook实现，可以达到修改包头部信息、包内容等信息。
IPVS、SNAT、DNAT都是通过net filter来实现的，用户态的配置net filter工具有iptables、ipvs、ipset等。

##netfilter钩子（hooks）
在内核协议栈中，有5个跟netfilter有关的钩子，数据包经过每个钩子时，都会检查上面是否注册有函数，如果有的话，就会调用相应的函数处理该数据包，它们的位置见下图：
```
         |
         | Incoming
         ↓
+-------------------+
| NF_IP_PRE_ROUTING |
+-------------------+
         |
         |
         ↓
+------------------+
|                  |         +----------------+
| routing decision |-------->| NF_IP_LOCAL_IN |
|                  |         +----------------+
+------------------+                 |
         |                           |
         |                           ↓
         |                  +-----------------+
         |                  | local processes |
         |                  +-----------------+
         |                           |
         |                           |
         ↓                           ↓
 +---------------+          +-----------------+
 | NF_IP_FORWARD |          | NF_IP_LOCAL_OUT |
 +---------------+          +-----------------+
         |                           |
         |                           |
         ↓                           |
+------------------+                 |
|                  |                 |
| routing decision |<----------------+
|                  |
+------------------+
         |
         |
         ↓
+--------------------+
| NF_IP_POST_ROUTING |
+--------------------+
         |
         | Outgoing
         ↓
```
- NF_IP_PRE_ROUTING: 接收的数据包刚进来，还没有经过路由选择，即还不知道数据包是要发给本机还是其它机器。
- NF_IP_LOCAL_IN: 已经经过路由选择，并且该数据包的目的IP是本机，进入本地数据包处理流程。
- NF_IP_FORWARD: 已经经过路由选择，但该数据包的目的IP不是本机，而是其它机器，进入forward流程。
- NF_IP_LOCAL_OUT: 本地程序要发出去的数据包刚到IP层，还没进行路由选择。
- NF_IP_POST_ROUTING: 本地程序发出去的数据包，或者转发（forward）的数据包已经经过了路由选择，即将交由下层发送出去。

**从上面的流程中，我们还可以看出，不考虑特殊情况的话，一个数据包只会经过下面三个路径中的一个**：
- 本机收到目的IP是本机的数据包: NF_IP_PRE_ROUTING -> NF_IP_LOCAL_IN
- 本机收到目的IP不是本机的数据包: NF_IP_PRE_ROUTING -> NF_IP_FORWARD -> NF_IP_POST_ROUTING
- 本机发出去的数据包: NF_IP_LOCAL_OUT -> NF_IP_POST_ROUTING
```
注意： netfilter所有的钩子（hooks）都是在内核协议栈的IP层，由于IPv4和IPv6用的是不同的IP层代码，所以iptables配置的rules只会影响IPv4的数据包，而IPv6相关的配置需要使用ip6tables。
```
### IPTables中的Tables与Chains
#### Tables
The nat table is used to implement network address translation rules. As packets enter the network stack, rules in this table will determine whether and how to modify the packet's source or destination addresses in order to impact the way that the packet and any response traffic are routed. This is often used to route packets to networks when direct access is not possible.

The Mangle Table
The mangle table is used to alter the IP headers of the packet in various ways. For instance, you can adjust the TTL (Time to Live) value of a packet, either lengthening or shortening the number of valid network hops the packet can sustain. Other IP headers can be altered in similar ways.

This table can also place an internal kernel "mark" on the packet for further processing in other tables and by other networking tools. This mark does not touch the actual packet, but adds the mark to the kernel's representation of the packet.

The Raw Table
The iptables firewall is stateful, meaning that packets are evaluated in regards to their relation to previous packets. The connection tracking features built on top of the netfilter framework allow iptables to view packets as part of an ongoing connection or session instead of as a stream of discrete, unrelated packets. The connection tracking logic is usually applied very soon after the packet hits the network interface.

The raw table has a very narrowly defined function. Its only purpose is to provide a mechanism for marking packets in order to opt-out of connection tracking.

The Security Table
The security table is used to set internal SELinux security context marks on packets, which will affect how SELinux or other systems that can interpret SELinux security contexts handle the packets. These marks can be applied on a per-packet or per-connection basis.
#### Chains
Within each iptables table, rules are further organized within separate "chains". While tables are defined by the general aim of the rules they hold, the built-in chains represent the netfilter hooks which trigger them. Chains basically determine when rules will be evaluated.

As you can see, the names of the built-in chains mirror the names of the netfilter hooks they are associated with:

PREROUTING: Triggered by the NF_IP_PRE_ROUTING hook.
INPUT: Triggered by the NF_IP_LOCAL_IN hook.
FORWARD: Triggered by the NF_IP_FORWARD hook.
OUTPUT: Triggered by the NF_IP_LOCAL_OUT hook.
POSTROUTING: Triggered by the NF_IP_POST_ROUTING hook.
Chains allow the administrator to control where in a packet's delivery path a rule will be evaluated. Since each table has multiple chains, a table's influence can be exerted at multiple points in processing. Because certain types of decisions only make sense at certain points in the network stack, every table will not have a chain registered with each kernel hook.
### Tables与Chains的关系是什么

|Tables↓/Chains→	|PREROUTING	|INPUT	|FORWARD	|OUTPUT	|POSTROUTING|
|-----------------------|-------------------|----------|-----------------|-------------|--------------|
|(routing decision) |    |    |    |				✓	|    |
|raw	                    | ✓|    |    |  			✓	|    |
|(connection tracking enabled)|	✓ |    |    |			✓	|    |
|mangle  |	✓	|  ✓	|  ✓	|  ✓	|  ✓ |
|nat (DNAT)|	✓	|    |    |	✓	|    |
|(routing decision)| 	✓		|    |    |	✓	|    |
|filter		|    |  ✓	  |    ✓  |	✓  |    |	
|security|	 |	✓	  |    ✓  |	✓   |    |	
|nat (SNAT)|    |		✓		|    |    |	✓ |

### Rules
规则是存放在特定表特性chains内的具体执行动作，每条rule包含下面两部分信息：
#### Matching
Matching就是如何匹配一个数据包，匹配条件很多，比如协议类型、源/目的IP、源/目的端口、in/out接口、包头里面的数据以及连接状态等，这些条件可以任意组合从而实现复杂情况下的匹配。
#### Target
Targets就是找到匹配的数据包之后怎么办，常见的有下面几种：
- DROP：直接将数据包丢弃，不再进行后续的处理
- RETURN： 跳出当前chain，该chain里后续的rule不再执行
- QUEUE： 将数据包放入用户空间的队列，供用户空间的程序处理
- ACCEPT： 同意数据包通过，继续执行后续的rule
- 跳转到其它用户自定义的chain继续执行
当然iptables包含的targets很多很多，但并不是每个表都支持所有的targets。

### SNAT转发
#### SNAT
SNAT为源地址转换，通常用于隐藏实际的主机IP地址，常用场景是通过SNAT网关实现多台主机共享Internet访问。
SNAT发包流程为：

![](./_image/2018-08-05-18-25-30.jpg)

- 内网主机访问Internet服务，发送消息报，src: 192.168.1.100, dst: tw.yahoo.com
- 内核IP协议栈通过路由寻址（通常需要将内网主机的默认路由指向NAT网关服务），将IP包通过默认路由发送到NAT网关 192.168.1.2
- NAT网关在POSTROUTING的hook中将源地址修改为NAT网关的地址192.168.1.2
```
通过IPTables配置NAT网关POSTROUTING hook的实例：
iptables -t nat -A POSTROUTING -o eth0 -s subnet -j SNAT --to nat-instance-ip
在上述例子中，subnet为 192.168.1.0/24 ，nat-instance-ip 为 192.168.1.2
```
- 在NAT网关上的内核IP协议再次通过路由表寻址，根据目的地址到路由表中查找源地址为 public ip
- 发送包，并在 /etc/proc/net/nf_conntrack 中记录转发规则，用于收包时匹配使用

SNAT收包流程为：

![](./_image/2018-08-05-18-41-08.jpg)
当public ip接收到Internet服务的回包时，在内核的PREROUTING的hook中（iptables设置进去的？），根据 /etc/proc/net/nf_conntrack 的规则匹配发现需要转换，将目的地址转换为192.168.1.100
下一步的内核协议栈执行routing逻辑，发现目的地址不是本机，且本地打开了转发选项，因此走转发流程，在转发流程中包被转发给192.168.1.100
#### DNAT
DNAT收包流程：

![](./_image/2018-08-05-21-05-07.jpg)
- 外部服务访问public IP的端口，src 61.xx.xx.xx:port, dst: public ip:port
- 内核协议栈收到IP包之后，回调PREROUTING的hook，DNAT在此hook中配置了转发规则，将public ip上的端口转发到192.168.1.210上的端口，内核协议栈执行完此hook后，目的地址变为：192.168.1.210:port2；内核协议栈判断目的地址不是本机，执行转发逻辑，将包转发到目的主机；此时，src 61.xx.xx.xx:port, dst: public ip:port
```
在一些ELB的实现中，采用了Full NAT模式，会在DNAT的同时增加SNAT
```
- 同时内核中的netfilter记录下转发记录到/proc/sys/net/nf_conntrack
- 192.168.1.210接收到消息包之后处理，回包给61.xx.xx.xx:port，回包的流程与SNAT的流程类似

## 参考
[netfilter/iptables简介](https://segmentfault.com/a/1190000009043962)
[A Deep Dive into Iptables and Netfilter Architecture](https://www.digitalocean.com/community/tutorials/a-deep-dive-into-iptables-and-netfilter-architecture)