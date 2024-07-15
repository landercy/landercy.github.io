# DOCKER-COMPOSE

```yml
networks:
  work_net:
    external: true
services:
  mssql2019:
    container_name: mssql2019
    image: mcr.microsoft.com/mssql/server:2019-latest
    restart: unless-stopped
    networks:
      work_net:
    ports:
      - '1433:1433'
    user: root
    volumes:
      - './data:/var/opt/mssql'
    environment:
      ACCEPT_EULA: 'Y'
      SA_PASSWORD: 'Admin123'
```

```sh
sudo docker exec -it mssql2019 sqlcmd -S 127.0.0.1 -U SA

ln -s /opt/mssql-tools/bin/sqlcmd /bin/sqlcmd
```

# case

```sh
sudo docker cp /tmp/wms.bak mssql2019:/tmp/
sudo docker exec -it mssql2019 sqlcmd -S 172.16.10.35 -U SA -Q "RESTORE DATABASE [wms] FROM DISK = N'/tmp/wms.bak'"
```

> Directory lookup for the file "C:\Program Files\Microsoft SQL Server\MSSQL13.MSSQLSERVER\MSSQL\DATA\wms.mdf" failed with the operating system error 2(The system cannot find the file specified.).

```sql
RESTORE FILELISTONLY FROM DISK = '/tmp/wms.bak'
GO

```

| LogicalNamecol | PhysicalNamecol                                                                  |
| -------------- | -------------------------------------------------------------------------------- |
| wms            | C:\Program Files\Microsoft SQL Server\MSSQL13.MSSQLSERVER\MSSQL\DATA\wms.mdf     |
| wms_log        | C:\Program Files\Microsoft SQL Server\MSSQL13.MSSQLSERVER\MSSQL\DATA\wms_log.ldf |


```sql
RESTORE DATABASE wms
FROM DISK = '/tmp/wms.bak'
WITH MOVE 'wms' TO '/var/opt/mssql/data/wms.mdf',
MOVE 'wms_log' TO '/var/opt/mssql/data/wms_log.ndf'
GO
```


```sh
# mssql-server
sudo curl -o /etc/yum.repos.d/mssql-server.repo https://packages.microsoft.com/config/rhel/7/mssql-server-2019.repo
# sqlcmd
sudo curl -o /etc/yum.repos.d/msprod.repo https://packages.microsoft.com/config/rhel/7/prod.repo 

sudo yum install -y mssql-server mssql-tools
```

```

  1) Evaluation (free, no production use rights, 180-day limit)
  2) Developer (free, no production use rights)
  3) Express (free)
  4) Web (PAID)
  5) Standard (PAID)
  6) Enterprise (PAID)
  7) Enterprise Core (PAID)
  8) I bought a license through a retail sales channel and have a product key to enter.
 
Details about editions can be found at
https://go.microsoft.com/fwlink/?LinkId=852748&clcid=0x409
 
Use of PAID editions of this software requires separate licensing through a
Microsoft Volume Licensing program.
By choosing a PAID edition, you are verifying that you have the appropriate
number of licenses in place to install and run this software.
 
Enter your edition(1-8): 3
The license terms for this product can be found in
/usr/share/doc/mssql-server or downloaded from:
https://go.microsoft.com/fwlink/?LinkId=855862&clcid=0x409
 
The privacy statement can be viewed at:
https://go.microsoft.com/fwlink/?LinkId=853010&clcid=0x409
 
Do you accept the license terms? [Yes/No]:yes
 
Enter the SQL Server system administrator password: Qq123123
Confirm the SQL Server system administrator password:
Configuring SQL Server...
 
The licensing PID was successfully processed. The new edition is [Express Edition].
ForceFlush is enabled for this instance.
ForceFlush feature is enabled for log durability.
Created symlink from /etc/systemd/system/multi-user.target.wants/mssql-server.service to /usr/lib/systemd/system/mssql-server.service.
Setup has completed successfully. SQL Server is now starting.
```

> t06: 8CWo^$n8WVp1UB4G 

配置
```
+--------------------------------------------------------------+
Please run 'sudo /opt/mssql/bin/mssql-conf setup'
to complete the setup of Microsoft SQL Server
+--------------------------------------------------------------+
...
Complete!

[root@et01 ~]#  /opt/mssql/bin/mssql-conf setup
Choose an edition of SQL Server:
```

状态
```
systemctl status mssql-server
```
工具
```
sudo ln -sf /opt/mssql-tools/bin/bcp /usr/bin/bcp
sudo ln -sf /opt/mssql-tools/bin/sqlcmd /usr/bin/sqlcmd
```

命令行
```
# 链接
sqlcmd -S localhost -U SA -P
Password: Qq123123
1> select 1
2> go
```
技术文档

https://docs.microsoft.com/zh-cn/sql/sql-server/?view=sql-server-2017
```
# 查询所有数据库名
select name from sysdatabases;
# 新建数据库
create database wms
# 使用数据库
use wms
# 不区分大小写
alter database 数据库名称 COLLATE Chinese_PRC_CI_AS
# 新建用户
create login wms with password='wms123', default_database=wms

--读取库中的所有表名
select name from sysobjects where xtype='u'
--读取指定表的所有列名
select name from syscolumns where id=(select max(id) from sysobjects where xtype='u' and name='表名')
```

# 备份 & 还原
```
sqlcmd -Q "BACKUP DATABASE [dbname] TO DISK = N'/<path>/<filename>'"
sqlcmd -Q "RESTORE DATABASE [dbname] FROM DISK = N'/<path>/<filename>'"

/opt/mssql-tools/bin/sqlcmd -S localhost -U SA -Q "RESTORE DATABASE [wms_prod]  FROM DISK = N'/tmp/wms_prod.bak'"
/opt/mssql-tools/bin/sqlcmd -S localhost -U SA -Q "RESTORE DATABASE [wmsauthservice]  FROM DISK = N'/tmp/wmsauthservice.bak'"

```

# 语法
```
sqlcmd
  -U <user>
  -P <password>
  -S <server>
  -d <dbname>
  -q <query>
  -Q <query & exit>
```

# 报错
 
> /opt/mssql/bin/sqlservr: error while loading shared libraries: libldap-2.4.so.2: cannot open shared object file: No such file or directory

```
sudo yum install openldap-clients
```


sudo docker ps -a

sudo docker exec -it mssql01 "bash"
/opt/mssql-tools/bin/sqlcmd

SELECT @@SERVERNAME,
    SERVERPROPERTY('ComputerNamePhysicalNetBIOS'),
    SERVERPROPERTY('MachineName'),
    SERVERPROPERTY('ServerName')