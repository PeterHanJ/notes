# Docker网络

## 理解Docker网络

```shell
[root@venhjuhost /]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000 # 本机回环地址
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000 # 阿里云内网地址
    link/ether 00:16:3e:00:17:84 brd ff:ff:ff:ff:ff:ff
    inet 172.28.1.254/20 brd 172.28.15.255 scope global dynamic noprefixroute eth0
       valid_lft 315329398sec preferred_lft 315329398sec
    inet6 fe80::216:3eff:fe00:1784/64 scope link
       valid_lft forever preferred_lft forever
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default  # Docker网卡 172.17.0.1
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

三个网络

```shell
问题：Docker是如何处理容器访问的

# 进入tomcar容器
# ip addr查看容器网络环境，容器启动会得到一个 eth0@if31 ip地址，docker 分配的
[root@venhjuhost /]# docker exec -it 7ca908535c1a ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
30: eth0@if31: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default  
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0   # docker分配 172.17.0.2
       valid_lft forever preferred_lft forever

# linux可以ping通容器内部
[root@venhjuhost /]# ping 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.127 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.059 ms


```

原理

```shell
1.我们每启动一个docker容器，docker就会给容器分配一个ip.我们只要安装了docker,就会有一个网卡 docker0。
  桥接模式，使用的技术是evth-pair技术！

[root@venhjuhost /]# ip addr

# host 主机中多了一个 31: vethabe3ade@if30 （指向容器中的30）
31: vethabe3ade@if30: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
    link/ether 2a:50:74:dc:be:57 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::2850:74ff:fedc:be57/64 scope link
       valid_lft forever preferred_lft forever
       
# tomcat容器中多了一个 30: eth0@if31 （指向host中的31）       
30: eth0@if31: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default  
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0   # docker分配 172.17.0.2
       valid_lft forever preferred_lft forever

# 我们发现这个容器带来网卡，都是一对一对的
# veth-pair 就是一对的虚拟设备接口,一端连着协议，一端彼此相连
# 正因为有这个特性，veth-pair充当一个桥梁，连接各种虚拟网络设备
# 结论：容器和容器中间是可以互相ping通的。
# Docker使用的linux的桥接，宿主机中是一个Docker容器的网桥 docker0

```

```shell
# 思考：我们编写了一个微服务，database url = ip, 项目不重启，数据库ip换掉了，我们希望可以处理这个问题。
# 可以用名字来访问容器
  
```

## 容器互联 --link(不推荐使用)

```shell
# 有两个启动的tomcat 容器
CONTAINER ID   IMAGE             COMMAND             CREATED          STATUS          PORTS                                       NAMES
9dfe7df9aa0f   tomcat:9.0-jdk8   "catalina.sh run"   11 seconds ago   Up 9 seconds    0.0.0.0:8887->8080/tcp, :::8887->8080/tcp   tomcat02
8acfdad518f2   tomcat:9.0-jdk8   "catalina.sh run"   44 seconds ago   Up 42 seconds   0.0.0.0:8888->8080/tcp, :::8888->8080/tcp   tomcat01

# 尝试用名字来ping
[root@venhjuhost ~]# docker exec -it tomcat01 ping tomcat02
ping: tomcat02: Name or service not known

# 再创建一个tomcat03, 用--link来指定连接 tomcat01
[root@venhjuhost ~]# docker run -d -p 8886:8080 --name tomcat03 --link tomcat01 tomcat:9.0-jdk8

# tomcat03 可以 ping通 tomcat01
[root@venhjuhost ~]# docker exec -it tomcat03 ping tomcat01
PING tomcat01 (172.17.0.2) 56(84) bytes of data.
64 bytes from tomcat01 (172.17.0.2): icmp_seq=1 ttl=64 time=0.137 ms

# 但是连接只是单向的，tomcat01并不能通过 tomcat03命来连接
[root@venhjuhost ~]# docker exec -it tomcat01 ping tomcat03
ping: tomcat03: Name or service not known

# 查看tomcat03 context信息，在HostConfig中多了 Links的设定
  "Links": [
                "/tomcat01:/tomcat03/tomcat01"
           ],

```

## 探究 --link

```shell
# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
25ee6a721615   bridge    bridge    local        # 容器 bridge
1d8dcfd33835   host      host      local
b7d58b5dc5f0   none      null      local

