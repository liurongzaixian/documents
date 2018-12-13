# Centos下Java和Tomcat环境配置

## Java 安装与配置

### Java 安装

- 去oracle官网下载适应版本的JDK,例如: [jdk-8u181-linux-x64.tar.gz](http://download.oracle.com/otn-pub/java/jdk/8u181-b13/96a7b8442fe848ef90c96a2fad6ed6d1/jdk-8u181-linux-x64.tar.gz)，并拷贝到root用户的根目录

- 解压jdk-8u181-linux-x64.tar.gz文件到`/usr/local`目录下，命令为: `tar xf jdk-8u181-linux-x64.tar.gz -C /usr/local`，使用ls命令查看:`ls -alh /usr/local`，可以看到目录下多了一个文件夹jdk1.8.0_181

- 使用cd命令进入/usr/local目录`cd /usr/local`，创建软连接指向jdk1.8.0_181，命令为:`ln -s jdk1.8.0_181 java`

### Java 环境变量配置

- 在/etc/profile.d目录下创建java.sh文件，并导出JAVA_HOME环境变量到PATH中，命令为:

  ```bash
  cat <<EOF > /etc/profile.d/java.sh
  export JAVA_HOME=/usr/local/java
  export PATH=$PATH:$JAVA_HOME/bin
  EOF
  ```

- 重新加载/etc/profile文件，命令为:`source /etc/profile`

## Tomcat 安装与配置

### Tomcat 安装

- 去Apache官网下载适应版本的tomcat,例如: [apache-tomcat-8.5.32.tar.gz](http://mirrors.shu.edu.cn/apache/tomcat/tomcat-8/v8.5.32/bin/apache-tomcat-8.5.32.tar.gz)，并拷贝到root用户的根目录

- 解压apache-tomcat-8.5.32.tar.gz文件到`/usr/local`目录下，命令为: `tar xf apache-tomcat-8.5.32.tar.gz -C /usr/local`，使用ls命令查看:`ls -alh /usr/local`，可以看到目录下多了一个文件夹apache-tomcat-8.5.32

- 使用cd命令进入/usr/local目录`cd /usr/local`，创建软连接指向apache-tomcat-8.5.32，命令为:`ln -s apache-tomcat-8.5.32 tomcat`

- 新建一个系统用户tomcat，命令为:`useradd -r -s /sbin/nologin tomcat`

- 修改apache-tomcat-8.5.32文件夹的属组，命令为:`chgrp -R tomcat ./apache-tomcat-8.5.32/`

### Tomcat单机多实例配置

注意:下述的所有的LOGIN，请替换成自己的登录用户名,tomcat的实例个数按照需求创建。

- 用户添加到tomcat附加组，命令为:`usermod -a -G tomcat LOGIN`

- 用户根目录新建两个个文件夹例如:tomcat-service，命令为:`mkdir ~/tomcat-service{1,2}`

- 分别在tomcat-service1和tomcat-service2中新建bin、conf、webapps、work、temp文件夹，命令为:`mkdir ~/tomcat-service{1,2}/{bin,conf,webapps,work,temp}`

- 拷贝tomcat中的conf下所有文件到tomcat-service1和tomcat-service2中，命令为:`cp /usr/local/tomcat/conf/* ~/tomcat-service{1,2}/conf`

- 修改tomcat-service1和tomcat-service2 conf文件夹下的server.xml文件

  - tomcat-service1 的server.xml文件
    ```xml
    <Connector port="8080" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" />
    ```
    替换成
    ```xml
    <Connector port="8080" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" URIEncoding="UTF-8" />
    ```
    默认使用UTF-8进行编码。因为一般使用不到AJP，所以把AJP的相关配置文件注释掉
    ```xml
    <!-- <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" /> -->
    ```
  - tomcat-service2 的server.xml文件

    ```xml
    <Server port="8005" shutdown="SHUTDOWN">
    ```
    替换成
    ```xml
    <Server port="9005" shutdown="SHUTDOWN">
    ```
    更改SHUTDOWN监听的端口，避免端口冲突。
     ```xml
    <Connector port="8080" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" />
    ```
    替换成
    ```xml
    <Connector port="9080" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" URIEncoding="UTF-8" />
    ```
    默认使用UTF-8进行编码，同时更改STARTUP端口。
    因为一般使用不到AJP，所以把AJP的相关配置文件注释掉
    ```xml
    <!-- <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" /> -->
    ```

- 编写tomcat服务启动脚本

  - tomcat-service1的启动脚本

    ```bash
    cat <<EOF > ~/tomcat-service1/bin/startup.sh
    #!/bin/bash
    export JAVA_HOME=/usr/local/java
    export CATALINA_HOME=/usr/local/tomcat
    export CATALINA_BASE="/home/LOGIN/tomcat-service1"
    export CATALINA_TMPDIR="$CATALINA_BASE/temp"
    export JAVA_OPTS="-server -Xmx1g -Djava.awt.headless=true -Dtomcat.name=tomcat-service1"

    #创建logs目录
    if [ ! -d "$CATALINA_BASE/logs" ]; then
      mkdir $CATALINA_BASE/logs
    fi

    #创建temp目录
    if [ ! -d "$CATALINA_BASE/temp" ]; then
      mkdir $CATALINA_BASE/temp
    fi

    # 调用tomcat启动脚本
    $CATALINA_HOME/bin/catalina.sh start

    EOF
    ```
  - tomcat-service2启动脚本

    ```bash
    cat <<EOF > ~/tomcat-service2/bin/startup.sh
    #!/bin/bash
    export JAVA_HOME=/usr/local/java
    export CATALINA_HOME=/usr/local/tomcat
    export CATALINA_BASE="/home/LOGIN/tomcat-service2"
    export CATALINA_TMPDIR="$CATALINA_BASE/temp"
    export JAVA_OPTS="-server -Xmx1g -Djava.awt.headless=true -Dtomcat.name=tomcat-service2"

    #创建logs目录
    if [ ! -d "$CATALINA_BASE/logs" ]; then
      mkdir $CATALINA_BASE/logs
    fi

    #创建temp目录
    if [ ! -d "$CATALINA_BASE/temp" ]; then
      mkdir $CATALINA_BASE/temp
    fi

    # 调用tomcat启动脚本
    $CATALINA_HOME/bin/catalina.sh start

    EOF
    ```
  -Xmx 参数的值可以根据需要自己调整，默认值是1g。

- 编写tomcat服务停止脚本

  - tomcat-service1停止脚本

  ```bash
    cat <<EOF > ~/tomcat-service1/bin/shutdown.sh
    #!/bin/bash
    export JRE_HOME=/usr/local/java
    export CATALINA_HOME=/usr/local/tomcat
    export CATALINA_BASE="/home/LOGIN/tomcat-service1"
    export CATALINA_TMPDIR="$CATALINA_BASE/temp"

    bash $CATALINA_HOME/bin/shutdown.sh "$@"

    EOF
  ```

  - tomcat-service2停止脚本

  ```bash
    cat <<EOF > ~/tomcat-service2/bin/shutdown.sh
    #!/bin/bash
    export JRE_HOME=/usr/local/java
    export CATALINA_HOME=/usr/local/tomcat
    export CATALINA_BASE="/home/LOGIN/tomcat-service2"
    export CATALINA_TMPDIR="$CATALINA_BASE/temp"

    bash $CATALINA_HOME/bin/shutdown.sh "$@"

    EOF
  ```

- Centos 7 中把tomcat服务加入到service中，并允许开机自启动，以tomcat-service1为例。在/usr/lib/systemd/system新建tomcatservice1.service

  ```bash
  cat <<EOF > /usr/lib/systemd/system/tomcatservice1.service
  [Unit]
  Description=Tomcat service1 service
  After=syslog.target network.target remote-fs.target nss-lookup.target

  [Service]

  User=LOGIN
  Group=tomcat
  Environment="JENKINS_HOME=/data/gitstore/JENKINS_HOME"

  Type=forking
  ExecStart=/home/LOGIN/tomcat-service1/bin/startup.sh
  ExecReload=/bin/kill -s HUP $MAINPID
  ExecStop=/home/LOGIN/tomcat-service1/bin/shutdown.sh
  PrivateTmp=true

  [Install]
  WantedBy=multi-user.target

  EOF
  ```

  允许开机自启动`systemctl enable tomcatservice1`

  启动tomcatservice1服务`systemctl start tomcatservice1`

  停止tomcatservice1服务`systemctl stop tomcatservice1`