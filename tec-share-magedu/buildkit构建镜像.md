# 0、容器技术简介

容器技术除了的docker之外，还有coreOS的rkt、google的gvisor、以及docker开源的containerd、redhat的podman、阿⾥的pouch等，为了保证容器⽣态的标准性和健康可持续发展，包括Linux 基⾦会、Docker、微软、红帽、⾕歌和IBM等公司在2015年6⽉共同成⽴了⼀个叫open container（OCI）的组织，其⽬的就是制定开放的标准的容器规范，⽬前OCI⼀共发布了两个规范，分别是runtime spec和image format spec，有了这两个规范，不同的容器公司开发的容器只要兼容这两个规范，就可以保证容器的可移植性和相互可操作性。 containerd官网：[](https://www.1024sky.cn/blog/article/79698)[https://containerd.io/](https://containerd.io/)gvisor官网：[](https://www.1024sky.cn/blog/article/79698)[https://gvisor.dev/](https://gvisor.dev/)podman官网：[](https://www.1024sky.cn/blog/article/79698)[https://podman.io](https://podman.io)pouch项目地址：[](https://www.1024sky.cn/blog/article/79698)[https://github.com/alibaba/pouch](https://github.com/alibaba/pouch)buildkit: 从Docker公司的开源出来的⼀个镜像构建⼯具包，⽀持OCI标准的镜像构建,项目地址[](https://www.1024sky.cn/blog/article/79698)[https://github.com/moby/buildkit](https://github.com/moby/buildkit)

google kaniko

docker 大一统的工具 镜像打包，镜像管理，容器管理 第一代构建技术

buildkitd 第二代构建技术 contanerd 镜像管理，pull push run

# 1、containerd部署安装

## 1.1 安装

```Plain
<https://github.com/containerd/containerd/releases/download/v1.7.3/cri-containerd-cni-1.7.3-linux-amd64.tar.gz>

tar -C / -xvf cri-containerd-cni-1.7.3-linux-amd64.tar.gz

```

## 1.2 配置

Containerd 的默认配置文件为 /etc/containerd/config.toml，我们可以通过如下所示的命令生成一个默认的配置：

```Plain
# mkdir -p /etc/containerd
# containerd config default > /etc/containerd/config.toml
```

对于使用 systemd 作为 init system 的 Linux 的发行版，使用 systemd 作为容器的 cgroup driver 可以确 保节点在资源紧张的情况更加稳定，所以推荐将 containerd 的 cgroup driver 配置为 systemd。

修改前面生成的配置文件 `/etc/containerd/config.toml`，在`plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options`配置块下面将`SystemdCgroup`设置为 true.

然后再为镜像仓库配置一个加速器，需要在 cri 配置块下面的 registry 配置块下面进行配置 registry.mirrors ：

`Plain Text [plugins."io.containerd.grpc.v1.cri"] ... # sandbox_image = "registry.k8s.io/pause:3.6" sandbox_image = "registry.aliyuncs.com/k8sxio/pause:3.8"`

## 1.3 启动

`Plain Text [root@VM-0-16-centos ~]# systemctl daemon-reload [root@VM-0-16-centos ~]# systemctl enable --now containerd`

# 2、buildkitd组成部分

> buildkitd(服务端)，⽬前⽀持runc和containerd作为镜像构建环境，默认是runc，可以更换为containerd。 buildctl(客户端)，负责解析Dockerfile⽂件，并向服务端buildkitd发出构建请求。

# 3、部署buildkitd

## 3.1、下载二进制包

```
wget <https://github.com/moby/buildkit/releases/download/v0.12.0/buildkit-v0.12.0.linux-amd64.tar.gz>
```

**解压压缩包，将二进制文件软连接至path环境变量**

```
root@k8s-master01:/usr/local/src# ls
buildkit-v0.11.6.linux-amd64.tar.gz
root@k8s-master01:/usr/local/src# tar xf buildkit-v0.11.6.linux-amd64.tar.gz
root@k8s-master01:/usr/local/src# ls
bin  buildkit-v0.11.6.linux-amd64.tar.gz
root@k8s-master01:/usr/local/src# cd bin
root@k8s-master01:/usr/local/src/bin# ls
buildctl               buildkit-qemu-arm   buildkit-qemu-mips64    buildkit-qemu-ppc64le  buildkit-qemu-s390x  buildkitd
buildkit-qemu-aarch64  buildkit-qemu-i386  buildkit-qemu-mips64el  buildkit-qemu-riscv64  buildkit-runc
root@k8s-master01:/usr/local/src/bin# ln -s /usr/local/src/bin/* /usr/local/bin/

```

> 能够正常在bash中执行buildkit –help ，表示对应命令已经正常软连接至path环境中。

## 3.2、提供buildkit.socket文件

```
root@k8s-master01:/usr/local/src/bin# cat /lib/systemd/system/buildkit.socket
[Unit]
Description=BuildKit
Documentation=https://github.com/moby/buildkit
[Socket]
ListenStream=%t/buildkit/buildkitd.sock
[Install]
WantedBy=sockets.target
```

## 3.3、提供buildkit.service文件

```
root@k8s-master01:/usr/local/src/bin# cat /lib/systemd/system/buildkitd.service
[Unit]
Description=BuildKit
Requires=buildkit.socket
After=buildkit.socketDocumentation=https://github.com/moby/buildkit
[Service]
ExecStart=/usr/local/bin/buildkitd --oci-worker=false --containerd-worker=true
[Install]
WantedBy=multi-user.target
root@k8s-master01:/usr/local/src/bin#
```

## 3.4、启动buildiktd服务

```
root@k8s-master01:/usr/local/src/bin# systemctl daemon-reload
root@k8s-master01:/usr/local/src/bin# systemctl enable buildkitd
Created symlink /etc/systemd/system/multi-user.target.wants/buildkitd.service → /lib/systemd/system/buildkitd.service.
root@k8s-master01:/usr/local/src/bin# systemctl restart buildkitd
root@k8s-master01:/usr/local/src/bin# systemctl status buildkitd
```

# 4、基于⾃定义镜像创建测试容器

4.1 安装nerdctl

`Plain Text wget <https://github.com/containerd/nerdctl/releases/download/v1.4.0/nerdctl-1.4.0-linux-amd64.tar.gz`>

## 4.1、nerdctl命令

```
root@k8s-node01:~# nerdctl run -d -p 80:80 harbor.magedu.net/magedu/nginx-base:1.22.0
WARN[0000] skipping verifying HTTPS certs for "harbor.magedu.net"
root@k8s-node01:~#
```