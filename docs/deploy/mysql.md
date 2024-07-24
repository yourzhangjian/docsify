# MySQL下载与部署

### 1、下载地址(自选系统)

> [MySQL官网下载地址](https://downloads.mysql.com/archives/community/)

​	[windows 8.0.21 快捷下载](https://downloads.mysql.com/archives/get/p/23/file/mysql-8.0.21-winx64.zip)

​	[linux 8.0.21 快捷下载](https://downloads.mysql.com/archives/get/p/23/file/mysql-8.0.21-1.el7.x86_64.rpm-bundle.tar)

### 2、Window安装

### 3、Linux安装(centos7)

```bash
# 删除自带mariadb数据库(防止冲突)
rpm -qa | grep mariadb
rpm -e --nodeps mariadb-libs-5.5.68-1.el7.x86_64

# 解压安装包
 mkdir mysql && tar -xvf mysql-8.0.21-1.el7.x86_64.rpm-bundle.tar -C ./mysql
 
 # 进入安装目录安装rpm
 cd mysql/
 
 # 必须安装包(逐个执行)
 rpm -ivh mysql-community-common-8.0.21-1.el7.x86_64.rpm
 rpm -ivh mysql-community-libs-8.0.21-1.el7.x86_64.rpm
 rpm -ivh mysql-community-client-8.0.21-1.el7.x86_64.rpm
 rpm -ivh mysql-community-server-8.0.21-1.el7.x86_64.rpm
 
 # 非必须安装包
 rpm -ivh mysql-community-devel-8.0.21-1.el7.x86_64.rpm
 rpm -ivh mysql-community-embedded-compat-8.0.21-1.el7.x86_64.rpm
 rpm -ivh mysql-community-libs-compat-8.0.21-1.el7.x86_64.rpm
 rpm -ivh mysql-community-test-8.0.21-1.el7.x86_64.rpm
 
 # 可能出错需安装依赖
 yum install perl-JSON -y
 yum install perl-Test-* -y
 
 # 查看服务状态
 systemctl status mysqld
 
 # 启动/停止服务
 systemctl start mysqld
 systemctl stop mysqld
 
 # 初始化数据库
 mysqld --initialize --console
 
 # 目录授权
 chown -R mysql:mysql /var/lib/mysql/
 
 # 查看临时密码
 cat /var/log/mysqld.log
 grep 'temporary password' /var/log/mysqld.log
 
 2024-06-18T15:51:16.283836Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: >xvMKo-+>3Cb

# 重置密码
ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass4!';

ERROR 1819 (HY000): Your password does not satisfy the current policy requirements

# 如果第一次像设置简单点的密码就先执行以下两行
set global validate_password.policy=0;
set global validate_password.length=6;

# 创建远程可读用户(创建|修改|授权)
CREATE USER 'test'@'%' IDENTIFIED WITH mysql_native_password BY 'test';
ALTER USER 'test'@'%' IDENTIFIED WITH mysql_native_password BY 'test';
GRANT SELECT ON *.* TO 'test'@'%';

# 配置文件、数据存储位置
/etc/my.cnf
/var/lib/mysql 

# 数据备份与数据恢复
mysqldump  -u root -p --all-databases > dump.sql
mysqldump  -u root -p --databases db1 db2 db3 > dump.sql
mysqldump  -u root -p test t1 t3 t7 > dump.sql

mysql -u root -p db1 < dump.sql

 
```

