# Nexus安装与使用

### 1、Centos安装Nexus

```bash
# centos7系统下载源换成阿里源 (https://developer.aliyun.com/mirror/centos)
cd /etc/yum.repos.d
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
# wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
# wget无法用就离线下载上传至/etc/yum.repos.d, 并改名 
mv Centos-7.repo CentOS-Base.repo
# 运行生成缓存
yum makecache
# 下载vim文本编辑工具
yum install -y vim

# 防火墙配置
systemctl status firewalld
systemctl stop firewalld
systemctl start firewalld
systemctl restart firewalld
systemctl disable firewalld
systemctl enable firewalld

# 配置Java环境(Java包放至/opt下)
tar -zxvf jdk-8u202-linux-x64.tar.gz 
export JAVA_HOME=/opt/jdk1.8.0_202
export PATH=$JAVA_HOME/bin:$PATH

vim /etc/profile
source /etc/profile
java -version

# 配置Nexus(Nexus包放至/opt下)
mkdir nexus3
tar -zxvf nexus-3.70.2-01-java8-unix.tar.gz -C ./nexus3

vim /opt/nexus3/nexus-3.70.2-01/etc/nexus-default.properties
application-port=8083
nexus-context-path=/nexus3

# nexus脚本可以作为OSX和Unix上的后台应用程序，通过start、stop、restart、force-reload和status命令来管理存储库管理器
# 从bin文件夹下的应用程序目录启动存储库管理器
./nexus run
# 启动存储库管理器并在后台运行
./nexus start
# 停止存储库管理器在后台运行
./nexus stop

# 访问
http://192.168.147.145:8083/nexus3/
# 打印初始密码
cat /opt/nexus3/sonatype-work/nexus3/admin.password
# 登录并修改密码 admin\c03371dd-92ca-499c-b90b-6223052f9d68
admin\zhangjian520@
# 禁用规则并重启(访问其它外网导致后台报错)
设置图标 -> System -> Capabilities -> Outreach: Management -> Disable

# 添加代理Maven仓库, maven-* 标识
https://repo.maven.apache.org/maven2/
https://maven.aliyun.com/repository/public
https://mirrors.huaweicloud.com/repository/maven/

# 未设置开机自启, 重启系统后须./nexus start后台启动
```



### 2、Maven 配置文件 settings.xml

```xml
<!-- Since Maven 3.8.1 http repositories are blocked. -->
<!-- 3.9.7 注释mirror, 一切下载从私有仓库或私有代理仓库下载; IDEA未关闭配置后需要重启下
<mirror>
  <id>maven-default-http-blocker</id>
  <mirrorOf>external:http:*</mirrorOf>
  <name>Pseudo repository to mirror external repositories initially using HTTP.</name>
  <url>http://0.0.0.0/</url>
  <blocked>true</blocked>
</mirror>
 -->

<!-- 1、配置localRepository信息 -->
<localRepository>D:\Maven\repository</localRepository>

<!-- 2、配置servers信息 -->
<server>
  <id>nexus</id>
  <username>admin</username>
  <password>zhangjian520@</password>
</server>

<!-- 3、配置profiles信息 -->
<profile>
  <id>nexus-jdk8</id>

  <activation>
    <jdk>1.8</jdk>
  </activation>

  <repositories>
    <repository>
      <id>maven-releases</id>
      <name>Nexus私有仓库</name>
      <url>http://192.168.147.145:8083/nexus3/repository/maven-releases/</url>
      <layout>default</layout>
      <snapshotPolicy>always</snapshotPolicy>
    </repository>
    <!-- 私有仓库找不到的组件就在代理仓库找, 测试只用私有仓库(基本组件存于本地仓库) -->
    <!-- 
    <repository>
      <id>maven-apache</id>
      <name>Nexus代理仓库</name>
      <url>http://192.168.147.145:8083/nexus3/repository/maven-apache/</url>
      <layout>default</layout>
      <snapshotPolicy>always</snapshotPolicy>
    </repository>
    -->
  </repositories>
</profile>

<!-- 4、配置activeProfiles信息, 启用某一规则 -->
<activeProfile>nexus-jdk8</activeProfile>
```



### 3、IDEA推送组件到私有仓库

```bash
# 配置私有仓库发布地址
# <id>nexus</id> 要与配置文件的 server id 对应
<distributionManagement>
    <repository>
        <id>nexus</id>
        <name>私有仓库-正式版本</name>
        <url>http://192.168.147.145:8083/nexus3/repository/maven-releases/</url>
    </repository>
    <snapshotRepository>
        <id>nexus</id>
        <name>私有仓库-快照版本</name>
        <url>http://192.168.147.145:8083/nexus3/repository/maven-snapshots/</url>
    </snapshotRepository>
</distributionManagement>

# 发布组件
# 使用IDEA中的 deploy 发布
# 使用mvn命令 D:\Maven\apache-maven-3.9.7-nexus\bin\mvn deploy 发布

# 使用IDEA mvn deploy, 制作starters不要在里面写build打包工具, 不然打包出来用不了
```

#### 

### 4、IDEA拉取组件

