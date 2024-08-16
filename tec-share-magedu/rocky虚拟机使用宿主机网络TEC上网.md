## 安装docker

 本部分安装使用的阿里云源进行安装， 参考链接请移步文章底部
```shell
# step 1: 安装必要的一些系统工具
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
# Step 2: 添加软件源信息
sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# Step 3
sudo sed -i 's+download.docker.com+mirrors.aliyun.com/docker-ce+' /etc/yum.repos.d/docker-ce.repo
# Step 4: 更新并安装Docker-CE
sudo yum makecache fast
sudo yum -y install docker-ce
# Step 4: 开启Docker服务
sudo service docker start

# 注意：
# 官方软件源默认启用了最新的软件，您可以通过编辑软件源的方式获取各个版本的软件包。例如官方并没有将测试版本的软件源置为可用，您可以通过以下方式开启。同理可以开启各种测试版本等。
# vim /etc/yum.repos.d/docker-ce.repo
#   将[docker-ce-test]下方的enabled=0修改为enabled=1
#
# 安装指定版本的Docker-CE:
# Step 1: 查找Docker-CE的版本:
# yum list docker-ce.x86_64 --showduplicates | sort -r
#   Loading mirror speeds from cached hostfile
#   Loaded plugins: branch, fastestmirror, langpacks
#   docker-ce.x86_64            17.03.1.ce-1.el7.centos            docker-ce-stable
#   docker-ce.x86_64            17.03.1.ce-1.el7.centos            @docker-ce-stable
   docker-ce.x86_64            17.03.0.ce-1.el7.centos            docker-ce-stable
#   Available Packages
# Step2: 安装指定版本的Docker-CE: (VERSION例如上面的17.03.0.ce.1-1.el7.centos)
# sudo yum -y install docker-ce-[VERSION]

```

## windows宿主机启动v2ray

![](http://cdn.oyfacc.cn/20240727112043.png)

![](http://cdn.oyfacc.cn/20240727112458.png)

对应的在代理配置的底部，可以看到局域网监听的端口，等下需要用到这个端口

![](http://cdn.oyfacc.cn/20240727112528.png)

## 配置VM的网络

![](http://cdn.oyfacc.cn/20240727112845.png)

![](http://cdn.oyfacc.cn/20240727112902.png)

![](http://cdn.oyfacc.cn/20240727112934.png)

![](http://cdn.oyfacc.cn/20240727113210.png)

需要记住这个vm net8的地址，等下需要用到

## 配置宿主机代理并且测试

```shell

这里是我本地的地址，上面的图片的88.1是复制的图片，供参考步骤

export https_proxy="http://192.168.158.1:10811"
export http_proxy="http://192.168.158.1:10811"
curl www.google.com
## 返回就成功了
<!doctype html><html itemscope="" itemtype="http://schema.org/WebPage" ..
```


## 配置docker的代理
```shell
# unit单元添加配置，一定要按照这个配置，config.json文件亲测不生效
vi  /usr/lib/systemd/system/docker.service

[Service]
Environment="HTTPS_PROXY=http://192.168.158.1:10811"
Environment="NO_PROXY="
Environment="HTTP_PROXY=http://192.168.158.1:10811"

#重载配置
systemctl daemon-reload
systemctl restart docker

## 测试

docker pull nginx
Using default tag: latest
latest: Pulling from library/nginx
.....
0f11e17345c5: Pull complete 
Digest: sha256:6af79ae5de407283dcea8b00d5c37ace95441fd58a8b1d2aa1ed93f5511bb18c
Status: Downloaded newer image for nginx:latest

```

##
参考链接
https://developer.aliyun.com/mirror/docker-ce?spm=a2c6h.13651102.0.0.57e31b11elMDYv
https://blog.csdn.net/csdner250/article/details/137168407
https://www.lfhacks.com/tech/pull-docker-images-behind-proxy/