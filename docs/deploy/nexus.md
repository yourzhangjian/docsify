# Nexus安装与使用

```bash
# Maven官方地址：https://maven.apache.org
# Maven搜索地址：https://mvnrepository.com

# maven 中央仓库
https://repo.maven.apache.org/maven2/
https://repo1.maven.org/maven2/

# aliyun maven仓库
https://developer.aliyun.com/mvn/guide
<repository>
  <id>central</id>
  <url>https://maven.aliyun.com/repository/central</url>
  <releases>
    <enabled>true</enabled>
  </releases>
  <snapshots>
    <enabled>true</enabled>
  </snapshots>
</repository>

# huaweicloud maven下载加速网址
https://mirrors.huaweicloud.com/apache/maven/
# huaweicloud maven仓库 https://mirrors.huaweicloud.com/home
https://repo.huaweicloud.com/repository/maven/
<mirror>
    <id>huaweicloud</id>
    <mirrorOf>*</mirrorOf>
    <url>https://mirrors.huaweicloud.com/repository/maven/</url>
</mirror>

# maven maven仓库
# maven settings 私库配置
<server>
  <id>nexus</id>
  <username>admin</username>
  <password>yourzhangjian</password>
</server>

<profile>
  <id>nexus-jdk8</id>
  <activation>
    <jdk>1.8</jdk>
  </activation>
  <repositories>
    <repository>
      <id>nexus</id>
      <name>Nexus私有仓库</name>
      <url>http://127.0.0.1:8088/nexus/repository/maven-releases/</url>
      <layout>default</layout>
      <snapshotPolicy>always</snapshotPolicy>
    </repository>
    <repository>
      <id>maven-huaweicloud</id>
      <name>maven-huaweicloud代理仓库</name>
      <url>http://127.0.0.1:8088/nexus/repository/maven-huaweicloud/</url>
      <layout>default</layout>
      <snapshotPolicy>always</snapshotPolicy>
    </repository>
  </repositories>
</profile>

<activeProfiles>
  <activeProfile>nexus-jdk8</activeProfile>
</activeProfiles>

```



### 1、下载与安装

```bash
# 1、下载安装包并解压 https://help.sonatype.com/en/download.html
# 2、window安装 nexus.exe /run , 其他命令 /start, /stop, /restart, /force-reload and /status	(搭配数据库使用需要license)
# 3、初始化启动好了在这个文件查看密码 sonatype-work/nexus3/admin.password, 登录会重置密码 admin\yourzhangjian
# 4、配置端口 application-port=8081 in$data-dir/etc/nexus.properties
application-port=8088
# 5、配置上下文 update the nexus-context-path property in the $data-dir/etc/nexus.properties
nexus-context-path=/nexus
# 6、4\5两点可以在$data-dir/etc/nexus-default.properties => 更改后访问 http://127.0.0.1:8088/nexus
# 7、设置=>System=>Capabilities=>Outreach:Management=>Disable => 禁用这个,网络问题无法访问,控制台输出报错
```



### 2、构建starters组件

```bash
# 1、构建自定义spring-boot-starter, 标准构建需要autoconfigure 和 starter 两个模块, 名称为：
demo-spring-boot-autoconfigure
demo-spring-boot-starter
# 2、starter组件自动装配 resources => 不同较大版本有所区别 
# 文档 https://docs.spring.io/spring-boot/docs/2.3.2.RELEASE/reference/html/spring-boot-features.html#boot-features-understanding-auto-configured-beans
META-INF/spring.factories
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.mycorp.libx.autoconfigure.LibXAutoConfiguration,\
com.mycorp.libx.autoconfigure.LibXWebAutoConfiguration
# 3、全局扫描组件 @ComponentScan(basePackages = {"io.gitee.yourzhangjian", "org.yourzhangjian"}) 
```



### 3、IDEA发布到私有仓库

```bash
# 手动上传jar包 File Classifier Extension + GAV  => 文件、主类路径、扩展 + GAV坐标
```

###### IDEA部署组件

```bash
# 在maven配置文件settings设置Nexus用户,需要部署与拉取的id值要和当前id一致
<server>
  <id>nexus</id>
  <username>admin</username>
  <password>yourzhangjian</password>
</server>
```

```java
// 1、配置仓库发布私有仓库地址
<distributionManagement>
    <!-- Nexus部署策略 Hosted->Deployment policy->Allow redeploy -->
    <repository>
        <id>nexus</id>
        <name>私有仓库-正式版本</name>
        <url>http://127.0.0.1:8088/nexus/repository/maven-releases/</url>
    </repository>
    <snapshotRepository>
        <id>nexus</id>
        <name>私有仓库-快照版本</name>
        <url>http://127.0.0.1:8088/nexus/repository/maven-snapshots/</url>
    </snapshotRepository>
</distributionManagement>
// 使用IDEA mvn deploy, 制作starters不要在里面写build打包工具, 不然打包出来用不了
<!--  这里不要用打包工具, 不然install打包出来会有问题  -->
<!--    <build>-->
<!--        <finalName>${project.artifactId}</finalName>-->
<!--        <plugins>-->
<!--            <plugin>-->
<!--                <groupId>org.springframework.boot</groupId>-->
<!--                <artifactId>spring-boot-maven-plugin</artifactId>-->
<!--            </plugin>-->
<!--        </plugins>-->
<!--    </build>-->

// 2、配置私有仓库拉取地址
<repositories>
    <repository>
        <id>nexus</id>
        <name>私有仓库</name>
        <url>http://127.0.0.1:8088/nexus/repository/maven-releases/</url>
    </repository>
</repositories>
// 仓库拉取优先级 本地仓库 > 中央仓库 > 私有仓库
        
```