```bash
# 配置私有仓库拉取地址(直接更新pom就行)
<repositories>
    <repository>
        <id>nexus</id>
        <name>私有仓库-正式版本</name>
        <url>http://192.168.147.145:8083/nexus3/repository/maven-releases/</url>
    </repository>
</repositories>

# 使用maven工具 mvn部署现有组件到私有仓库(单个jar包使用)
mvn deploy:deploy-file -DgroupId=<group-id> \
-DartifactId=<artifact-id> \
-Dversion=<version> \
-Dpackaging=<type-of-packaging> \
-Dfile=<path-to-file> \
-DrepositoryId=<id-to-map-on-server-section-of-settings.xml> \
-Durl=<url-of-the-repository-to-deploy>
# 默认情况下，deploy:deploy-file生成一个通用POM,禁用设置为false
-DgeneratePom=false
# 如果第三方JAR的POM已经存在，并且您希望将其与JAR一起部署，我们应该使用部署文件目标的pomFile参数(下载的组件推荐使用)
mvn deploy:deploy-file -DpomFile=<path-to-pom> \
-Dfile=<path-to-file> \
-DrepositoryId=<id-to-map-on-server-section-of-settings.xml> \
-Durl=<url-of-the-repository-to-deploy>

# 仓库拉取优先级 本地仓库 > 中央仓库 > 私有仓库
# 比如 ojdbc6-11.2.0.3.jar 、sqljdbc4-4.0.jar 这两个组件在中央仓库找不到了, 在其它博客上下载而来
<dependency>
    <groupId>com.microsoft.sqlserver</groupId>
    <artifactId>sqljdbc4</artifactId>
    <version>4.0</version>
</dependency>
<dependency>
    <groupId>com.oracle</groupId>
    <artifactId>ojdbc6</artifactId>
    <version>11.2.0.3</version>
</dependency>
# 本地安装方法
mvn install:install-file -DgroupId=com.oracle -DartifactId=ojdbc6 -Dversion=11.2.0.3 -Dpackaging=jar -Dfile=D:\Maven\ojdbc6-11.2.0.3.jar
mvn install:install-file -DgroupId=com.microsoft.sqlserver -DartifactId=sqljdbc4 -Dversion=4.0 -Dpackaging=jar -Dfile=D:\Maven\sqljdbc4-4.0.jar

# 1、测试 单个组件jar包上传(sqljdbc4)、删除原有组件后下载
mvn deploy:deploy-file -DgroupId=com.microsoft.sqlserver -DartifactId=sqljdbc4 -Dversion=4.0 -Dpackaging=jar -Dfile=D:\Maven\sqljdbc4-4.0.jar -DrepositoryId=nexus -Durl=http://192.168.147.145:8083/nexus3/repository/maven-releases/

# 2、测试 本地组件jar、pom上传(ojdbc6)、这种情况就在本地仓库存在的组件上传(如果没有就使用本地安装方法执行、将后缀为jar和pom的文件复制出来、删除原有组件)
mvn deploy:deploy-file -DpomFile=D:\Maven\ojdbc6-11.2.0.3.pom -Dfile=D:\Maven\ojdbc6-11.2.0.3.jar -DrepositoryId=nexus -Durl=http://192.168.147.145:8083/nexus3/repository/maven-releases/
```



### 5、IDEA推送组件到中央仓库

```bash
# Ctrl+Shift+R 全局替换
# 配置 javadoc 注释模板: IDEA => Editor => File and Code Templates 
/**
 * {@code @author:} yourzhangjian
 * {@code @createDate:} ${YEAR}-${MONTH}-${DAY} ${HOUR}:${MINUTE}:${SECOND}
 * {@code @desc:} ${NAME}
 */

# 文档中心 https://central.sonatype.org/publish/requirements/
# 1、注册中央仓库账号(自行注册)
https://central.sonatype.com/
# 2、创建命令空间, 需要有相应的账号如 GitHub\GitLab\Gitee\Bitbucket\... 的用户名(看文档创建就行)
io.github.myusername\io.gitlab.myusername\io.gitee.myusername\io.bitbucket.myusername\...
# 3、验证命令空间, 创建好了命令空间会有一个"Verification Key", 复制它并在相应的账号创建开源Key项目,验证大概一天时间,若长时间没成功就有问题了
# 4、你的开源项目groupId要规范 如<groupId>io.gitee.yourzhangjian</groupId>
# 5、Maven settings.xml 创建凭证
https://central.sonatype.com/account
账号 => View Account => Generate User Token => settings.xml填写
# 6、 创建您自己的密钥对并将其分发到密钥服务器，以便用户可以验证它 https//www.gnupg.org/download/ 
# 查看版本
gpg --version
# 生成密钥对 Real name: sonatype邮箱 => Email address：sonatype邮箱 => 确认O
# 确认后会让你输入密码 都填sonatype密码
gpg --gen-key
# 密钥对清单
gpg --list-keys
# 发布Public Key keyserver.ubuntu.com\keys.openpgp.org\pgp.mit.edu
gpg --keyserver keyserver.ubuntu.com --send-keys AE8D5EB2F8B3F4EBAF091959940BB0E596706198

# 7、发布依赖要求 详情看文档 https://central.sonatype.org/publish/requirements/#webhook-request-details
Supply Javadoc and Sources
Provide Files Checksums
Sign Files with GPG/PGP
Sufficient Metadata
Correct Coordinates
Project Name, Description and URL
License Information
Developer Information
SCM Information
# 8、发布
```

