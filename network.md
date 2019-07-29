### 网卡配置
- 修改文件
```
/etc/sysconfig/network-scripts/ifcfg-NIC（network interface card）名称（网卡的配置文件）
DEVICE   定义这张网卡的识别名称
BOOTPROTO    启动这张网卡的方式，
        dhcp（bootp）：代表通过DHCP协议获得IP
        static/none  ：代表固定的IP地址
HWADDR       网卡的mac地址 （物理地址）
ONBOOT       启动network服务时，是否要启动或停止这张网卡
TYPE         网卡的类型  Enternet（以太网）   Bridge（网桥）
USERCTL      是否允许普通用户可以启动或停止这张网卡
IPV6INIT     是否在这张网卡上启用ipv6的功能
PEERDNS      是否允许自动修改/etc/resolv.conf配置文件（记录域名解析服务器的配置文件）
IPADDR       指定网卡的IP地址（ipv4）
NETMASK      子网掩码
PREFIX        以掩码位的方式定义子网掩码
MTU          设置网卡最大传输单元（以太网默认1500）
METRIC        定义默认的路由成本
GATEWAY       设置网络的默认网关
DNS1          主DNS
DNS2          备用DNS
IPV6ADDR      配置ipv6地址
IPV6_DEFAULTGW  配置ipv6网关
/usr/share/doc/initscripts-9.0****/sysconfig.txt

000.000.000.000 32位
0000：0000：0000：0000：0000：0000 128位
目前在用ipv6地址 是2001开头的  
前缀 默认是64
```

- 使用管理命令配置IP
```
ifconfig
语法： ifconfig  参数|选项  网卡名称   动作（短命令）
        -a   显示所有网络设置
            up                   启用网卡    ifup（如果没有配置文件，不好用）
            down                 停用网卡    ifdown（----------------------）
            mtu mun              指定mtu值
            metric mun           指定路由成本
            Hw  class address    修改硬件信息（基本用不上）
            add address/mask     添加一个ipv6地址
            del address/mask     删除一个ipv6地址
            arp/-arp             启用/停用arp功能
            broadcast/-broadcast ---------广播功能 

使用ifconfig配置ip
ifconfig eth4 192.168.1.251 netmask 255.255.255.0 boradcast 192.168.1.255 up
```

### 链路聚合

ethtool NIC名称

bonding（linux 链路聚合模块）
0、1、2、3、4、5、6   七种模式
mode=0  表示负载均衡方式，两块或两块以上的网卡，并且都在工作，收发包模式是rr（一替一个收发包），需要交换机配合使用。
mode=1 表示冗余方式，两块网卡只有一块工作，另一块为备份，如果工作网卡失效，备份网卡马上接替工作。
mode=6 表示负载均衡模式 ，两块或两块以上的网卡，并且都在工作，不需要交换配合，带宽叠加

加载bonding模块
1、临时加载  modprobe bonding

2、永久加载

vim /etc/modprobe.d/bonding.conf

alias bond0() bonding

在虚拟网卡中  ： BONDING_OPTS=“miimon=80 mode=6”
在SLAVE网卡中：MASTER=bond0 SLAVE=yes

miimon 是链路监测的时间间隔，单位毫秒  


ip别名
NICname：alisa
eth0：0（0-255）
操作使用ifconfig
 

### 主机名

linux下可以使用hostname来查看主机名
修改主机名：
- 临时修改
hostname name

- 永久修改主机名
vim /etc/sysconfig/network

域名解析
本机域名解析数据库     /etc/hosts
DNS                    /etc/resolv.conf     nameserver 114.114.114.114


名称服务切换器
/etc/nsswitch.conf 

hosts: files dns

路由
linux查看路由
    route （-n）  |  netstat -r
Destination    目的地址
Gateway        网关
Genmask        掩码
Flags       标识   U=启用  H=目的地址是一台主机  G=网关   C=缓存的路由记录  ！=拒绝的路由记录
Metric       路由成本
Iface        接口（出口网卡）

