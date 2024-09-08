# GitLab安装与使用

### 1、Centos安装GitLab (不支持window系统)

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

# 官方库下载gitlab-ce
https://packages.gitlab.com/gitlab/gitlab-ce
# 安装指南
https://docs.gitlab.com/17.2/omnibus/installation/
# 安装要求
https://docs.gitlab.com/17.2/ee/install/requirements.html
使用2核4G虚拟机安装供个人使用

# rpm安装
yum install -y policycoreutils-python
rpm -Uvh gitlab-ce-17.2.4-ce.0.el7.x86_64.rpm

# 安装后提示配置文件位置和重新配置命令, 其他配置 https://docs.gitlab.com/17.2/omnibus/settings/
/etc/gitlab/gitlab.rb
gitlab-ctl reconfigure

# 配置外部访问(有域名可配置域名)
external_url "http://192.168.147.132"

# gitlab-ctl start服务启动后打印初始密码(或者在/etc/gitlab/gitlab.rb配置gitlab_rails['initial_root_password'] = '<my_strong_password>')
cat /etc/gitlab/initial_root_password
# 登录用户名+密码
root
EfnCwg4vokgL2jQnBfJ/VRbJRTFid1/EeGs74Wcu3c4=
# 改密码 => zhangjian520@ => 不改24小时后会重置
# 创建 测试管理员: test\zhangjian520@
# 注册 张剑: zhangjian\your520@
# 注册的用户默认为中文界面, 管理添加的用户为英文界面, 修改语言
# 中文界面: 头像 -> 偏好设置 -> 本地化(语言) => 选择
# 英文界面: 头像 -> Preferences -> Localization(Language) => 选择
# 不允许用户注册设置: Admin area -> Settings -> General -> Sign-up restrictions -> Sign-up enabled => Save changes

# 基本操作命令 (系统重启默认自启GitLab, 需要一些时间启动)
gitlab-ctl status
gitlab-ctl start
gitlab-ctl stop
gitlab-ctl restart

# 卸载指南(Uninstall the Linux package) => https://docs.gitlab.com/17.2/omnibus/installation/
```



### 2、IDEA推送代码

```bash
# IDEA配置新加GitLab账户, Generate会自动打开网页
http://192.168.147.132/
# 添加新的令牌, 复制 个人访问令牌(有期限限制)

# GitLab新建项目仓库, 推送现有文件夹
cd existing_folder
git init --initial-branch=master
git remote add origin http://192.168.147.132/zhangjian/nexus-spring-boot-starter.git
git add .
git commit -m "Initial commit"
git push --set-upstream origin master

# 直接使用IDEA的push即可
```

