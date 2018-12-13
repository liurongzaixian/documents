# CentOS7 下使用 yum 安装 MySQL57


### MySQL57 下载安装

1. 去 [MySQL](https://dev.mysql.com/downloads/repo/yum/) 官网找到适应的版本

1. 使用 wget 命令下载到 root 用户根目录下  
    `wget 'https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm'`

1. 使用 rpm 命令安装  
    `rpm -Uvh mysql57-community-release-el7-11.noarch.rpm`

1. 使用 yum 命令查看安装情况  
    `yum repolist all | grep mysql`

1. 安装 mysql-community-server  
    `yum install mysql-community-server`

1. 启动 mysql 服务  
    `systemctl start mysqld`

1. 查看 mysql 服务状态  
    `systemctl status mysqld`

----

### MySQL57 配置

1. 查看 mysql 初始密码  
    `grep 'temporary password' /var/log/mysqld.log`

1. 修改密码，<span style="color:red">对新密码复杂度有一定要求!</span>  
    ```
    mysql -uroot -p
    Enter password: 输入初始密码
    mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'MySQL57!';
    ```
1. 查看现有数据库  
    `mysql> show databases;`

1. 修改 mysql 配置  
    `/etc/my.cnf`
----
