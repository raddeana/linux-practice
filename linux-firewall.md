### linux防火墙
linux防火墙是linux内核的一个功能，iptables（管理工具）
结构：四个表五个链
#### 4 表
- raw表 连接跟踪表 不常用，跟踪记录连接情况 PREROUTING  OUPUT
- mangle表 矫正表 不常用，策略路由、QOS限速使用的 5个链都有
- nat表 地址转换表 比较常用，SNAT DNAT PREROUTING POSTROUTING OUTPUT
- filter表 过滤表 经常用 过滤数据包使用的 INPUT OUTPUT FORWARD

#### 5链
- INPUT：接收，发送方不一定是你，但接收方一定是你
- OUTPUT：发送，接收方不一定是你，但发送方一定是你
- FORWARD：转发，接收及发送方全都不是你
- PREROUTING：路由前时，指代在决定去向前处理
- POSTROUTING：路由后时，指代在决定去向后处理

#### 表的优先级
raw -> mangle -> nat -> filter


#### 数据包走向：
- 入站
  - RAW：PREROUTING -> MANGLE：PREROUTING -> NAT：PREROUTING -> 路由选择 -> MALGLE：INPUT -> FILTER：INPUT
- 出站
  - 路由选择 -> RAW：OUPUT -> MALGLE：OUTPUT -> NET：OUTPUT -> FILTER：OUTPUT -> MANGLE：POSTROUTING -> NET：POSTROUTING
- 过路包
  - RAW：PREROUTING -> MANGLE：PREROUTING -> NAT：PREROUTING -> 路由选择 -> MANGLE：FORWARD -> FILTER :FORWARD -> MANGLE：POSTROUTING -> NET：POSTROUTING


iptables
```
语法： 
  iptables -t（指定表名） 表名 命令 链命 [n]  过滤规则 -j（指定动作） 动作
  -t：table，指定表，filter nat raw 。。。。。如果未指定，*****默认是filter

命令：
  -A：append，追加一条规则，添加为规则列表中最后一条规则
  -D：删除一条规则，后面跟数字n表示删除的规则为第n条规则
  -I：insert ，插入一条规则，后面跟数字n表示插入的规则为第n条规则
  -L：列出表中的规则，******默认是filter表 --list 配合-t使用，指定查看的表
  -N：创建自定义链（一般很少用）
  -P： --policy：定义默认策略
      iptables -t 表 -P （链） [ACCEPT DROP]
  -F：--flush：删除表中所有规则
  -X：删除自定义链
  -R：--replace 替换规则，结合n使用
  n：一个数字，常配合 -I -D -R 使用    
  -n：不进行IP与hostname的反解。将服务使用的端口号来显示
  -v：列出详细信息，包括通过该规则的数据包的总数、计数器
  --line-number：显示行号

  动作：用于指定能够匹配规则的数据包要执行的动作，处理动作有：ACCEPT、REJECT、DROP、MASQUERADE、LOG、DNAT、SNAT
  ACCEPT：（接受）放行符合规则的数据包，进行完此处理动作后将不再匹配其他规则，直接跳到下一个规则链
  REJECT：（拒绝，告诉来源）拒绝符合规则的数据包，并传送封包给对方，可以传送的封包有几个选择：icmp port-unreachable、icmp echo-reply或是tcp-reset（这个封包会要求对方关闭连接），进行完此处理动作后，将不再匹配其他规则，直接中断过滤程序
  DROP：（丢弃）丢弃符合规则的数据包，进行完此动作后将不再匹配其他规则，直接中断过滤程序
  LOG：将匹配的数据包的信息记录到/var/log中，详细存放位置请参考/etc/rsyslog.conf中的设置。进行完此动作后，继续匹配其他规则
  DNAT：目标地址转换，用在PREROUTING链
  SNAT:源地址转换，用在POSTROUTING链  --to-source 指定IP    
  MASQUERADE：伪装，用在源地址转换即POSTROUTING链


  过滤规则（网卡、协议、源地址、目的地址、源端口、目的端口）

  -i|o 指定网络接口（interface） 设置数据包出入口
      -i：数据包入接口，需要与INPUT链配合
      -o：数据包出接口，需要与OUPUT链配合
  -p 协议：设置此规则使用的协议 tcp、udp、icmp（ping）、udplite、esp、ah  or   all

  -s ：来源IP、网段  192.168.1.1/32   192.168.1.0/24    !表示取反 ！-s 192.168.1.1/32  代表除了1.1之外所有  
  -d：目的IP、网段

  --sport 源端口      要使用必须有-p 并且在-p后面
  --dport 目的端口
```

#### 规则顺序的重要性
- 自上而下逐条规则依次匹配
- 一旦匹配成功就不再继续往下匹配(LOG动作除外)
- 如果都匹配不上，遵循默认规则

