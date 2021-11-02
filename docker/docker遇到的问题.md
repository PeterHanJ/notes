# Docker遇到的问题

## 问题1：启动，停止Docker 服务

```shell
# 启动 docker
sudo systemctl start docker

# 停止 dockr
sudo systemctl stop docker.socket
sudo systemctl stop docker
```

## 问题2：修改container

可以通过修改配置文件来修改container

1. Stop docker enginer
2. 进入容器目录 /var/lib/docker/containers/
3. 修改 config.v2.json 和 hostconfig.json

```shell
# 添加host和container端口
# docker container 所在路径
#/var/lib/docker/containers/

[root@venhjuhost containers]# pwd
/var/lib/docker/containers
[root@venhjuhost containers]# ls -a
.  ..  f3e7bc4c3f624cb8dffdff9b35a4b95482f56235d07f0ea8783112e437f2d92e


#修改 config.v2.json 和 hostconfig.json
root@venhjuhost f3e7bc4c3f624cb8dffdff9b35a4b95482f56235d07f0ea8783112e437f2d92e]# ls -l
total 48
drwx------ 2 root root     6 Jul 17 11:21 checkpoints
-rw------- 1 root root  3092 Jul 17 12:50 config.v2.json       # 这个
-rw-r----- 1 root root 20826 Jul 17 12:37 f3e7bc4c3f624cb8dffdff9b35a4b95482f56235d07f0ea8783112e437f2d92e-json.log
-rw-r--r-- 1 root root  1472 Jul 17 12:50 hostconfig.json      # 和这个 
-rw-r--r-- 1 root root    13 Jul 17 11:25 hostname         
-rw-r--r-- 1 root root   174 Jul 17 11:25 hosts
drwx-----x 2 root root     6 Jul 17 11:21 mounts
-rw-r--r-- 1 root root    80 Jul 17 11:25 resolv.conf
-rw-r--r-- 1 root root    71 Jul 17 11:25 resolv.conf.hash
[root@venhjuhost f3e7bc4c3f624cb8dffdff9b35a4b95482f56235d07f0ea8783112e437f2d92e]#

# config.v2.json, ports添加8888
 "Ports": {
       "8080/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "8888"
                    },
                    {
                        "HostIp": "::",
                        "HostPort": "8888"
                    }
                ]
    },
    
# hostconfig.json, PortBindings添加8080
 "PortBindings": {
        "8080/tcp": [
                    {
                        "HostIp": "",
                        "HostPort": "8888"
                    }
         ]
  },



```

