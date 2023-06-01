---
title: Solving asymmetric routing issues
date: 2023-06-01 14:15:00 +07:00
tags: [aws]
description: A blog about solving asymmetric routing issues
---

We sometimes come across a scenario where a non Amazon Linux EC2 instance is configured with 2 ENIs from the same subnet which are referring to the same route table. Since each ENI gets a private IP address assigned from the same network, this leads to a very common asymmetric routing issue.

As an experiment, a non Amazon Linux EC2 instance is launched in a subnet. A new ENI is then created in the same subnet in which the EC2 is running. Once the new ENI is created and the instance is up, test ping connectivity from another EC2 instance from the same VPC. 

#### Note: Ensure that ICMP inbound rule is allowed in the security group associated with the non Amazon Linux EC2 instance.

#### Observation #1:

Ping succeeds as the instance only has 1 ENI attached.
```
sh-4.2$ ping 172.31.15.216
PING 172.31.15.216 (172.31.15.216) 56(84) bytes of data.
64 bytes from 172.31.15.216: icmp_seq=1 ttl=64 time=1.63 ms
64 bytes from 172.31.15.216: icmp_seq=2 ttl=64 time=1.24 ms
```
Now, attach the newly created ENI to the instance and test ping connectivity to the secondary IP address (private IP address of the new ENI) from the other EC2 instance to the non Amazon Linux EC2 instance.

#### Observation #2:
```
sh-4.2$ ping 172.31.0.218
PING 172.31.0.218 (172.31.0.218) 56(84) bytes of data.
```
As shown in the above output, ping connectivity to the secondary IP address failed due to asymmetric routing issues. This issue occurs when 2 ENIs are in the same subnet referring to the same route table. In Asymmetric Routing, a packet traverses from a source to a destination in one path and takes a different path when it returns to the source. To avoid this asymmetric routing issue, some additional configuration is required at the OS level.

First, log into the non Amazon Linux instance via its public ip address and note down the eth0 and eth1 IP configuration. This will come handy in the following steps. The command route -n will show the default gateway IP address which will be the same for both the ENIs in this case.

```
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 9001
        inet 172.31.15.216  netmask 255.255.240.0  broadcast 172.31.15.255

eth1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 9001
        inet 172.31.0.218  netmask 255.255.240.0  broadcast 172.31.15.255
```

```
route -n
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         172.31.0.1      0.0.0.0         UG    100    0        0 eth0
0.0.0.0         172.31.0.1      0.0.0.0         UG    101    0        0 eth1
```
Next, run the following ip rule and route commands (with the root user) for each interface ensuring the correct matching between the interface name and the corresponding IP address.

```
#eth0
ip rule add from 172.31.2.244 lookup 100
ip route add default via 172.31.0.1 dev eth0 table 100

#eth1
ip rule add from 172.31.0.218 lookup 200
ip route add default via 172.31.0.1 dev eth1 table 200
```
Once the above rules and routes are added, verify the newly added rules and routes using the "ip route" and "ip rule" commands. In case you swap the ENI number and the corresponding private IP address, delete the incorrectly added ip rule by running the command "ip rule del pref #". Enter the five digit number such as (32765) associated with a rule that you want to delete after the word pref. For example:

```
ip rule
0:	from all lookup local
32764:	from 172.31.0.218 lookup 200
32765:	from 172.31.15.216 lookup 100
```

After adding the rules and routes commands, test ping connectivity to both the private IP addresses of the non Amazon Linux instance from another instance.

#### Observation #3:

```
sh-4.2$ ping 172.31.0.218
PING 172.31.0.218 (172.31.0.218) 56(84) bytes of data.
64 bytes from 172.31.0.218: icmp_seq=1 ttl=64 time=1.28 ms
64 bytes from 172.31.0.218: icmp_seq=2 ttl=64 time=1.24 ms

sh-4.2$ ping 172.31.15.216
PING 172.31.15.216 (172.31.15.216) 56(84) bytes of data.
64 bytes from 172.31.15.216: icmp_seq=1 ttl=64 time=1.18 ms
64 bytes from 172.31.15.216: icmp_seq=2 ttl=64 time=1.27 ms
```
#### Note: both the ENIs are pinging after adding the running rules and routes commands. However, ping connectivity to the secondary IP address will be lost if the instance is rebooted and the same rules and routes commands will need to be run again to restore connectivity. 

To make the route changes permanent so that they can survive a reboot, add them to the interfaces file:

```
#eth0
echo "from 172.31.2.244 lookup 100" >> /etc/sysconfig/network-scripts/rule-eth0
echo "172.31.0.0/20 dev eth0 table 100" >> /etc/sysconfig/network-scripts/route-eth0
echo "default via 172.31.0.1 dev eth0 table 100" >> /etc/sysconfig/network-scripts/route-eth0

#eth1
echo "from 172.31.0.218 lookup 200" >> /etc/sysconfig/network-scripts/rule-eth1
echo "172.31.0.0/20 dev eth1 table 200" >> /etc/sysconfig/network-scripts/route-eth1
echo "default via 172.31.0.1 dev eth1 table 200" >> /etc/sysconfig/network-scripts/route-eth1
```

After adding the above configuration, reboot the instance. Once the instance is up after reboot, ping connectivity to the primary and secondary private IP addresses should still be working.

#### Note: In case an instance networking is managed by NetworkManager, additional OS level configuration will be required. This will be covered in another blog as this blog is getting a little bit on the long side.
