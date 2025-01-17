



### mysql单机

```text
docker run 
	-e MYSQL_ROOT_PASSWORD=12345687 
	-d -p 3306:3306 
	-v /Users/heronsbill/docker_data/mysql/standalone/data:/var/lib/mysql 
	-v /Users/heronsbill/docker_data/mysql/standalone/log:/var/log/mysql
	-v /Users/heronsbill/docker_data/mysql/standalone/conf:/etc/mysql
	--name docker-mysql 087c6f86492c


update user set Host='%' where User='root';
flush privileges;
```

```
docker run -e MYSQL_ROOT_PASSWORD=12345687 -d -p 3306:3306 -v /Users/heronsbill/docker_data/mysql/standalone/data:/var/lib/mysql -v /Users/heronsbill/docker_data/mysql/standalone/log:/var/log/mysql -v /Users/heronsbill/docker_data/mysql/standalone/conf:/etc/mysql --name docker-mysql 087c6f86492c
```





### mysql主从

1. 新建主服务容器实例3307

   ```
   docker run -e MYSQL_ROOT_PASSWORD=12345687 -d -p 3307:3306 -v /Users/heronsbill/docker_data/mysql/mysql-master/data:/var/lib/mysql -v /Users/heronsbill/docker_data/mysql/mysql-master/log:/var/log/mysql -v /Users/heronsbill/docker_data/mysql/mysql-master/conf:/etc/mysql --name mysql-master 087c6f86492c
   ```

2. 创建my.cnf配置文件

   ```
   ## 设置客户端与服务端字符集
   [client]
   default_character_set=utf8
   [mysqld]
   collation_server = utf8_general_ci
   character_set_server = utf8
   
   ## 设置server_id，同一局域网中需要唯一
   server_id=101
   ## 指定不需要同步的数据库名称
   binlog-ignore-db=mysql
   ## 开启二进制日志功能
   log-bin=mall-mysql-bin
   ## 设置二进制日志使用内存大小（事务）
   binlog_cache_size=1M
   ## 设置使用的二进制日志格式（mixed,statement,row）
   binlog_format=mixed
   ## 二进制日志过期清理时间。默认值为0，表示不自动清理。
   expire_logs_days=7
   ## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。
   ## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
   slave_skip_errors=1062
   ```

3. 修改完配置后重启master实例

   ```
   docker restart mysql-master
   ```

4. 进入到mysql-master容器，创建数据同步用户

   ```
   docker exec -it mysql-master /bin/bash
   mysql -uroot -p12345687
   CREATE USER 'slave'@'%' IDENTIFIED BY '123456';
   GRANT REPLICATION SLAVE,REPLICATION CLIENT ON test.* TO 'salve'@'%';
   或者
   GRANT ALL PRIVILEGES ON *.* TO 'slave'@'%';
   
   ```

5. 新建服务器容器实例3308

   ```
   docker run -e MYSQL_ROOT_PASSWORD=12345687 -d -p 3308:3306 -v /Users/heronsbill/docker_data/mysql/mysql-slave/data:/var/lib/mysql -v /Users/heronsbill/docker_data/mysql/mysql-slave/log:/var/log/mysql -v /Users/heronsbill/docker_data/mysql/mysql-slave/conf:/etc/mysql --name mysql-slave 087c6f86492c
   ```

6. 创建my.cnf配置文件

   ```
   ## 设置客户端与服务端字符集
   [client]
   default_character_set=utf8
   [mysqld]
   collation_server = utf8_general_ci
   character_set_server = utf8
   
   ## 设置server_id，同一局域网中需要唯一
   server_id=102
   ## 指定不需要同步的数据库名称
   binlog-ignore-db=mysql
   ## 开启二进制日志功能
   log-bin=mall-mysql-bin
   ## 设置二进制日志使用内存大小（事务）
   binlog_cache_size=1M
   ## 设置使用的二进制日志格式（mixed,statement,row）
   binlog_format=mixed
   ## 二进制日志过期清理时间。默认值为0，表示不自动清理。
   expire_logs_days=7
   ## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。
   ## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
   slave_skip_errors=1062
   ## relay_log配置中继日志
   relay_log=mall-mysql-relay-bin  
   ## log_slave_updates表示slave将复制事件写进自己的二进制日志
   log_slave_updates=1  
   ## slave设置为只读（具有super权限的用户除外）
   read_only=1
   ```

