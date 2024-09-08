# Centos7 安装oracle 19c

## 1、oracle 19c 下载安装(使用rpm方式安装)

**1.1、下载oracle-database-preinstall并安装**

```bash
curl -o oracle-database-preinstall-19c-1.0-1.el7.x86_64.rpm https://yum.oracle.com/repo/OracleLinux/OL7/latest/x86_64/getPackage/oracle-database-preinstall-19c-1.0-1.el7.x86_64.rpm

yum -y localinstall oracle-database-preinstall-19c-1.0-1.el7.x86_64.rpm
```



**1.2、安装好了可以删除这个rpm包**

```bash
rm oracle-database-preinstall-19c-1.0-1.el7.x86_64.rpm
```



**1.3、此页面下载oracle rpm包**

[**https://www.oracle.com/database/technologies/oracle-database-software-downloads.html**](https://www.oracle.com/database/technologies/oracle-database-software-downloads.html)



**1.4、安装 oracle 数据库**

```bash
yum -y localinstall oracle-database-ee-19c-1.0-1.x86_64.rpm
```



**1.5、初始化配置**

```bash
vi /etc/init.d/oracledb_ORCLCDB-19c

export CREATE_AS_CDB=false （这个地方改为false，这是问你要不要创建容器化数据库，如果是true以后创建的用户好像是都要加c##）
```



**执行配置初始化**

```bash
/etc/init.d/oracledb_ORCLCDB-19c configure   
```



**完成后会输出(这个过程会有点久、十分钟左右)**

This script creates a container database (ORCLCDB) with one pluggable database (ORCLPDB1) and configures the listener at the default port (1521).



**1.6、以上脚本在root用户下执行**

## 2、oracle 19c 环境变量配置及sqlplus登录

**1.1、配置环境变量**

**切换到oracle账户  su oracle**



**编辑环境变量**

export ORACLE_BASE=/opt/oracle

export ORACLE_HOME=/opt/oracle/product/19c/dbhome_1

export ORACLE_SID=ORCLCDB

export PATH=$ORACLE_HOME/bin:$PATH:$HOME/.local/bin:$HOME/bin



**刷新环境变量**

source ~/.bash_profile



**1.2、oracle登录及创建用户、数据表**

**登录(这个要执行上面的环境、不然找不到sqlplus命令)**

sqlplus / as sysdba



**查看已启用的用户**

select username from dba_users where account_status='OPEN';



**修改用户密码**

alter user sys identified by 密码;



**创建用户|用户添加权限 => 创建用户就是创建一个新的数据库**

create user oracle identified by 123456;

grant connect,resource,create session,unlimited tablespace to oracle;

授予权限：grant 权限 to 用户名;

撤销权限：revoke 权限 from 用户名;



**删除用户**

drop user test;



**创建数据表**

create table ados.test(id number primary key not null);



**查询所有表、当前用户的表**

select table_name from all_tables;

select table_name from user_tables;

显示当前用户 show user; 



**1.3、卸载数据库**

yum -y remove oracle-database-ee-19c.x86_64



## 3、centos7 重启 oracle (手动启动)

**linux重启后oracle数据库服务未启动**

**su oracle 刷新环境变量**

**source ~/.bash_profile**



**执行脚本方法**

（1） 以oracle身份登录数据库，命令：su -oracle

（2） 进入Sqlplus控制台，命令：sqlplus /nolog

（3） 以系统管理员登录，命令：connect / as sysdba

（4） 启动数据库，命令：startup

（5） 如果是关闭数据库，命令：shutdown immediate

（6） 退出sqlplus控制台，命令：exit

（7） 进入监听器控制台，命令：lsnrctl

（8） 启动监听器，命令：start

（9） 退出监听器控制台，命令：exit



**如果要远程登录数据库, 就关闭防火墙（学习就全部关闭）**

**systemctl stop firewalld**

**systemctl disable firewalld**

## 4、actable使用oracle数据库

**截至最新版本1.5.0的actable目前都只支持mysql数据， oracle等其它数据库暂不支持!**

**看源码...**

@PostConstruct

 public void startHandler() {

​        databaseType = this.springContextUtil.getConfig("mybatis.database.type") == null ? MYSQL : this.springContextUtil.getConfig("mybatis.database.type");

​        if (MYSQL.equals(databaseType)) {

​            log.info("databaseType=mysql，开始执行mysql的处理方法");

​            this.sysMysqlCreateTableManager.createMysqlTable();

​        } else if (ORACLE.equals(databaseType)) {

​            log.info("databaseType=oracle，开始执行oracle的处理方法");

​        } else if (SQLSERVER.equals(databaseType)) {

​            log.info("databaseType=sqlserver，开始执行sqlserver的处理方法");

​        } else if (POSTGRESQL.equals(databaseType)) {

​            log.info("databaseType=postgresql，开始执行postgresql的处理方法");

​        } else {

​            log.info("没有找到符合条件的处理方法！");

​        }



​    }

 **目前只对mysql 执行了建表方法 this.sysMysqlCreateTableManager.createMysqlTable();**

**源码中oracle的代码设计了部分、有兴趣可以参照mysql来编写oracle的建表方法(下载官方actable源码)**