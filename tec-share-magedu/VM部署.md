## **Prometheus**概述

### **Prometheus工作流程**

Prometheus的工作流程核心是，以主动拉取pull的方式搜集被监控对象的metrics数据（监控指标数据），并将这些metrics数据存储到一个内存TSDB（时间序列数据库）中，并定期将内存中的指标同步到本地硬盘。

**基本架构如下图所示**

![https://prod-files-secure.s3.us-west-2.amazonaws.com/5740f48f-ac28-4d05-a829-3374825bf1fc/9db465a1-00e5-4d3d-8649-1eef4ae9d560/AssitanteForMEdu33c1d08a-338d-4f01-a445-e9a876143e15技术分享3c41fc97-a7c6-4e18-b814-f1ffa11bc280victoriametrics部署84ceedae-2838-46b8-b3c4-8c490232d5c7image-46.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/5740f48f-ac28-4d05-a829-3374825bf1fc/9db465a1-00e5-4d3d-8649-1eef4ae9d560/AssitanteForMEdu33c1d08a-338d-4f01-a445-e9a876143e15%E6%8A%80%E6%9C%AF%E5%88%86%E4%BA%AB3c41fc97-a7c6-4e18-b814-f1ffa11bc280victoriametrics%E9%83%A8%E7%BD%B284ceedae-2838-46b8-b3c4-8c490232d5c7image-46.png)

图可能有点复杂，简单总结如下：

- 从配置文件加载采集配置
- 周期性得往抓取对象发起抓取请求，得到数据
- 将数据写入本地盘或者写往远端存储
- alertmanager从监控数据中发现异常数据做出报警

如果我们现在有多个集群，并希望他们的监控数据存储到一起，可以进行聚合查询，用上述部署方案显然是不够的，因为上述方案中的Prometheus只能识别出本集群内的被监控目标。另外就是网络限制，多个集群之间的网络有可能是不通的，这就使得即使在某个集群中知道另一个集群的地址，也没法去抓取数据，要解决这些问题那就得用高可用方案。

### **Prometheus官方给出的三种高可用方案**

HA：即两套 Prometheus 采集完全一样的数据，外边挂负载均衡

![https://prod-files-secure.s3.us-west-2.amazonaws.com/5740f48f-ac28-4d05-a829-3374825bf1fc/ca6d53ec-46fe-4fdd-9068-804af836811f/AssitanteForMEdu33c1d08a-338d-4f01-a445-e9a876143e15技术分享3c41fc97-a7c6-4e18-b814-f1ffa11bc280victoriametrics部署84ceedae-2838-46b8-b3c4-8c490232d5c7image-47.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/5740f48f-ac28-4d05-a829-3374825bf1fc/ca6d53ec-46fe-4fdd-9068-804af836811f/AssitanteForMEdu33c1d08a-338d-4f01-a445-e9a876143e15%E6%8A%80%E6%9C%AF%E5%88%86%E4%BA%AB3c41fc97-a7c6-4e18-b814-f1ffa11bc280victoriametrics%E9%83%A8%E7%BD%B284ceedae-2838-46b8-b3c4-8c490232d5c7image-47.png)

HA + 远程存储：Prometheus通过 Remote write 写入到远程存储

![https://prod-files-secure.s3.us-west-2.amazonaws.com/5740f48f-ac28-4d05-a829-3374825bf1fc/ad185c0f-0ce5-4610-962a-2246cf22c523/AssitanteForMEdu33c1d08a-338d-4f01-a445-e9a876143e15技术分享3c41fc97-a7c6-4e18-b814-f1ffa11bc280victoriametrics部署84ceedae-2838-46b8-b3c4-8c490232d5c7image-48.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/5740f48f-ac28-4d05-a829-3374825bf1fc/ad185c0f-0ce5-4610-962a-2246cf22c523/AssitanteForMEdu33c1d08a-338d-4f01-a445-e9a876143e15%E6%8A%80%E6%9C%AF%E5%88%86%E4%BA%AB3c41fc97-a7c6-4e18-b814-f1ffa11bc280victoriametrics%E9%83%A8%E7%BD%B284ceedae-2838-46b8-b3c4-8c490232d5c7image-48.png)

联邦集群：即 Federation，按照功能进行分区，不同的 Shard 采集不同的数据，由 Global 节点来统一存放，解决监控数据规模的问题。

![https://prod-files-secure.s3.us-west-2.amazonaws.com/5740f48f-ac28-4d05-a829-3374825bf1fc/0b01869d-425f-4d10-b94e-c165499a7465/AssitanteForMEdu33c1d08a-338d-4f01-a445-e9a876143e15技术分享3c41fc97-a7c6-4e18-b814-f1ffa11bc280victoriametrics部署84ceedae-2838-46b8-b3c4-8c490232d5c7image-49.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/5740f48f-ac28-4d05-a829-3374825bf1fc/0b01869d-425f-4d10-b94e-c165499a7465/AssitanteForMEdu33c1d08a-338d-4f01-a445-e9a876143e15%E6%8A%80%E6%9C%AF%E5%88%86%E4%BA%AB3c41fc97-a7c6-4e18-b814-f1ffa11bc280victoriametrics%E9%83%A8%E7%BD%B284ceedae-2838-46b8-b3c4-8c490232d5c7image-49.png)

使用官方建议的多副本 + 联邦仍然会遇到一些问题，本质原因是 Prometheus 的本地存储没有数据同步能力，要在保证可用性的前提下再保持数据一致性是比较困难的，基本的多副本Proxy 满足不了要求，比如：

- Prometheus 集群的后端有 A 和 B 两个实例，A 和 B之间没有数据同步。A宕机一段时间，丢失了一部分数据，如果负载均衡正常轮询，请求打到A 上时，数据就会异常。
- 如果A和B的启动时间不同，时钟不同，那么采集同样的数据时间戳也不同，就多副本的数据不相同
- 就算用了远程存储，A 和 B 不能推送到同一个TSDB，如果每人推送自己的 TSDB，数据查询走哪边就是问题
- 官方建议数据做 Shard，然后通过 Federation来实现高可用，但是边缘节点和 Global节点依然是单点，需要自行决定是否每一层都要使用双节点重复采集进行保活。也就是仍然会有单机瓶颈

### **实际需求**

- 长期存储：1个月左右的数据存储，每天可能新增几十G，希望存储的维护成本足够小，有容灾和迁移。最好是存放在云上的TSDB 或者对象存储、文件存储上。
- 无限拓展：我们有上百集群，几千节点，上万个服务，单机 Prometheus无法满足，且为了隔离性，最好按功能做 Shard，如 Node机器监控与 K8S POD资源等业务监控分开、主机监控与日志监控也分开，者按业务类型分开。
- 全局视图：按类型分开之后，虽然数据分散了，但监控视图需要整合在一起，一个Grafana 里n个面板就可以看到所有地域+集群的监控数据，操作更方便，不用多个Grafana的Dashboard 切来切去。
- 无侵入性：不会因为增加监控业务而重新加载Prometheus配置，因为重新加载Prometheus配置对Prometheus稳定是有风险的。

## VictoriaMetrics概述

### victoriametrics是什么

[https://github.com/VictoriaMetrics/VictoriaMetrics](https://github.com/VictoriaMetrics/VictoriaMetrics)

VictoriaMetrics is a fast, cost-effective and scalable monitoring solution and time series [database.It](http://database.It) can be used as a long-term remote storage for Prometheus.

是一个支持高可用、经济高效且可扩展的监控解决方案和时间序列数据库，可用于 Prometheus 监控数据做长期远程存储。

Thanos 方案也可以用来解决 Prometheus的高可用和远程存储的问题，那么为什么我们还要使用 VictoriaMetrics呢？相对于 Thanos，VictoriaMetrics主要是一个可水平扩容的本地全量持久化存储方案(性能比thanos性能要好，thanos不是本地全量的，它很多历史数据是存放在对象存储当中的，如果要查询历史数据都要从对象存储当中去拉取，这肯定比本地获取数据要慢，VM要比Thanos性能要好，这是必然的)，VictoriaMetrics不仅仅是时序数据库，它的优势主要体现在一下几点：

1. 对外支持 Prometheus 相关的 API，可以直接用于 Grafana 作为 Prometheus 数据源使用（可以直接使用VictoriaMetrics 对接grafana）
2. 指标数据摄取和查询具备高性能和良好的可扩展性，性能比 InfluxDB 和 TimescaleDB 高出 20 倍 在处理高基数时间序列时，内存方面也做了优化，比 InfluxDB 少 10x 倍，比 Prometheus、Thanos 或 Cortex 少 7 倍
3. 高性能的数据压缩方式，与 TimescaleDB 相比，可以将多达 70 倍的数据点存入有限的存储空间，与 Prometheus、Thanos 或 Cortex 相比，所需的存储空间减少 7 倍
4. 它针对具有高延迟 IO 和低 IOPS 的存储进行了优化
5. 提供全局的查询视图，多个 Prometheus 实例或任何其他数据源可能会将数据摄取到 VictoriaMetrics
6. 操作简单
7. VictoriaMetrics 由一个没有外部依赖的小型可执行文件组成
8. 所有的配置都是通过明确的命令行标志和合理的默认值完成的 所有数据都存储在 - storageDataPath 命令行参数指向的目录中 可以使用 vmbackup/vmrestore 工具轻松快速地从实时快照备份到 S3 或 GCS 对象存储中
9. 支持从第三方时序数据库获取数据源
10. 由于存储架构，它可以保护存储在非正常关机（即 OOM、硬件重置或 kill -9）时免受数据损坏
11. 同样支持指标的 relabel 操作

# **架构**

VM 分为单节点和集群两个方案，根据业务需求选择即可。单节点版直接运行一个二进制文件既，官方建议采集数据点(data points)低于 100w/s，推荐 VM 单节点版，简单好维护，但不支持告警。集群版支持数据水平拆分。下图是 `VictoriaMetrics` 集群版官方的架构图。

![https://picdn.youdianzhishi.com/images/1650529246872.jpg](https://picdn.youdianzhishi.com/images/1650529246872.jpg)

主要包含以下几个组件：

- `vmstorage`：数据存储以及查询结果返回，默认端口为 8482
- `vminsert`：数据录入，可实现类似分片、副本功能，默认端口 8480
- `vmselect`：数据查询，汇总和数据去重，默认端口 8481
- `vmagent`：数据指标抓取，支持多种后端存储，会占用本地磁盘缓存，默认端口 8429
- `vmalert`：报警相关组件，不如果不需要告警功能可以不使用该组件，默认端口为 8880

集群方案把功能拆分为 `vmstorage`、`vminsert`、`vmselect` 组件，如果要替换 Prometheus，还需要使用 `vmagent`、`vmalert`。从上图也可以看出 `vminsert` 以及 `vmselect` 都是无状态的，所以扩展很简单，只有 `vmstorage` 是有状态的。

`vmagent` 的主要目的是用来收集指标数据然后存储到 VM 以及 Prometheus 兼容的存储系统中（支持 remote_write 协议即可）。

下图是 vmagent 的一个简单架构图，可以看出该组件也实现了 metrics 的 push 功能，此外还有很多其他特性：

- 替换 prometheus 的 scraping target
- 支持基于 prometheus relabeling 的模式添加、移除、修改 labels，可以方便在数据发送到远端存储之前进行数据的过滤
- 支持多种数据协议，influx line 协议，graphite 文本协议，opentsdb 协议，prometheus remote write 协议，json lines 协议，csv 数据
- 支持收集数据的同时，并复制到多种远端存储系统
- 支持不可靠远端存储（通过本地存储 `remoteWrite.tmpDataPath` )，同时支持最大磁盘占用
- 相比 prometheus 使用较少的内存、cpu、磁盘 io 以及网络带宽

![https://picdn.youdianzhishi.com/images/1650529595671.jpg](https://picdn.youdianzhishi.com/images/1650529595671.jpg)

## VictoriaMetrics二进制集群版部署

### 实验环境

### 机器角色

|机器|服务|
|---|---|
|192.168.158.130|prometheus+grafana|
|192.168.158.131|vm2|
|192.168.158.132|vm1|

### 整体架构图

[https://excalidraw.com/#json=2yp3te4lUrVpUxTsrCC0d,O7HlneGblzdnzkY8h6yUXw](https://excalidraw.com/#json=2yp3te4lUrVpUxTsrCC0d,O7HlneGblzdnzkY8h6yUXw)

### 实验步骤

### 机器初始化

1. 修改hostname

`hostnamectl set-hostname pg.magedu.com hostnamectl set-hostname vm1.magedu.com hostnamectl set-hostname vm2.magedu.com`

1. 关闭防火墙

`systemctl stop firewalld systemctl disable firewalld`

1. 关闭selinux

`setenforce 0`

1. 时间同步

`rpm -ivh <http://mirrors.wlnmp.com/centos/wlnmp-release-centos.noarch.rpm> yum -y install wntp ntpdate ntp1.aliyun.com`

### 下载软件包

`wget <https://github.com/VictoriaMetrics/VictoriaMetrics/releases/download/v1.90.0/victoria-metrics-linux-amd64-v1.90.0-cluster.tar.gz`>

`wget <https://github.com/prometheus/prometheus/releases/download/v2.39.1/prometheus-2.39.1.linux-amd64.tar.gz`>

`wget <https://github.com/prometheus/node_exporter/releases/download/v1.4.0/node_exporter-1.4.0.linux-amd64.tar.gz`>

`wget <https://dl.grafana.com/enterprise/release/grafana-enterprise-8.4.1-1.x86_64.rpm`>

### 安装grafana

1. 安装rpm包

`cd 你上传包位置 rpm -i --nodeps grafana-enterprise-8.4.1-1.x86_64.rpm`

1. 修改配置文件

```jsx
vim /etc/grafana/grafana.ini
```

```jsx
Server
[server](about:blank#server) # Protocol (http, https, h2, socket)
protocol = http
The ip address to bind to, empty will bind to all interfaces
http_addr = 0.0.0.0
The http port to use

http_port = 3000
```

```

1. 测试访问

访问 <http://你的ip:3000> 测试访问是否正常
```

1. 配置vmstore服务

```restorecon
/data/vmstorage-data

vi /etc/systemd/system/vmstorage.service

[Unit] Description=Vmstorage Server After=network.target

[Service] Restart=on-failure WorkingDirectory=/tmp
ExecStart=/usr/local/bin/vmstorage-prod

-loggerTimezone Asia/Shanghai

-storageDataPath /data/vmstorage-data

-httpListenAddr :8482

-vminsertAddr :8400

-vmselectAddr :8401

[Install] WantedBy=multi-user.target

systemctl enable –now vmstorage.service systemctl status
vmstorage.service

netstat -lntp | grep vmstorage tcp 0 0 0.0.0.0:8400 0.0.0.0:* LISTEN
2391/vmstorage-prod tcp 0 0 0.0.0.0:8401 0.0.0.0:* LISTEN
2391/vmstorage-prod tcp 0 0 0.0.0.0:8482 0.0.0.0:* LISTEN
2391/vmstorage-prod

curl 192.168.158.132:8482/metrics

```

1. 配置insert服务

```Plain
vi /etc/systemd/system/vminsert.service

[Unit]
Description=Vminsert Server
After=network.target

[Service]
Restart=on-failure
WorkingDirectory=/tmp
ExecStart=/usr/local/bin/vminsert-prod \\
-httpListenAddr :8480 \\
-storageNode=192.168.158.131:8401,192.168.158.132:8401

[Install]
WantedBy=multi-user.target

systemctl enable --now vminsert.service
systemctl status vminsert.service

netstat -lntp | grep vminsert
tcp        0      0 0.0.0.0:8480            0.0.0.0:*               LISTEN      2551/vminsert-prod

curl 192.168.158.132:8480/metrics
```

1. 配置select服务

vi /etc/systemd/system/vmselect.service

[Unit] Description=Vminsert Server After=network.target

[Service] Restart=on-failure WorkingDirectory=/tmp ExecStart=/usr/local/bin/vmselect-prod -httpListenAddr :8481 -storageNode=192.168.158.131:8400,192.168.158.132:8400

[Install] WantedBy=multi-user.target

systemctl enable –now vmselect.service systemctl status vmselect.service

netstat -lntp | grep vmselect tcp 0 0 0.0.0.0:8481 0.0.0.0:* LISTEN 2706/vmselect-prod

curl 192.168.158.132:8481/metrics

````

#### 安装prometheus

1. 安装

```Plain Text
tar zxvf prometheus-2.39.1.linux-amd64.tar.gz

mkdir /apps
mv prometheus-2.39.1.linux-amd64 prometheus
mv prometheus /apps

cd /apps/prometheus/
 ./promtool check config prometheus.yml

vi /etc/systemd/system/prometheus.service
[Unit]
Description=The Prometheus monitoring system and time series database.
Documentation=https://prometheus.io
After=network.target

[Service]
WorkingDirectory=/apps/prometheus/
ExecStart=/apps/prometheus/prometheus --config.file=/apps/prometheus/prometheus.yml --web.enable-lifecycle
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure

[Install]
WantedBy=multi-user.target

systemctl daemon-reload
systemctl enable --now prometheus.service
systemctl status prometheus.service
````

1. 修改配置文件

```Plain

# my global config

global: scrape_interval: 15s # Set the scrape interval to every 15
seconds. Default is every 1 minute. evaluation_interval: 15s # Evaluate
rules every 15 seconds. The default is every 1 minute. # scrape_timeout
is set to the global default (10s). # 集群版 remote_write: - url:
<http://192.168.158.132:8480/insert/0/prometheus> - url:
<http://192.168.158.131:8480/insert/0/prometheus>

重启 systemctl restart prometheus

```

#### 安装node exporter

```Plain
tar zxvf node_exporter-1.4.0.linux-amd64.tar.gz
mv node_exporter-1.4.0.linux-amd64 node_exporter
mv node_exporter /apps

cd

nohup
```

### 配置抓取任务

```Plain

scrape_configs: # The job name is added as a label
`job=<job_name>` to any timeseries scraped from this
config. - job_name: “prometheus”

```

# metrics_path defaults to '/metrics'

# scheme defaults to 'http'.

static_configs:

- targets: ["192.168.158.130:9100","192.168.158.131:9100"]

### 其他

1. vmui

```Plain
<http://192.168.158.131:8481/select/0/vmui/>
```

1. 热重载配置

```Plain

irate(node_cpu_seconds_total{job=“node”}[5m]) ```
```