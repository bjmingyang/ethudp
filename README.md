<pre>
EthUDP
======
<pre>
Ethernet over UDP

1. mode e
Bridge two ethernets using UDP

          |-------Internet---------|
          |                        |
     |    |                        |    |
     |    |IPA                  IPB|    |
 eth1|    |eth0                eth0|    |eth1
+----+----+----+              +----+----+----+
|   server A   |              |   server B   |
+--------------+              +--------------+

Each server connects Internet via interface eth0, IP is IPA & IPB.

On server A, run following command
ip link set eth1 up
ifconfig eth1 mtu 1508
./EthUDP -e IPA 6000 IPB 6000 eth1

On server B, run following command
ip link set eth1 up
ifconfig eth1 mtu 1508
./EthUDP -e IPB 6000 IPA 6000 eth1

will bridge eth1 of two hosts via internet using UDP port 6000

how it works:
* open raw socket for eth1
* open udp socket to remote host
* read packet from raw socket, send to udp socket
* read packet from udp socket, send to raw socket

2. mode i
create a tap tunnel interface using UDP

       |------------Internet--------------|
       |                                  |
       |                                  |
       |IPA                            IPB|
       |eth0                          eth0|
+------+-------+                  +-------+------+
|   server A   +--IP1--------IP2--+   server B   |
+--------------+                  +--------------+

Each server connects Internet via interface eth0, IP is IPA & IPB.

On server A, run following command
./EthUDP -i IPA 6000 IPB 6000 IP1 masklen

On server B, run following command
./EthUDP -i IPB 6000 IPA 6000 IP2 masklen

will create a tap tunnel interface and setup IP1/masklen IP2/masklen via internet using UDP port 6000

how it works:
* open tap raw socket, setip addr
* open udp socket to remote host
* read packet from raw socket, send to udp socket
* read packet from udp socket, send to raw socket

Note:
1. support 802.1Q VLAN frame transport
2. support automatic tcp mss fix
3. if your NIC support GRO, you should disable it by
ethtool -K eth1 gro off
4. support connection from NATed server
If server B connect from NAT IP, please run
./EthUDP -e -p password IPA 6000 0.0.0.0 0 eth1 in A
./EthUDP -e -p password IPB 6000 IPA 6000 eth1 in B


常用模式：
某Linux服务器B，对外有NAT，因此无法直接从外网访问或管理。

借助某台有公网IP的Linux服务器A，利用UDP数据包建立一个隧道接口，只要B和A能使用UDP通信，就可以从A上直接登录B。

具体做法为：
1. 假定A的公网IP是 IPA，通信使用UDP 6000端口，新建的隧道接口A的IP是 ipa/24，密码为password，A上运行
./EthUDP -i -p password IPA 6000 0.0.0.0 0 ipa 24
2. 假定B的隧道接口是ipb/24，B上运行
./EthUDP -i -p password 0.0.0.0 0 IPA 6000 ipb 24
此后，A和B可以通过 ipa/ipb 互相通信

从公网仅仅能看到B与A的6000端口之间有UDP通信

注意：密码和所有数据包均明文传输
</pre>
