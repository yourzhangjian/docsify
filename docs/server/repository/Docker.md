# Docker安装与使用

### 1、安装docker

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

# 离线安装docker
# 1、卸载旧版本
yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine
# 2、上传docker包至/opt下并安装
cd /opt/docker
yum install -y docker-ce-rootless-extras-24.0.9-1.el7.x86_64.rpm docker-ce-24.0.9-1.el7.x86_64.rpm docker-ce-cli-24.0.9-1.el7.x86_64.rpm containerd.io-1.6.9-3.1.el7.x86_64.rpm docker-buildx-plugin-0.14.1-1.el7.x86_64.rpm docker-compose-plugin-2.6.0-3.el7.x86_64.rpm
docker -v
# 4、启动与自启
systemctl start docker
systemctl enable docker

# 5、卸载docker
yum remove -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
rm -rf /var/lib/docker
rm -rf /var/lib/containerd

# 配置镜像加速、配置docker国内(阿里)加速
# 访问 https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors 获取加速地址
vim /etc/docker/daemon.json
{
  "registry-mirrors": ["https://som51gyv.mirror.aliyuncs.com"],
  "dns": ["8.8.8.8", "8.8.4.4"]
}
systemctl daemon-reload && systemctl restart docker
```

### 2、上传镜像Harbor私服

```bash
# 配置本机hosts, 不用ip访问
vim /etc/hosts
192.168.147.146   registry.harbor.com

# 配置私服信息
{
  "insecure-registries": ["192.168.147.146","registry.harbor.com"]
}
systemctl restart docker

# 登录\推出docker私服
docker login registry.harbor.com -u zhangjian -p Harbor520@
docker login 192.168.147.146 -u zhangjian -p Harbor520@

docker logout registry.harbor.com
docker logout 192.168.147.146:80

# 镜像导出导入
docker save portainer/portainer-ce:2.21.0 | gzip > portainer-ce-2.21.0.tar.gz
docker load < portainer-ce-2.21.0.tar.gz
# 推送镜像到私服
docker tag portainer/portainer-ce:2.21.0 192.168.147.146/registry/portainer-ce:latest
docker tag portainer/portainer-ce:2.21.0 registry.harbor.com/registry/portainer-ce:latest

docker image push 192.168.147.146/registry/portainer-ce:latest
docker image push registry.harbor.com/registry/portainer-ce:latest

# 拉取私服镜像
docker pull 192.168.147.146/goharbor/portainer-ce:2.21.0
docker pull registry.harbor.com/goharbor/portainer-ce:2.21.0
```