# docker network inspect
[root@venhjuhost ~]#  docker network inspect 25ee6a721615
[
    {
        "Name": "bridge",
        "Id": "25ee6a7216156ed8940c436415e2b6ff7c0eba7ebf687c506fc6bd612de77cad",
        "Created": "2021-07-17T16:13:53.982514471+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",           # 172.17.0.2 ~ 172.17.255.254
                    "Gateway": "172.17.0.1"              # Gateway
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {                                   # Contaner
            "11480669f2411dbc7808ede856c8885ff90ba71f79249ef2dedf45ed6cbbadf0": {
                "Name": "tomcat03",
                "EndpointID": "cccea74e687d973c7f35ee96aa93b2b307bfbba4246ea8e1e8275e5e537be078",
                "MacAddress": "02:42:ac:11:00:04",
                "IPv4Address": "172.17.0.4/16",
                "IPv6Address": ""
            },
            "8acfdad518f236c88025e9bb87ab41a40a7aae820b694f4126914c49385a118b": {
                "Name": "tomcat01",
                "EndpointID": "31e510efc8b1a6308f428ab32d3fa75c8710fbdbd35eb29877dae59526dd7d2f",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            },
            "9dfe7df9aa0fe44fd7e02f2e3ccc179e727193011f3ebfc36362a98da015d1e6": {
                "Name": "tomcat02",
                "EndpointID": "3555b3a52580236e417014a6c02d8ae89522e8b05a5e782b5a3500fd232acb8e",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]

# docker inspect tomcat03
# 查看tomcat03 context信息，在HostConfig中多了 Links的设定
  "Links": [
                "/tomcat01:/tomcat03/tomcat01"
           ],

# docker exec -it tomcat03 cat /etc/hosts
# 查看tomcat03 hosts文件信息
[root@venhjuhost ~]# docker exec -it tomcat03 cat /etc/hosts
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.2      tomcat01 8acfdad518f2        # 加了一个 tomcat01 的mapping
172.17.0.4      11480669f241

```

本质探究：
--link 就是我们在hosts中配置中增加了一个 tomcat01 的mapping
不建议使用。

## 自定义网络

```shell
# 查看所有的docker网络
[root@venhjuhost ~]# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE   
25ee6a721615   bridge    bridge    local   # docker默认
1d8dcfd33835   host      host      local   # 和宿主机共享网络
b7d58b5dc5f0   none      null      local   # 不配置网络 

```

网络模式

- bridge ：桥接， docker 默认 （自己创建也用桥接模式）
- host ：   和host共享网络
- none:     不配置网络
- container: 容器内联通 （用的少）

测试

```shell
# 我们直接启动容器，其实是默认用 --net bridge (docker0)
# -P 自动分配端口给容器expose端口
docker run -d -P --name tomcat01 tomcat
docker run -d -P --name tomcat01 --net bridge tomcat

# docker network create
# 自定义一个网络
# --drive 默认桥接
# --subnet 192.168.0.0/16  192.168.0.2 ~ 192.168.255.254
# --gateway
[root@venhjuhost ~]# docker network create --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet
257cf1f12ff7d69b697a5a8de0fab89eb5f4f3d80661abdaf69f47715a2fc3e9

[root@venhjuhost ~]# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
25ee6a721615   bridge    bridge    local
1d8dcfd33835   host      host      local
257cf1f12ff7   mynet     bridge    local       # 自己创建的
b7d58b5dc5f0   none      null      local

# 用自定义网络运行容器 --net mynet
[root@venhjuhost ~]# docker run -d -P --name tomcat-net-1 --net mynet tomcat:9.0-jdk8
2103adbc6f55aa2453325dba8c30567d95235d25fef01daf357af069b397d537
[root@venhjuhost ~]# docker run -d -P --name tomcat-net-2 --net mynet tomcat:9.0-jdk8
bff14d7762e989f2e5ee32df9502eae41e8225ccc292df680f8107f0c870089a

# 查看自定义网络
[root@venhjuhost ~]# docker network inspect mynet

# 现在通过 ping + 容器名可以 ping通
# 我们自定义的网咯docker已经帮我们维护好了对应的关系
[root@venhjuhost ~]# docker exec -it tomcat-net-1 ping tomcat-net-2
PING tomcat-net-2 (192.168.0.3) 56(84) bytes of data.
64 bytes from tomcat-net-2.mynet (192.168.0.3): icmp_seq=1 ttl=64 time=0.121 ms

```

好处：

- 不同的集群使用不同的网络，保证集群是安全和健康的



## 网络连通

```shell
# 当tomcat01 运行在brirge(docker0 ，172.17.0.0)网段下
# 要去连接mynet(192.168.0.0)下，需要使用docker network connect命令
Usage:  docker network connect [OPTIONS] NETWORK CONTAINER
Connect a container to a network
Options:
      --alias strings           Add network-scoped alias for the container
      --driver-opt strings      driver options for the network
      --ip string               IPv4 address (e.g., 172.30.100.104)
      --ip6 string              IPv6 address (e.g., 2001:db8::33)
      --link list               Add link to another container
      --link-local-ip strings   Add a link-local address for the container

# tomcat01 连接自定义网络mynet
[root@venhjuhost ~]# docker network connect mynet tomcat01

# 查看mynet
# 在mynet中，container中加了一条tomcat01的设定，分配地址192.168.0.4
 "1c72d05fd7ef62e932885f60883ce66e21c59875059be3afcad679851d5fb86d": {
                "Name": "tomcat01",
                "EndpointID": "8a41cc75fd78f26554bf0fcb6018eef003276fd70e1a2b6f89e5bbfa9636db0f",
                "MacAddress": "02:42:c0:a8:00:04",
                "IPv4Address": "192.168.0.4/16",        # 分配了一个mynet下的ip地址
                "IPv6Address": ""
            },


# 查看tomcat01中的ip addr,也同时多了一条192.168.0.4的地址
45: eth1@if46: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:c0:a8:00:04 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.0.4/16 brd 192.168.255.255 scope global eth1
       valid_lft forever preferred_lft forever

# 这就相当于一个container有两个ip
# 其中mynet(192.168.0.0)那个就可以和mynet中的container通信
[root@venhjuhost ~]# docker exec -it tomcat01 ping tomcat-net-1
PING tomcat-net-1 (192.168.0.2) 56(84) bytes of data.
64 bytes from tomcat-net-1.mynet (192.168.0.2): icmp_seq=1 ttl=64 time=0.135 ms
64 bytes from tomcat-net-1.mynet (192.168.0.2): icmp_seq=2 ttl=64 time=0.099 ms

```



 