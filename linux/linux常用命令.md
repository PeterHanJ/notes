# 常用命令

## 网路相关

### netstat

```shell
# 查找监听端口
netstat -an | grep 8989

# 安装 netstat
# yum install net-tools     [On CentOS/RHEL]
# apt install net-tools     [On Debian/Ubuntu]
# zypper install net-tools  [On OpenSuse]
# pacman -S net-tools      [On Arch Linux]
```

### ip addr

```shell
[root@venhjuhost /]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:16:3e:00:17:84 brd ff:ff:ff:ff:ff:ff
    inet 172.28.1.254/20 brd 172.28.15.255 scope global dynamic noprefixroute eth0
       valid_lft 315329398sec preferred_lft 315329398sec
    inet6 fe80::216:3eff:fe00:1784/64 scope link
       valid_lft forever preferred_lft forever
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:6d:ad:ae:77 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:6dff:fead:ae77/64 scope link
       valid_lft forever preferred_lft forever
31: vethabe3ade@if30: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
    link/ether 2a:50:74:dc:be:57 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::2850:74ff:fedc:be57/64 scope link
       valid_lft forever preferred_lft forever

```

### tcpdump

```shell
# 用简单的话来定义tcpdump，就是：dump the traffic on a network，根据使用者的定义对网络上的数据包进行截获的包分析工具。  tcpdump可以将网络中传送的数据包的“头”完全截获下来提供分析。它支持针对网络层、协议、主机、网络或端口的过滤，并提供and、or、# not等逻辑语句来帮助你去掉无用的信息。

 
```

### firewall-cmd

```shell
# 查看firewall服务状态
systemctl status firewalld

# 开启、重启、关闭、firewalld.service服务
# 开启
service firewalld start
# 重启
service firewalld restart
# 关闭
service firewalld stop

# 查看防火墙规则
firewall-cmd --list-all    # 查看全部信息
firewall-cmd --list-ports  # 只看端口信息

# 开启端口
开端口命令：firewall-cmd --zone=public --add-port=80/tcp --permanent
重启防火墙：systemctl restart firewalld.service

命令含义：
--zone #作用域
--add-port=80/tcp  #添加端口，格式为：端口/通讯协议
--permanent   #永久生效，没有此参数重启后失效
```

# VIM主题设置

https://cloud.tencent.com/developer/article/1726769

