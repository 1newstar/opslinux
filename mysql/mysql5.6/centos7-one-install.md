## 一、设置主机host
```
cat > /etc/hosts <<EOF
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.56.10 master master.example.com
192.168.56.20 node1  node1.example.com
192.168.56.30 node2  node2.example.com
EOF
```

## 二、设置yum源
```
#Centos自带的repo是不会自动更新每个软件的最新版本，所以无法通过yum方式安装MySQL的高级版本。所以，即使使劲用yum -y install mysql mysql-server mysql-devel，也是没用的。 所以，正确的安装mysql5姿势是要先安装带有可用的mysql5系列社区版资源的rpm包

rpm -Uvh http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm

#这个时候查看当前可用的mysql安装资源：

yum repolist enabled | grep "mysql.*-community.*"
```

## 三、通过 Yum 来安装 MySQL
```
yum clean all   
yum -y install mysql-community-server
```

## 四、启动mysql服务

```
systemctl enable mysqld
systemctl disable mysqld

systemctl start mysqld
systemctl restart mysqld

systemctl stop mysqld

systemctl status mysqld
```

## 五、设置字符集
```
#修改 /etc/my.cnf 文件，添加字符集的设置

vim /etc/my.cnf
[mysqld]
character_set_server = utf8

[mysql]
default-character-set = utf8

#重启 MySQL ,可以看到字符集已经修改了

#最终配置
cat > /etc/my.cnf <<EOF
# For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/5.6/en/server-configuration-defaults.html

[mysqld]
#
# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128M
#
# Remove leading # to turn on a very important data integrity option: logging
# changes to the binary log between backups.
# log_bin
#
# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.
# join_buffer_size = 128M
# sort_buffer_size = 2M
# read_rnd_buffer_size = 2M
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
character_set_server = utf8

# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

# Recommended in standard MySQL setup
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
EOF
```

## 六、mysql修改密码

### 方法一：
```
/usr/bin/mysqladmin -u root password '123456'

#Mysql赋权限和修改密码
mysql> grant all privileges on *.* to root@"%" identified by '123456';
Query OK, 0 rows affected, 1 warning (0.01 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> show grants for root@"%";
+-------------------------------------------------------------+
| Grants for root@%                                           |
+-------------------------------------------------------------+
| GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION |
+-------------------------------------------------------------+
1 row in set (0.00 sec)
```

### 方法二: 安全设置（设置密码）
```
[root@master ~]# mysql_secure_installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MySQL
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MySQL to secure it, we'll need the current
password for the root user.  If you've just installed MySQL, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none):   ----输入当前密码，当前为空，直接Enter
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MySQL
root user without the proper authorisation.

Set root password? [Y/n] y        ----设置root密码
New password:    ----123456
Re-enter new password:    ----123456
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MySQL installation has an anonymous user, allowing anyone
to log into MySQL without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y    ----删除匿名用户
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] n   ----禁止root远程登录
 ... skipping.

By default, MySQL comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y    ----移除test数据库
 - Dropping test database...
ERROR 1008 (HY000) at line 1: Can't drop database 'test'; database doesn't exist
 ... Failed!  Not critical, keep moving...
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y    ----重新加载权限
 ... Success!

All done!  If you've completed all of the above steps, your MySQL
installation should now be secure.

Thanks for using MySQL!


Cleaning up...
[root@master ~]#
```

