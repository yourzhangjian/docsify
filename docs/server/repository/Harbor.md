# Harbor安装与使用

### 1、Centos安装Docker

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


# 安装docker
# 1、卸载旧版本
yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine
# 2、设置仓库下载源
# yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo # 官方源(目前不能访问\请使用国内源)
# 安装必要的一些系统工具(阿里下载) yum install -y yum-utils device-mapper-persistent-data lvm2
yum install -y yum-utils
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# 3、下载docker与buildx\compose构建(最新版)
yum install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
docker -v
# 4、启动与自启
systemctl start docker
systemctl enable docker

# 5、卸载docker
yum remove -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
rm -rf /var/lib/docker
rm -rf /var/lib/containerd
```



### 2、Centos安装Harbor

```bash
# 上传harbor至/opt
cd /opt/
tar -zxvf harbor-offline-installer-v2.11.1.tgz && cd harbor

# 配置yml信息, 先./prepare后再./install.sh,数据配置详看当前下的配置 docker-compose.yml 
cp harbor.yml.tmpl harbor.yml
vim harbor.yml

hostname: 192.168.147.146
# 注释https配置(这是需要证书配置的)
harbor_admin_password: zhangjian520@
password: zhangjian520@

# 执行安装脚本(如果已经执行了就执行docker compose命令, 低版本docker使用 docker-compose)
# 192.168.147.146 => admin\zhangjian520@
./install.sh
# docker compose基本命令 => https://docs.docker.com/reference/cli/docker/compose/
docker compose -f docker-compose.yml up -d
docker compose -f docker-compose.yml start
docker compose -f docker-compose.yml restart
docker compose -f docker-compose.yml stop
docker compose -f docker-compose.yml down

# 下载 Portainer 可视化docker界面(已有镜像安装上传至/opt) => https://docs.portainer.io/
docker save portainer/portainer-ce:2.21.0 | gzip > portainer-ce-2.21.0.tar.gz
docker load < portainer-ce-2.21.0.tar.gz
# 运行Portainer, 9000是访问端口
docker volume create portainer_data
docker run -d -p 8000:8000 -p 9443:9443 -p 9000:9000 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:2.21.0
# 访问: 192.168.147.146:9000 => admin\zhangjian520@
# 新建用户 zhangjian\Harbor520@


# 推送镜像
# 配置本机hosts, 不用ip访问
vim /etc/hosts
192.168.147.146   registry.harbor.com
# 配置私服信息
{
  "insecure-registries": ["192.168.147.146","registry.harbor.com"]
}
systemctl restart docker

# 登录\推出docker私服
docker login registry.harbor.com -u admin -p zhangjian520@
docker login 192.168.147.146 -u admin -p zhangjian520@

docker logout registry.harbor.com
docker logout 192.168.147.146:80

# 镜像打标签
docker tag portainer/portainer-ce:2.21.0 registry.harbor.com/goharbor/portainer-ce:2.21.0 
docker tag goharbor/harbor-exporter:v2.11.1 registry.harbor.com/goharbor/harbor-exporter:v2.11.1 
docker tag goharbor/redis-photon:v2.11.1 registry.harbor.com/goharbor/redis-photon:v2.11.1 
docker tag goharbor/trivy-adapter-photon:v2.11.1 registry.harbor.com/goharbor/trivy-adapter-photon:v2.11.1 
docker tag goharbor/harbor-registryctl:v2.11.1 registry.harbor.com/goharbor/harbor-registryctl:v2.11.1 
docker tag goharbor/registry-photon:v2.11.1 registry.harbor.com/goharbor/registry-photon:v2.11.1 
docker tag goharbor/nginx-photon:v2.11.1 registry.harbor.com/goharbor/nginx-photon:v2.11.1 
docker tag goharbor/harbor-log:v2.11.1 registry.harbor.com/goharbor/harbor-log:v2.11.1
docker tag goharbor/harbor-jobservice:v2.11.1 registry.harbor.com/goharbor/harbor-jobservice:v2.11.1
docker tag goharbor/harbor-core:v2.11.1 registry.harbor.com/goharbor/harbor-core:v2.11.1
docker tag goharbor/harbor-portal:v2.11.1 registry.harbor.com/goharbor/harbor-portal:v2.11.1 
docker tag goharbor/harbor-db:v2.11.1 registry.harbor.com/goharbor/harbor-db:v2.11.1
docker tag goharbor/prepare:v2.11.1 registry.harbor.com/goharbor/prepare:v2.11.1


# 推送镜像
docker image push registry.harbor.com/goharbor/portainer-ce:2.21.0
docker image push registry.harbor.com/goharbor/harbor-exporter:v2.11.1 
docker image push registry.harbor.com/goharbor/redis-photon:v2.11.1
docker image push registry.harbor.com/goharbor/trivy-adapter-photon:v2.11.1
docker image push registry.harbor.com/goharbor/harbor-registryctl:v2.11.1 
docker image push registry.harbor.com/goharbor/registry-photon:v2.11.1
docker image push registry.harbor.com/goharbor/nginx-photon:v2.11.1
docker image push registry.harbor.com/goharbor/harbor-log:v2.11.1
docker image push registry.harbor.com/goharbor/harbor-jobservice:v2.11.1
docker image push registry.harbor.com/goharbor/harbor-core:v2.11.1 
docker image push registry.harbor.com/goharbor/harbor-portal:v2.11.1
docker image push registry.harbor.com/goharbor/harbor-db:v2.11.1
docker image push registry.harbor.com/goharbor/prepare:v2.11.1


```

