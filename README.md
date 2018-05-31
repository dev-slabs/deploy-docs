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
