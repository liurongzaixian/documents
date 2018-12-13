# 59 MySQL 服务的 备份与还原


### 备份

1. 使用 backup 用户 备份文件  
    ```
    # 配置文件
    /etc/my.cnf  
    # 以下具体路径在 my.cnf 中  
    /data/mysql  
    /var/log/mysqld.log  
    # 备份 mysql-5.7.22-el7-x86_64.tar.gz 安装包（还原时尽量使用同一版本）
    ```

1. 备份脚本 mysqlbackup.sh
    ```
    rsync -vza --progress -e ssh --delete 
    backup@10.3.175.59:/data/mysql/* /home/backup/mysqlbackup/data >/home/backup/mysqlbackup/logs/syncmysql.log 2>&1
    ```

1. crontab 定时任务   
    ```
    crontab -e
    0 23 * * *  /home/backup/mysqlbackup/mysqlbackup.sh >/dev/null 2>&1
    ```

1. 使用 root 用户重启 crond 服务
    ```
    su root
    systemctl reload crond
    ```


----

### 还原

1. 将对应备份的 配置文件 数据文件 放入
    ```
    # 配置文件
    /etc/my.cnf  
    # 以下具体路径在 my.cnf 中  
    /data/mysql
    ```
----