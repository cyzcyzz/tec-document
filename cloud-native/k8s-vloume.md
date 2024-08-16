---
title: "K8S-Vloume"
date: 2020-05-16T11:20:29+08:00
description: ""
draft: false
tags: [volume]
categories: [容器]
---

# 基本介绍

- 场景一：当我们用K8S创建了一个pod的时候，容器顶部附加了一个可写层用于程序运行时的数据读写，但在容器结束运行的时候，这个可写层也会随之消失，数据也会消失不见，数据无持久化。
- 场景二：每个pod都有自己的文件系统，当本次pod结束后，后创建的pod不能有效识别到前一个pod遗留的数据，数据不共享问题。

K8S通过定义存储卷来解决上述的问题。存储卷本身不是顶级资源，像pod， service那样，而是被定义为pod的一个组成部分，因为不是独立的K8S对象，不能独立的创建和删除，而且pod的所有容器均可共享使用这个卷。

下面一个简单的应用实例，第一个容器有htdocs目录和logs目录，存放html和日志文件，第二个容器运行了一个代理来创建html，第三个容器来收集日志

![volume-1](http://cdn.oyfacc.cn/k8s/vloume-1.png)

增加共享卷以后

![volume-2](http://cdn.oyfacc.cn/k8s/volume-2.png)

webserver写入日志，日志收集器来收集日志，两个容器内的文件系统的logs文件夹应该挂载同一个卷，htdocs和html文件夹，一个生成文件，一个读取文件，也应该使用同一个共享卷。

### 卷类型

K8S中有多种卷的类型，下面的列表可以参考：

| emptDir                                      | 存储临时数据的简单空目录               |
| -------------------------------------------- | -------------------------------------- |
| hostPath～                                   | 用于将目录从work                       |
| emptDir～                                    | 存储临时数据的简单空目录               |
| gitRepo                                      | 检出Git仓库内容来初始化                |
| nfs                                          | 挂载pod中的NFS共享卷                   |
| 谷歌高效存储卷 亚马逊弹性块存储 微软磁盘卷～ | 公有云存储                             |
| cinder cephfs glusterfs～                    | 其他网络类型的存储                     |
| configmap secret～                           | K8S部分资源和集群信息公开给pod的特殊卷 |
| PVC～                                        | 预置或者动态配置的持久存储类型         |

上面有波浪线的是使用类型比较多的卷，一个容器可以挂在多个不通类型的卷到不通的目录上。

### 使用示例

emptDir

```shell
apiVersion: v1
kind: Pod
metadata::
  name: fortune
spec:
  containers;
  - images; luksa/fortune
    name: html-gen
    volumeMounts:
    - name: html # 挂载的卷的名称
      mountPath: /var/htdocs # 这里指明容器内的挂在位置
      .......
   volumes:
   - name: html   # 这里定义了一个卷
     emptyDir: {}  
```

hostPath

这种类型属于比较常用的，我们生产线上都是挂载hostpath进去，写日志，ds filebeat去收集本机的日志到某一个日志机上

```shell
 volumeMounts:
 - mountPath: /home/logs # 挂载位置
   name: hostpath # 卷名称
      ....
      volumes:
        - name: hostpath #卷名称
          hostPath:
            path: /www-volume/logs # 主机路径
```

## PV&&PVC

之前的两种类型都需要pod的开发人员了解集群中的真实网络存储的基础设施，这对专业开发人员并不友好，理想的状况是，开发人员不需要知道这些底层设施，当我需要时，申请就可以了。 这就是pv和pvc的作用。

![pv](http://cdn.oyfacc.cn/2020-05-23-PVandPVC.png)

```shell
# 定义持久卷
apiVersion: v1
kind: persistentVolume
metadata:
  name: mongodb-pv
spec:
  capacity:
    storage: 1Gi  # 大小
  accessModes: # 可以被单个客户端挂载
  - ReadWriteOnce
  - ReadOnlyMany
  persistentVolumeReclimPolicy: Retain # 声明被释放后，pv将会保留
  gcePersistentDisk:
    pdName: mongodb
    fstype: ext4
```

持久卷不属于任何名称空间，是集群层面的资源，持久卷声明是属于某一个名称空间的资源，不能跨名称空间调用

```shell
# pvc
apiVersion: v1
kind: pVC #这里应该写全称
metadata: 
  name: monfodb-pvc
spec:
  resource:
    requests:
      storage: 1Gi
    accessModes:
    - ReadWriteOnce
    storageClassName: ""
    
    
...
volumes:
- name: mongodb-data
  persistentVolumeClaim:
    claimName: monfodb-pvc
```

删除持久卷：remian模式下需要手动释放

回收策略配置： recycle 和 delete将会自动释放持久卷

## storageclass

使用pv和pvc可以屏蔽底层的基础设施，但是仍但需要一个集群管理员来创建pv，为了解决这个问题，可以通过动态配置持久卷来自动自行次任务。

创建storageclass需要一个中间程序来连接我们的存储，无论是网络存储，云存储还是其他的存储。

```shell
apiVersions: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
procisioner: kubenet. #这里是插件
parameters:
  type: pd
   # 传给插件的参数
  
  
 。。。
 pvc声明
 spec:
 storageClass:fast
...
```

