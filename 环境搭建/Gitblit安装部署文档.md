# Gitblit 搭建git源码库

## Gitblit安装

- gitblit官网下载最新的release版本的[war](http://dl.bintray.com/gitblit/releases/gitblit-1.8.0.war)包到用户根目录下。

- 新建一个tomcat实例，例如tomcat-gitblit,并设置开机自启动,参考文档`CentOS7下Java和Tomcat环境配置`。

- 把gitblit-1.8.0.war 拷贝到tomcat-gitblit实例中的webapps目录下，命令为:`cp gitblit-1.8.0.war ~/tomcat-gitblit/webapps/gitserver.war`。

- 启动tomcat-gitblit服务。

- 修改gitblit默认的数据存放位置
  - 在用户根目录新建git_repositories目录，命令为:`mkdir ~/git_repositories`
  - 修改tomcat-gitblit/webapps/gitserver/WEB-INF/web.xml文件，启用自定义数据存放,指向/home/LOGIN/git_repositories，display-name修改为gitserver

    ```xml
    <env-entry>
      <description>The base folder is used to specify the root location of your Gitblit data.</description>
      <env-entry-name>baseFolder</env-entry-name>
      <env-entry-type>java.lang.String</env-entry-type>
      <env-entry-value>/home/LOGIN/git_repositories</env-entry-value>
    </env-entry>

    <!-- Gitblit Displayname -->
    <display-name>gitserver</display-name>
    ```
- 重新启动tomcat-gitblit服务。

- 浏览器访问http://ip:port/gitserver ，使用默认帐号admin登录，密码admin，注意提前开放防火墙端口，否则无法访问。

## Gitblit集成LDAP

- 使用vi编辑器编辑~/git_repositories/gitblit.properties文件，命令为:`vi ~/git_repositories/gitblit.properties`,在文件中添加如下

  ```properties
  realm.authenticationProviders = ldap
  realm.ldap.server = xxxxxxxx
  realm.ldap.username = xxxxx
  realm.ldap.password = xxxxx
  realm.ldap.maintainTeams = true
  realm.ldap.accountBase = ou=xxx,dc=xxx,dc=com
  realm.ldap.accountPattern = (&(objectClass=person)(sAMAccountName=${username}))
  realm.ldap.groupBase = ou=xxx,dc=xxx,dc=com
  #Gitblit网页上的过滤，只能查看到【Git_】开头的组(域控中存在几个git的安全组，名字为Git_Application)
  realm.ldap.groupMemberPattern = (&(objectClass=group)(name=Git_*)(member=${dn}))
  realm.ldap.admins = @Git_Admins

  # 不显示ssh协议的仓库地址
  web.showSshDaemonUrls = false
  # 不显示git协议的仓库地址
  web.showGitDaemonUrls = false
  # 不显示默认的http协议的仓库地址
  web.showHttpServletUrls = false
  # 显示自定义的http协议的仓库地址
  web.otherUrls = xxxxx
  # 针对otherUrls开启权限控制
  web.advertiseAccessPermissionForOtherUrls = true
  # 页面上不允许fork
  web.allowForking = false

  # 禁止删除非空的仓库
  web.allowDeletingNonEmptyRepositories = false

  # 开启tickets
  tickets.service = com.gitblit.tickets.BranchTicketService


  #网页上时间的显示格式
  web.timezone = Asia/Shanghai
  web.timeFormat = HH:mm:ss
  web.datestampShortFormat = yyyy-MM-dd
  web.datestampLongFormat = EEEE, yyyy-MM-dd
  web.datetimestampLongFormat = EEEE, yyyy-MM-dd HH:mm:ss Z

  # 参考http://gitblit.com/faq.html，防止tomcat处于nginx后面的时候，发生/被转换的问题，有3种解决方案。
  # 1.修改web.forwardSlashCharacter属性为【web.forwardSlashCharacter = !】
  # 2.修改web.mountParameters属性为【web.mountParameters = false】
  # 3.在tomcat的catalina.sh的开始位置，加入
  #   CATALINA_OPTS="${CATALINA_OPTS} -Dorg.apache.tomcat.util.buf.UDecoder.ALLOW_ENCODED_SLASH=true"
  #现在使用第3种方式，同时配置下面的选项
  web.forwardSlashCharacter = !
  ```

- 重新启动tomcat-gitblit服务，并使用Ldap帐号登录。

## Gitblit 数据备份，[参考官方文档](http://gitblit.com/federation.html)

- 在备份机上下载[Federation Client](http://dl.bintray.com/gitblit/releases/fedclient-1.8.0.zip)到gitbackup目录。

- 使用vi编辑器编辑~/git_repositories/gitblit.properties文件，命令为:`vi ~/git_repositories/gitblit.properties`,在文件中添加如下

  ```properties
  federation.passphrase=XXXX
  ```

- 重新启动tomcat-gitblit服务，在catalina.out文件中找到如下内容,拷贝出来，留待后用

  ```txt
  ----[com.gitblit.manager.IServicesManager]----
  Federation ALL token = 5d8fd20bc68fab1b6d1bb2fcb79eb73df2b3d88c
  Federation USERS_AND_REPOSITORIES token = f0c98660c5c513495a3b283764ed5ed46650235e
  Federation REPOSITORIES token = 3bb1c5e51d4b38d7de512af7ee7622169c7c5a90

  ```

- 解压fedclient-1.8.0.zip 文件到当前目录命令为:`unzip fedclient-1.8.0.zip`，当前目录下多出fedclient文件夹。

- 在gitbackup目录下新建repositories，作为备份文件的存放路径。

- 修改fedclient下federation.properties文件如下，`vi ~/gitbackup/fedclient/federation.properties`

  ```properties
  git.repositoriesFolder = /home/LOGIN/gitbackup/repositories/git
  federation.name =voice-olami

  federation.voice.url = http://git.olavoice.com/gitserver
  federation.voice.token = 5d8fd20bc68fab1b6d1bb2fcb79eb73df2b3d88c
  federation.voice.frequency = 120 mins
  federation.voice.folder =
  federation.voice.bare = true
  federation.voice.mirror = true
  federation.voice.mergeAccounts = true

  ```

  备份分三种，Federation ALL token 表示备份所有文件包含用户信息，配置文件信息，代码库、Federation USERS_AND_REPOSITORIES token 表示备份用户信息，代码库、Federation REPOSITORIES token表示仅仅备份代码库。我们这里采用第一种，备份所有信息。

- 编写备份脚本gitbackup.sh文件

  ```bash
  cat <<EOF > /home/LOGIN/gitbackup/gitbackup.sh
  cd /home/LOGIN/gitbackup/fedclient
  java -jar fedclient.jar >/home/LOGIN/gitbackup/logs/syncgit.log 2>&1
  EOF
  ```

- 测试脚本文件，添加可执行权限，命令为:`chmod u+x /home/LOGIN/gitbackup/gitbackup.sh`,执行gitbackup.sh文件，看看/home/LOGIN/gitbackup/logs/syncgit.log是否正常打印。

- 添加crontab任务，每天定时执行`crontab -e`

  ```bash
  0 22 * * *  /home/LOGIN/gitbackup/gitbackup.sh >/dev/null 2>&1
  ```

- root用户重新reload crond服务`systemctl reload crond`

## Gitblit 数据恢复

- 拷贝/home/LOGIN/gitbackup/repositories/git下的所有文件到/home/LOGIN/git_repositories/git目录下

- 拷贝/home/LOGIN/git_repositories/git下的voice_users.conf到/home/LOGIN/git_repositories目录下并改名为users.conf

- 拷贝/home/LOGIN/git_repositories/git下的voice_gitblit.properties到/home/LOGIN/git_repositories目录下并改名为gitblit.properties

- 重启tomcat-gitblit服务