### 完整的一个pom文件, 稍加修改就可以使用

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>io.gitee.yourzhangjian</groupId>
    <artifactId>mybatis-actable-spring-boot</artifactId>
    <version>1.0.0</version>

    <name>${project.groupId}:${project.artifactId}</name>
    <description>基于java8+springboot2+mybatis+actable构建基础ERP项目CRUD依赖!</description>
    <url>https://gitee.com/yourzhangjian/mybatis-actable-spring-boot</url>

    <licenses>
        <license>
            <name>MIT License</name>
            <url>http://www.opensource.org/licenses/mit-license.php</url>
        </license>
    </licenses>

    <developers>
        <developer>
            <name>yourzhangjian</name>
            <email>507668447@qq.com</email>
            <organization>Gitee</organization>
            <organizationUrl>https://gitee.com</organizationUrl>
        </developer>
    </developers>

    <scm>
        <connection>scm:git:git://gitee.com/yourzhangjian/mybatis-actable-spring-boot.git</connection>
        <developerConnection>scm:git:ssh://gitee.com/yourzhangjian/mybatis-actable-spring-boot.git</developerConnection>
        <url>http://gitee.com/yourzhangjian/mybatis-actable-spring-boot/tree/master</url>
    </scm>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.2.RELEASE</version>
    </parent>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <!-- https://repo1.maven.org/maven2/ 中央仓库地址(默认加载依赖) -->
    <!-- https://central.sonatype.com/search  中央仓库依赖地址-->
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

<!--    私有仓库发布 -->
    <distributionManagement>
        <repository>
            <id>nexus-ados</id>
            <name>私有仓库-正式版本</name>
            <url>http://47.108.187.98:5555/nexus/repository/ados/</url>
        </repository>
        <snapshotRepository>
            <id>nexus-ados</id>
            <name>私有仓库-快照版本</name>
            <url>http://47.108.187.98:5555/nexus/repository/ados-snapshots/</url>
        </snapshotRepository>
    </distributionManagement>

<!--    远程仓库发布 -->
<!--    <build>-->
<!--        <plugins>-->
<!--            <plugin>-->
<!--                <groupId>org.apache.maven.plugins</groupId>-->
<!--                <artifactId>maven-source-plugin</artifactId>-->
<!--                <version>3.2.1</version>-->
<!--                <executions>-->
<!--                    <execution>-->
<!--                        <id>attach-sources</id>-->
<!--                        <goals>-->
<!--                            <goal>jar-no-fork</goal>-->
<!--                        </goals>-->
<!--                    </execution>-->
<!--                </executions>-->
<!--            </plugin>-->
<!--            <plugin>-->
<!--                <groupId>org.apache.maven.plugins</groupId>-->
<!--                <artifactId>maven-javadoc-plugin</artifactId>-->
<!--                <version>3.4.1</version>-->
<!--                <executions>-->
<!--                    <execution>-->
<!--                        <id>attach-javadocs</id>-->
<!--                        <goals>-->
<!--                            <goal>jar</goal>-->
<!--                        </goals>-->
<!--                    </execution>-->
<!--                </executions>-->
<!--            </plugin>-->
<!--            <plugin>-->
<!--                <groupId>org.apache.maven.plugins</groupId>-->
<!--                <artifactId>maven-gpg-plugin</artifactId>-->
<!--                <version>3.0.1</version>-->
<!--                <executions>-->
<!--                    <execution>-->
<!--                        <id>sign-artifacts</id>-->
<!--                        <phase>verify</phase>-->
<!--                        <goals>-->
<!--                            <goal>sign</goal>-->
<!--                        </goals>-->
<!--                    </execution>-->
<!--                </executions>-->
<!--            </plugin>-->
<!--            <plugin>-->
<!--                <groupId>org.sonatype.central</groupId>-->
<!--                <artifactId>central-publishing-maven-plugin</artifactId>-->
<!--                <version>0.5.0</version>-->
<!--                <extensions>true</extensions>-->
<!--                <configuration>-->
<!--                    <publishingServerId>central</publishingServerId>-->
<!--                    &lt;!&ndash; Automatic Publishing 自动发布, 直接帮你发布到仓库(新手不建议设置true) &ndash;&gt;-->
<!--                    <autoPublish>false</autoPublish>-->
<!--                    &lt;!&ndash; Wait for Publishing 等待部署被上传、验证和发布; 上传、验证或发布的失败将出现在控制台结果输出中; 取决于autoPublish=true &ndash;&gt;-->
<!--                    <waitUntil>published</waitUntil>-->
<!--                </configuration>-->
<!--            </plugin>-->
<!--        </plugins>-->
<!--    </build>-->
</project>
```

