# deploy-docs
d2d 网关部署文档(Linux)

(Ubuntu)
1、JDK环境(1.8)
    
    root@ubuntu:~# java -version
    java version "1.8.0_171"
    Java(TM) SE Runtime Environment (build 1.8.0_171-b11)
    Java HotSpot(TM) 64-Bit Server VM (build 25.171-b11, mixed mode)

   如果未安装JDK、自行到Oracle官网下载 
   
    官网地址
   
    http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
    
    安装JDK
        1、 解压 tar -zxvf jdk-8u144-linux-x64.tar.gz
        2、 配置环境变量
        
            root@ubuntu:~/tools/jdk/jdk1.8.0_144# pwd
            /root/tools/jdk/jdk1.8.0_144
            root@ubuntu:~/tools/jdk/jdk1.8.0_144# vim ~/.profile
            export JAVA_HOME=/root/tools/jdk/jdk1.8.0_144
            export PATH=$JAVA_HOME/bin:$PATH
            source ~/.profile(使配置文件生效)
        3、 检查java版本
            java -version 

2、docker安装

    查看版本信息
    root@ubuntu:~# lsb_release -a
    No LSB modules are available.
    Distributor ID: Ubuntu
    Description:    Ubuntu 16.04.4 LTS
    Release:        16.04   // 官方16.04LTS系统版本
    Codename:       xenial  // 16.04版本系统对应的代号
    root@ubuntu:~# 

    1、下载对应版本代号的deb包
    https://download.docker.com/linux/ubuntu/dists/xenial/pool/stable/amd64/
    root@ubuntu:~/tools/docker# wget https://download.docker.com/linux/ubuntu/dists/xenial/pool/stable/amd64/docker-ce_17.03.2~ce-0~ubuntu-xenial_amd64.deb
    2、安装
    root@ubuntu:~/tools/docker# sudo dpkg -i docker-ce_17.03.2~ce-0~ubuntu-xenial_amd64.deb        
    3、验证docker时候安装成功
    运行hello world镜像（直接运行即可　自动从服务器上拉取demo镜像）
    sudo docker run hello-world
    正确结果如下：
        ....    
        Hello from Docker!
        ....
    4、docker 更换阿里云镜像仓库地址
    root@ubuntu:~/tools/docker# vim /etc/systemd/system/multi-user.target.wants/docker.service 
    ExecStart=/usr/bin/dockerd -H fd:// --registry-mirror=https://jxus37ad.mirror.aliyuncs.com
    重新加载并重新启动
    sudo systemctl daemon-reload
    sudo systemctl restart docker
    
3、docker compose 安装
    
    1、sudo apt-get -y install python-pip
    2、sudo pip install --upgrade pip
    3、sudo pip install docker-compose
        
4、通过docker-compose安装mysql5.7

    1、mkdir docker-mysql && cd docker-mysql
    2、vim docker-compose.yml
        
        version: '2'
        services:
          db:
            image: mysql:5.7
            restart: always
            command: --max_allowed_packet=20245646    
          # Set max_allowed_packet to 256M (or any other value)
            environment:
          # mysql root 密码
              MYSQL_ROOT_PASSWORD: dangerous
            ports:
            - "3306:3306"
    3、docker-compose up -d 创建并运行
    
    4、检测mysql安装
        
       root@ubuntu:~/tools/docker-mysql# docker ps
       CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
       c7f2d38bb514        mysql:5.7           "docker-entrypoint..."   5 minutes ago       Up 12 seconds       0.0.0.0:3306->3306/tcp   docker-mysql_db_1
       root@ubuntu:~/tools/docker-mysql# docker exec -it c7f2d38bb514 bash
       root@c7f2d38bb514:/# mysql -uroot -p
       Enter password: 
       Welcome to the MySQL monitor.  Commands end with ; or \g.
       Your MySQL connection id is 2
       Server version: 5.7.22 MySQL Community Server (GPL)
       
       Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.
       
       Oracle is a registered trademark of Oracle Corporation and/or its
       affiliates. Other names may be trademarks of their respective
       owners.
       
       Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
       
       mysql> 

