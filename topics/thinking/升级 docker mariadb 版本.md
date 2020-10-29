# 升级 docker mariadb 版本

## 背景

因为准备把当前游戏服务器移动到腾讯云上，而腾讯云上的 mariadb 版本太低，经过沟通腾讯云可以上 mariadb 10.4.12 的版本，比我们当前使用的 10.2.24 版本要高，所以需要我们测试我们的服务器使用 10.4.12 版本的 mariadb 是否正常。

## 升级 mariadb 版本

首先升级镜像版本很简单，只需要修改 backend/db 目录下 DockerFile 中的 FROM 项就好了：

```xml
FROM mariadb:10.2.24
```

升级后容器可以正常启动，但是容器中运行某些 sql 文件时会遇到版本不匹配的报错：

```
ERROR 1558 (HY000) at line 3 in file: 'procs/migrate_db_reset.sql': Column count of mysql.proc is wrong. Expected 21, found 20. Created with MariaDB 100224, now running 100412. Please use mysql_upgrade to fix this error
```

提示我们需要运行 mysql_upgrade 命令去修复这个错误。

## 解决错误

首先先亮出一行命令：

```
docker run --name containername --mount type=bind,src=/dbdir,dst=/var/lib/mysql -p localport:3306 -d mariadb:10.4.12
```

这行命令就是使用挂载的方式启动 mariadb 10.4.12 的版本，挂载的数据目录就是上面命令的 dbdir，端口映射把 localport 改成正确的端口号。

运行命令后会启动容器，接下来我们要做的就是在容器内使用 mysql_upgrade 修复数据：

```
docker exec -it containername mysql_upgrade -uroot -p
```

运行修复命令后控制台会输出修复结果：

```
Phase 1/7: Checking and upgrading mysql database
Processing databases
mysql
mysql.column_stats                                 OK
mysql.columns_priv                                 OK
mysql.db                                           OK
mysql.event                                        OK
mysql.func                                         OK
mysql.gtid_slave_pos                               OK
mysql.help_category                                OK
mysql.help_keyword                                 OK
mysql.help_relation                                OK
mysql.help_topic                                   OK
mysql.host                                         OK
mysql.index_stats                                  OK
mysql.innodb_index_stats                           OK
mysql.innodb_table_stats                           OK
mysql.plugin                                       OK
mysql.proc                                         OK
mysql.procs_priv                                   OK
mysql.proxies_priv                                 OK
mysql.roles_mapping                                OK
mysql.servers                                      OK
mysql.table_stats                                  OK
mysql.tables_priv                                  OK
mysql.time_zone                                    OK
mysql.time_zone_leap_second                        OK
mysql.time_zone_name                               OK
mysql.time_zone_transition                         OK
mysql.time_zone_transition_type                    OK
mysql.user                                         OK
...
以下省略
```

全部输出 OK 后，表示数据修复完成。

然后就像以前一样正常的使用数据库了。