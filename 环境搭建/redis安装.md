# Redis安装配置

## Redis安装

- 安装epel-release源 `sudo yum install epel-release -y`

- 安装redis `sudo yum install redis -y`

- 允许远程访问redis,修改`/etc/redis.conf`配置如下

  ```text
  bind 0.0.0.0
  protected-mode no
  ```

- 启动redis服务的三种方式

  ```text
  service redis start

  redis-server /etc/redis.conf

  systemctl start redis
  ```

- 查看redis是否开启`ps -ef | grep redis`

- 关闭服务`redis-cli  shutdown`

- 设置开机自启动`systemctl enable redis.service` 或 `chkconfig redis on`

- 开放防火墙端口允许访问6379端口