5、目录结构介绍(部署的步骤和以下介绍顺序有关系)
    
    (1)map_server  #服务发现(为了动态的加入节点)
        ├── local.yaml #配置文件
        └── server     #可执行的二进制文件
        
        local.yaml
        name: Corda Network Map
        expiration: 8 #node每隔几秒去向 map server拿到配置
        server:
          applicationConnectors:#节点连接的端口
          - type: http
            port: 8092
          adminConnectors:#节点管理的端口
          - type: http
            port: 8091
        
        #启动命令
        nohup ./server &
    (2)notory节点
        ├── corda.jar     #corda对应的jar
        ├── cordapps      #dapp目录(所有dapp都放在此目录下)
        │   └── dapp.jar
        ├── everchain.txt #ervrchain的地址配置
        └── node.conf     #notory节点配置文件
        
        #node.conf
        #notory的节点名称
        myLegalName : "O=Nie pan,OU=Notary,L=London,C=GB"
        #解密包含节点证书和私钥的KeyStore文件（ /certificates/sslkeystore.jks）的密码
        keyStorePassword : "cordacadevpass"
        #解锁包含Corda网络根证书的Trust存储文件（ /certificates/truststore.jks）的密码。这是在第一个节点运行期间自动生成的开发证书的非秘密值。
        trustStorePassword : "trustpass"
        #ArtemisMQ上的节点可用于协议操作的主机和端口。
        p2pAddress : "localhost:12345"
        #可以向节点发出RPC请求的RPC系统的地址。如果未提供，那么节点将在没有RPC的情况下运行。现在不赞成使用rpcSettings块。
        rpcSettings = {
            #布尔值，指示节点是否应要求客户端使用SSL进行RPC连接，默认为false
            useSsl = true
            #布尔值，指示节点是否将连接到RPC的独立代理，默认为false
            standAloneBroker = false
            #用于RPC服务器绑定的主机和端口
            address : "0.0.0.0:10007"
            #用于RPC管理绑定的主机和端口（仅在useSsl为false时需要，因为该节点使用SSL连接到Artemis以确保在节点外部无法访问管理员权限）。
            adminAddress : "127.0.0.1:10008"
        }
        #mysql数据库配置
        dataSourceProperties : {
             dataSourceClassName : com.mysql.jdbc.jdbc2.optional.MysqlDataSource
            "dataSource.url" : "jdbc:mysql://*****:3306/tebon_notary"
            "dataSource.user" : ****
            "dataSource.password" : "*****"
        }
        #配置自定义notary服务
        notary : {
            #如果为true，将从CorDapp加载并安装公证服务
            custom=true
            #布尔值来确定公证是否是有效的或无效的。
            validating=true
        }
        #开发模式
        devMode : true
        #连接map server的ip端口
        compatibilityZoneURL : "http://127.0.0.1:8092"
       
        #启动notary节点
        [root@iz8vb30c5x7ch056nz03d1z notary]# nohup java -jar corda.jar &
        #启动成功标志
         Node for "Nie pan" started up and registered in 22.84 sec
    (3)party  
        ├── corda.jar #corda对应jar
        ├── cordapps  #dapp目录(所有dapp都放在此目录下)
        │     └── dapp.jar
        └── node.conf #party对应的配置文件
        
        #node.conf
        #party节点名称
        myLegalName : "O=Party A,L=Shanghai,C=GB"
        #解密包含节点证书和私钥的KeyStore文件（ /certificates/sslkeystore.jks）的密码
        keyStorePassword : "cordacadevpass"
        #解锁包含Corda网络根证书的Trust存储文件（ /certificates/truststore.jks）的密码。这是在第一个节点运行期间自动生成的开发证书的非秘密值。
        trustStorePassword : "trustpass"
        #数据库配置
        dataSourceProperties : {
            dataSourceClassName : com.mysql.jdbc.jdbc2.optional.MysqlDataSource
            "dataSource.url" : "jdbc:mysql://****:3306/partya"
            "dataSource.user" : *****
            "dataSource.password" : "*****"
        }
        #ArtemisMQ上的节点可用于协议操作的主机和端口。
        p2pAddress : "127.0.0.1:10049"
        rpcSettings = {
            #布尔值，指示节点是否应要求客户端使用SSL进行RPC连接，默认为false
            useSsl = false
            #布尔值，指示节点是否将连接到RPC的独立代理，默认为false
            standAloneBroker = false
            #用于RPC服务器绑定的主机和端口
            address : "0.0.0.0:10043"
            #ArtemisMQ上的节点可用于协议操作的主机和端口
            adminAddress : "0.0.0.0:10044"
        }
        #有权访问RPC系统的用户列表。列表中的每个用户都是具有以下字段的配置对象：
        rpcUsers : [
            { username=user1, password=letmein, permissions=[ ALL ] }
        ]
        #开发模式
        devMode : true
        #map serverip端口
        compatibilityZoneURL : "http://127.0.0.1:8092"
        
        #启动命令
        [root@iz8vb30c5x7ch056nz03d1z partya]# nohup java -jar corda.jar &
        #启动成功标志
        Node for "Party A" started up and registered in 258.93 sec