7. 修改完成后重启slave实例

   ```
   docker restart mysql-slave
   ```

   

8. 在master数据库查看主从同步状态

   ```
   show master status;
   ```

   ![image-20220515103343890](/Users/heronsbill/Library/Application Support/typora-user-images/image-20220515103343890.png)



9. 进入mysql-slave容器，在从数据库配置主从复制

   ```
   docker exec -it mysql-slave /bin/bash
   mysql -uroot -p12345687
   
   change master to master_host='192.168.106.254', master_user='slave', master_password='123456', master_port=3307, master_log_file='mall-mysql-bin.000004', master_log_pos=1108, master_connect_retry=30;
   
   master_host：主数据库的IP地址；
   master_port：主数据库的运行端口；
   master_user：在主数据库创建的用于同步数据的用户账号；
   master_password：在主数据库创建的用于同步数据的用户密码；
   master_log_file：指定从数据库要复制数据的日志文件，通过查看主数据的状态，获取File参数；
   master_log_pos：指定从数据库从哪个位置开始复制数据，通过查看主数据的状态，获取Position参数；
   master_connect_retry：连接失败重试的时间间隔，单位为秒。
   
   ```

   ![image-20220515104302680](/Users/heronsbill/Library/Application Support/typora-user-images/image-20220515104302680.png)

10. 在从数据库中查看主从同步状态

    ```
    show slave status \G;
    ```

11. 在从数据库中开启主从同步

    ```
    start slave；
    ```

    ![image-20220515104622577](/Users/heronsbill/Library/Application Support/typora-user-images/image-20220515104622577.png)

12. 查看数据库状态发现已经同步

    ```
    show slave status \G;
    ```

    ![image-20220515110519284](/Users/heronsbill/Library/Application Support/typora-user-images/image-20220515110519284.png)

13. 主从复制测试。



问题：对于出现connecting问题的，需关闭io线程，并重启

![image-20220515110748441](/Users/heronsbill/Library/Application Support/typora-user-images/image-20220515110748441.png)

```
stop slave IO_THREAD;
start slave;
```

![image-20220515110834626](/Users/heronsbill/Library/Application Support/typora-user-images/image-20220515110834626.png)















### redis

```
docker run  
	-p 6379:6379 
	--name docker-redis 
	--privileged=true 
	-v /Users/heronsbill/docker_data/redis/redis.conf:/etc/redis/redis.conf 
	-v /Users/heronsbill/docker_data/redis/data:/data 
	-d redis
	redis-server /etc/redis/redis.conf


docker run -p 6379:6379 --name docker-redis --privileged=true -v /Users/heronsbill/docker_data/redis/redis.conf:/etc/redis/redis.conf -v /Users/heronsbill/docker_data/redis/data:/data -d redis redis-server /etc/redis/redis.conf
```





### es

```
docker run 
	-e ES_JAVA_OPTS="-Xms256m -Xmx256m" 
	-e "discovery.type=single-node" 
	-v /usr/share/elasticsearch/config:/Users/heronsbill/docker_data/es/config 
	-v /usr/share/elasticsearch/data:/Users/heronsbill/docker_data/es/data 
	-d 
	-p 9200:9200 
	-p 9300:9300 
	--name es e082d8ac7e5e


docker run -e ES_JAVA_OPTS="-Xms256m -Xmx256m" -e "discovery.type=single-node" -v /usr/share/elasticsearch/config:/Users/heronsbill/docker_data/es/config -v /usr/share/elasticsearch/data:/Users/heronsbill/docker_data/es/data -d -p 9200:9200 -p 9300:9300 --name es e082d8ac7e5e
```