#### 清除规则
- 语法：iptables [-t table] [-FXZ]
- -F：清除所有已制定的规则
- -X：清除用户“自定义”的链
- -Z：将所有的链的计数器与流量统计归零
- 注意：上述三个命令会将本机防火墙的所有规则清除，但不会改变默认策略（policy）！


#### iptables
服务管理 /etc/init.d/iptables start/stop/restart......
服务自启动  chkconfig  iptables on/off  

#### ip6tables
save 执行完保存之后，规则保存在/etc/sysconfig/iptables

#### 高级匹配规则（基于数据包状态来判断）
```
-m 指定高级参数
  state 数据包的状态
      --state 指定匹配数据包状态（NEW ESTABLISHED INVALID  RELATED）
      NEW：当tcp连接时 发送的第一个包  （SYN位被设置为1）状态就是NEW
      ESTABLISHED：传输数据的状态
      INVALID：无效的数据包
      RELATED：表示该数据包属于某个已经建立的链接所建立的新链接，eg ftp-data链接必定是源自某个ftp链接
例子：iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
     iptables -A INPUT -p tcp -m state --state NEW --dport 22 -j ACCEPT
icmp icmp数据包
     --icmp-type  icmp数据包类型
     echo-request 请求
     echo-reply   应答
例子：iptables -A INPUT -p icmp -m icmp --icmp-type echo-request -j DROP
     multiport 多端口
        --sport 源
        --dport 目的

例子：iptables -A INPUT -p tcp -s 192.168.20.3 -m multiport --sport 111,222,3333,4444,50000:50010 -j DROP
     iprange 指定IP范围
        --src-range  源IP范围
        --dst-range  目的IP范围

例子：iptables -A INPUT -m iprange --src-range 192.168.20.1-192.168.20.249 -j DROP
     connlimit 限定连接数  （established的数量）
        --connlimit-above n 限定链接数量

例子：iptables -A INPUT -s 192.168.20.250 -m connlimit --connlimit-above 3 -j REJECT（限制同时最多3个链接）
     limit 限定网速
        --limit （指定一秒通过的数据包数量）
    eg：-m limit --limit 100/second  MTU 1500    150000B/s  146.48Kb/换成带宽  1171.84K 

例子：iptables -A OUTPUT -s 192.168.1.0/24 -m limit --limit 100/second -j ACCEPT
     iptables -A OUTPUT -s 192.168.1.0/24 -j DROP  

     --limit-burst (计数器)

例子：iptables -A OUPUT -s 192.168.1.0/24 -m limit --limit 100/second --limit-burst 300 -j ACCEPT
     第一个包到第300不限速通过，第301包按照100包每秒的规则通过
  iptables -A OUTPUT -s 192.168.1.0/24 -j DROP

  mac 网卡的物理地址
      --mac-source 匹配源mac地址（只能用在进来的包）
例子： iptables -A INPUT -m mac --mac-source aa:bb:cc:dd:ff:gg
tos 匹配服务类型
    --tos
    0x10  最小延迟的
    0x08  最大吞吐量
    0x04  可靠性要求
    0x02  最小代价
    0x00  一般服务
例子： iptables -A INPUT -p tcp -m tos --tos 0x10 -j DROP
ttl 匹配数据包的生存时间
--ttl
例子：iptables -A INPUT -p tcp -m ttl --ttl 69 -j DROP

iptables -A INPUT -p icmp -j ACCEPT 
iptables -A INPUT -i lo -j ACCEPT 
iptables -A INPUT -p tcp   -s 192.168.180.0/24 --dport 22 -j ACCEPT 
iptables -A INPUT -p tcp  --dport 46846 -j ACCEPT 
iptables -A INPUT -p tcp  -s 192.168.180.0/24 --dport 80 -j ACCEPT 
```

#### 实例：
```
# Generated by iptables-save v1.4.7 on Fri Mar 31 18:30:24 2017
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [42:3886]

-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp -s 192.168.180.0/24 --dport 22 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 46846 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp -s 192.168.180.0/24 --dport 80 -j ACCEPT
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -i bond0 -o eth4 -j ACCEPT
-A FORWARD -i eth4 -o bond0 -j ACCEPT
-A FORWARD -j REJECT --reject-with icmp-host-prohibited 
COMMIT
# Completed on Fri Mar 31 18:30:24 2017
# Generated by iptables-save v1.4.7 on Fri Mar 31 18:30:24 2017
*nat
:PREROUTING ACCEPT [99:7667]
:POSTROUTING ACCEPT [1:108]
:OUTPUT ACCEPT [1:108]
-A POSTROUTING -s 192.168.180.0/24 -o eth4 -j SNAT --to-source 223.223.184.131 
COMMIT
# Completed on Fri Mar 31 18:30:24 2017
```
