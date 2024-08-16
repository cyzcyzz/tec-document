---
title: "K8S资源限制"
date: 2020-07-23T11:20:41+08:00
description: ""
draft: false
tags: []
categories: [容器]
---

k8s集群中，可以对两种资源进行限制，一是内存，二是cpu，使用Linux cGroups进行限制

### cpu限制

在Linux系统中，CPU的限制相比较内存的限制来说，更复杂一些。但是本身都可以通过cgroups来控制的，下面是一个K8S中的资源限制示例：

```yaml
resources:
  requests:
    memory: 50Mi
    cpu: 50m
  limits:
    memory: 100Mi
    cpu: 100m
```

单位后缀`m`表示千分之一核，也就是说，在K8S中，一个cpu核心被量化为1000m。上面的示例requests需要50m，也就是百分之五的cpu，要申请一个cpu的全部核心，可以使用1000m，或者1来表示。

这里调用docker来创建cgroups的时候，会有一些差异

Linux系统和docker都是按照1024的时间片值来计算的，这里和K8S有24的差值。我们称之为shares

shares用来设置CPU的相对值，如：a的shares值为1024，b的为512，则a的相对值为1024/1024+512 约为66%。shares有两个特点：

- a不忙，则b可以超过他的限制，a如果忙，则不能超过66的使用值
- 当有一个新的c 1024shares加入时，这个值重新计算。1024/1024+1024+512=40%，所以说这个值随着cgroups数量而改变的。

上面资源限制有两个字段，一个是requests，一个是limits。requests值，对应的就是我们上文介绍的shares值来计算的。limits又是怎么计算的呢？

其实他和requests使用的是不同的子系统来控制的。为什么会有单独的子系统呢。通过上文的介绍，你可能已经发现了，当一个进程没有设置shares的时候，他可以自由地使用cpu资源，而且shares子系统没法精准的控制使用的cpu，只能相对来控制。谷歌团队发现了这个问题，并且增加了一个子系统来控制：cpu带宽控制组。他定义了周期和配额两个属性。周期为1/10秒，100000微秒。配额为周期长度内可以使用的cpu时间数。两个配合起来可以精准的控制cpu的使用时长。周期值和配额均可以配置。

下面是几个例子：

```shell
# 1.限制只能使用1个CPU（每250ms能使用250ms的CPU时间）
$ echo 250000 > cpu.cfs_quota_us /* quota = 250ms */
$ echo 250000 > cpu.cfs_period_us /* period = 250ms */

# 2.限制使用2个CPU（内核）（每500ms能使用1000ms的CPU时间，即使用两个内核）
$ echo 1000000 > cpu.cfs_quota_us /* quota = 1000ms */
$ echo 500000 > cpu.cfs_period_us /* period = 500ms */

# 3.限制使用1个CPU的20%（每50ms能使用10ms的CPU时间，即使用一个CPU核心的20%）
$ echo 10000 > cpu.cfs_quota_us /* quota = 10ms */
$ echo 50000 > cpu.cfs_period_us /* period = 50ms */
```

上面例子中的100m,换算成这里就是10000/100000，默认的周期是100000，分别对应cfs_period_us=100000, cfs_quota_us=10000。

### 内存限制

内存限制相对于cpu就没有那么的复杂了。他直接把你设置的值映射到cgroups的内存控制子系统上。内存限制如上文的示例，requests代表你要申请的最少内存，也就是说，节点必须有这些内存可以申请，我可以超过这个内存，也可以少于这些内存，但节点一定要有。limits代表，我使用的内存上限，一旦超过，容器就要被oom了，所以建议两个设置合理的值。

### 默认限制

LimitRange 是用来设置 namespace 中 Pod 的默认的资源 request 和 limit 值，以及大小范围

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
  namespace: example
spec:
  limits:
  - default:  # default limit
      memory: 512Mi
      cpu: 2
    defaultRequest:  # default request
      memory: 256Mi
      cpu: 0.5
    max:  # max limit
      memory: 800Mi
      cpu: 3
    min:  # min request
      memory: 100Mi
      cpu: 0.3
    maxLimitRequestRatio:  # max value for limit / request
      memory: 2
      cpu: 2
    type: Container # limit type, support: Container / Pod / PersistentVolumeClaim
```

limitRange支持的参数如下：

- default 代表默认的limit
- defaultRequest 代表默认的request
- max 代表limit的最大值
- min 代表request的最小值
- maxLimitRequestRatio 代表 limit / request的最大值。由于节点是根据pod request 调度资源，可以做到节点超卖，maxLimitRequestRatio 代表pod最大超卖比例。