linux 管理（增、删、改、查）静态路由

route(临时的)
语法：route add|del [-net(指定网络)|-host(指定主机)] ip/mask|ip netmask num gw（指定网关） dev（指定接口） eth*

目的地址  下一跳（地址|出口）


eg：添加到主机的路由

#route add -host 192.168.1.1 dev eth0
#route add -host 172.16.1.1 gw 192.168.1.1

eg：添加到网络的路由

#route add -net 192.168.1.11 netmask 255.255.255.0 dev eth0
#route add -net 192.168.1.11 netmask 255.255.255.0 gw 192.168.1.1
#route add -net 192.168.1.0/24 dev eth0


永久的设置路由

编辑文件
vim  /etc/sysconfig/static-routes   （如果文件不存在，创建一个）

语法：
eg：  any net 192.168.3.0/24 gw 192.168.3.254
      any net 10.255.228.128 netmask 255.255.255.0 gw 192.168.2.3
      any host 192.168.18.1 dev eth0


扩展
静态路由

### 使用route 命令添加
 
- 使用route 命令添加的路由，机器重启或者网卡重启后路由就失效了，方法：
 
//添加到主机的路由
# route add –host 192.168.1.11 dev eth0
# route add –host 192.168.1.12 gw 192.168.1.1
 
//添加到网络的路由
# route add –net 192.168.1.11 netmask 255.255.255.0 dev eth0
# route add –net 192.168.1.11 netmask 255.255.255.0 gw 192.168.1.1
# route add –net 192.168.1.0/24 dev eth1
 
//添加默认网关
# route add default gw 192.168.2.1
//删除路由
# route del –host 192.168.1.11 dev eth0
 
- 还可以使用ip命令来添加、删除路由
ip route add default via 172.16.10.2 dev eth0
ip route add 172.16.1.0/24 via 172.16.10.2 dev eth0
 
格式如下：
ip route
default via（下一跳） gateway dev interface
ip/netmask via  gateway dev interface
 
### 在Linux下设置永久路由的方法：
- 在/etc/rc.local里添加
 方法：
 route add -net 192.168.3.0/24 dev eth0
route add -net 192.168.2.0/24 gw 192.168.2.254
 
- 在/etc/sysconfig/network里添加到末尾
 方法：
GATEWAY=gw-ip
或者
 GATEWAY=gw-dev
 
- /etc/sysconfig/static-routes :
 any net 192.168.3.0/24 gw 192.168.3.254
 any net 10.250.228.128 netmask 255.255.255.192 gw 10.250.228.129
  
如果在rc.local中添加路由会造成NFS无法自动挂载问题，所以使用static-routes的方法是最好的。无论重启系统和service network restart 都会生效。
  解决NFS问题的描述：
 按照linux启动的顺序，rc.local里面的内容是在linux所有服务都启动完毕，最后才被执行的，也就是说，这里面的内容是在netfs之后才被执行的，那也就是说在netfs启动的时候，服务器上的静态路由是没有被添加的，所以netfs挂载不能成功。
 
- 在/etc/sysconfig/network-script/route-interface（route-eth0）下添加路由(每个接口一个文件，如果没有就创建一个，只能添加针对该接口的路由)
格式如下：
network/prefix via gateway dev intf
例如给eth0添加一个默认网关:
vim /etc/sysconfig/network-scripts/route-eth0
#添加如下内容（可以省略dev eth0）
0.0.0.0/0 via 172.16.10.2 dev eth0 
ps：注意这里的掩码是0而不是32，因为这里是网段而不是路由。
 保存退出后，service network restart。（/etc/init.d/network restart）
使用route -n或netstat -r查看路由表。
[root@localhost ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
172.16.10.0     0.0.0.0         255.255.255.0   U     0      0        0 eth0
192.168.122.0   0.0.0.0         255.255.255.0   U     0      0        0 virbr0
169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 eth0
0.0.0.0         172.16.10.2     0.0.0.0         UG    0      0        0 eth0
默认路由已经被添加到路由表里面了。
