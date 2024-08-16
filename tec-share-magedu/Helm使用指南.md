## helm是什么

[Helm](https://links.jianshu.com/go?to=http%3A%2F%2Fhelm.sh%2F) 是一个 Kubernetes 应用的包管理工具，用来管理 [chart](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fhelm%2Fcharts)——预先配置好的安装包资源。说白了就是centos的yum，ubuntu的apt-get这样的包管理工具。helm可以更方便的部署k8s上的资源（pod、service等）。我们写好yaml，通过kubectl（或者调api）也能很好的管理我们需要的资源，那为什么还要有helm？这就和为什么要有yum一样了。**更方便**，**更高效**，**更容易部署复杂的应用**。helm2中需要tiller这个代理服务与apiserver通信，helm3中已经完全废弃，这样就更像yum了。下面类比yum介绍下helm的基本概念

## 基本概念

1. 先了解下helm中的基本概念 helm和yum一样也有源（repo）、包的概念：
    - chart：类似于yum的rpm包，里面定义了部署资源以及一些依赖的信息（deployment，service等）。和rpm包作用上类似，我们需要部署那些服务，按照格式定义好就行了。
    - repo：类似于yum源，helm也有自己的源，存放chart。
    - Release：chart部署到k8s后自己的唯一标识，
2. helm client helm client就是执行命令的客户端程序，负责创建chart，安装chart，维护release等。然后将用户执行的命令发送到apiserver。

## helm基本使用

---

## Helm项目安装

Helm项目提供了两种获取和安装Helm的方式。这是官方提供的获取Helm发布版本的方法。另外， Helm社区提供了通过不同包管理器安装Helm的方法。这些方法可以在下面的官方方法之后看到。

### 用二进制版本安装

每个Helm[版本](https://github.com/helm/helm/releases)都提供了各种操作系统的二进制版本，这些版本可以手动下载和安装。

1. 下载[需要的版本](https://github.com/helm/helm/releases)
2. 解压(`tar -zxvf helm-v3.0.0-linux-amd64.tar.gz`)
3. 在解压目录中找到`helm`程序，移动到需要的目录中(`mv linux-amd64/helm /usr/local/bin/helm`)

然后就可以执行客户端程序并[添加稳定仓库](https://helm.sh/zh/docs/intro/quickstart/#%E5%88%9D%E5%A7%8B%E5%8C%96): `helm help`.

## 通过包管理器安装

Helm社区提供了通过操作系统包管理器安装Helm的方式。但Helm项目不支持且不认为是可信的第三方。

### 使用Homebrew (macOS)

Helm社区成员贡献了一种在Homebrew构建Helm的方案，这个方案通常是最新的。

`Plain Text brew install helm`

(注意：还有一个emacs-helm的方案，当然这是另一个项目了。)

### 使用 dnf/yum (fedora)

从Fedora 35开始， 官方仓库可以使用helm了，可以调用以下命令安装helm：

`Plain Text sudo dnf install helm`

---

## ‘helm search’：查找 Charts

Helm 自带一个强大的搜索命令，可以用来从两种来源中进行搜索：

- `helm search hub` 从 [Artifact Hub](https://artifacthub.io/) 中查找并列出 helm charts。 Artifact Hub中存放了大量不同的仓库。
- `helm search repo` 从你添加（使用 `helm repo add`）到本地 helm 客户端中的仓库中进行查找。该命令基于本地数据进行搜索，无需连接互联网。

你可以通过运行 `helm search hub` 命令找到公开可用的charts：

`Plain Text $ helm search hub wordpress URL CHART VERSION APP VERSION DESCRIPTION <https://hub.helm.sh/charts/bitnami/wordpress> 7.6.7 5.2.4 Web publishing platform for building blogs and ... <https://hub.helm.sh/charts/presslabs/wordpress->... v0.6.3 v0.6.3 Presslabs WordPress Operator Helm Chart <https://hub.helm.sh/charts/presslabs/wordpress->... v0.7.1 v0.7.1 A Helm chart for deploying a WordPress site on ...`

上述命令从 Artifact Hub 中搜索所有的 `wordpress` charts。

如果不进行过滤，`helm search hub` 命令会展示所有可用的 charts。

使用 `helm search repo` 命令，你可以从你所添加的仓库中查找chart的名字。

`Plain Text $ helm repo add brigade <https://brigadecore.github.io/charts> "brigade" has been added to your repositories $ helm search repo brigade NAME CHART VERSION APP VERSION DESCRIPTION brigade/brigade 1.3.2 v1.2.1 Brigade provides event-driven scripting of Kube... brigade/brigade-github-app 0.4.1 v0.2.1 The Brigade GitHub App, an advanced gateway for... brigade/brigade-github-oauth 0.2.0 v0.20.0 The legacy OAuth GitHub Gateway for Brigade brigade/brigade-k8s-gateway 0.1.0 A Helm chart for Kubernetes brigade/brigade-project 1.0.0 v1.0.0 Create a Brigade project brigade/kashti 0.4.0 v0.4.0 A Helm chart for Kubernetes`

Helm 搜索使用模糊字符串匹配算法，所以你可以只输入名字的一部分：

`Plain Text $ helm search repo kash NAME CHART VERSION APP VERSION DESCRIPTION brigade/kashti 0.4.0 v0.4.0 A Helm chart for Kubernetes`

搜索是用来发现可用包的一个好办法。一旦你找到你想安装的 helm 包，你便可以通过使用 `helm install` 命令来安装它。

## ‘helm install’：安装一个 helm

包

使用 `helm install` 命令来安装一个新的 helm 包。最简单的使用方法只需要传入两个参数：你命名的release名字和你想安装的chart的名称。

```Plain
happy-panda LAST DEPLOYED: Tue Jan 26 10:27:17 2021 NAMESPACE: default
STATUS: deployed REVISION: 1 NOTES: ** Please be patient while the chart
is being deployed **

Your WordPress site can be accessed through the following DNS name
from within your cluster:

```

happy-panda-wordpress.default.svc.cluster.local (port 80)

```

To access your WordPress site from outside the cluster follow the
steps below:

1. Get the WordPress URL by running these commands:

NOTE: It may take a few minutes for the LoadBalancer IP to be
available. Watch the status with: ‘kubectl get svc –namespace default -w
happy-panda-wordpress’

export SERVICE_IP=$(kubectl get svc
--namespace default happy-panda-wordpress --template "{{ range (index
.status.loadBalancer.ingress 0) }}{{.}}{{ end }}")
   echo "WordPress URL: http://$SERVICE_IP/” echo “WordPress
Admin URL: http://$SERVICE_IP/admin”

1. Open a browser and access WordPress using the obtained
URL.
2. Login with the following credentials below to see your
blog:

echo Username: user echo Password: $(kubectl get secret –namespace
default happy-panda-wordpress -o jsonpath=“{.data.wordpress-password}” |
base64 –decode)

```

现在`wordpress` chart 已经安装。注意安装chart时创建了一个新的 _release_ 对象。上述发布被命名为 `happy-panda`。 （如果想让Helm生成一个名称，删除发布名称并使用`--generate-name`。）

在安装过程中，`helm` 客户端会打印一些有用的信息，其中包括：哪些资源已经被创建，release当前的状态，以及你是否还需要执行额外的配置步骤。

Helm按照以下顺序安装资源：

- Namespace
    
- NetworkPolicy
    
- ResourceQuota
    
- LimitRange
    
- PodSecurityPolicy
    
- PodDisruptionBudget
    
- ServiceAccount
    
- Secret
    
- SecretList
    
- ConfigMap
    
- StorageClass
    
- PersistentVolume
    
- PersistentVolumeClaim
    
- CustomResourceDefinition
    
- ClusterRole
    
- ClusterRoleList
    
- ClusterRoleBinding
    
- ClusterRoleBindingList
    
- Role
    
- RoleList
    
- RoleBinding
    
- RoleBindingList
    
- Service
    
- DaemonSet
    
- Pod
    
- ReplicationController
    
- ReplicaSet
    
- Deployment
    
- HorizontalPodAutoscaler
    
- StatefulSet
    
- Job
    
- CronJob
    
- Ingress
    
- APIService
    

Helm 客户端不会等到所有资源都运行才退出。许多 charts 需要大小超过 600M 的 Docker 镜像，可能需要很长时间才能安装到集群中。

你可以使用 `helm status` 来追踪 release 的状态，或是重新读取配置信息：

```Plain
$ helm status happy-panda
NAME: happy-panda
LAST DEPLOYED: Tue Jan 26 10:27:17 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
** Please be patient while the chart is being deployed **

Your WordPress site can be accessed through the following DNS name from within your cluster:

    happy-panda-wordpress.default.svc.cluster.local (port 80)

To access your WordPress site from outside the cluster follow the steps below:

1. Get the WordPress URL by running these commands:

  NOTE: It may take a few minutes for the LoadBalancer IP to be available.
        Watch the status with: 'kubectl get svc --namespace default -w happy-panda-wordpress'

   export SERVICE_IP=$(kubectl get svc --namespace default happy-panda-wordpress --template "{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}")
   echo "WordPress URL: http://$SERVICE_IP/"
   echo "WordPress Admin URL: http://$SERVICE_IP/admin"

2. Open a browser and access WordPress using the obtained URL.

3. Login with the following credentials below to see your blog:

  echo Username: user
  echo Password: $(kubectl get secret --namespace default happy-panda-wordpress -o jsonpath="{.data.wordpress-password}" | base64 --decode)
```

上述信息展示了 release 的当前状态。

### 安装前自定义 chart

上述安装方式只会使用 chart 的默认配置选项。很多时候，我们需要自定义 chart 来指定我们想要的配置。

使用 `helm show values` 可以查看 chart 中的可配置选项：

```Plain
image parameters ## Please, note that this will override the image
parameters, including dependencies, configured to use the global value
## Current available global Docker image parameters: imageRegistry and
imagePullSecrets ## # global: # imageRegistry: myRegistryName #
imagePullSecrets: # - myRegistryKeySecretName # storageClass:
myStorageClass

## Bitnami WordPress image
version

## ref:
<https://hub.docker.com/r/bitnami/wordpress/tags/>

## 

image: registry: docker.io repository: bitnami/wordpress tag:
5.6.0-debian-10-r35 [..]

```

然后，你可以使用 YAML 格式的文件覆盖上述任意配置项，并在安装过程中使用该文件。

```Plain
$ echo '{mariadb.auth.database: user0db, mariadb.auth.username: user0}' > values.yaml
$ helm install -f values.yaml bitnami/wordpress --generate-name
```

上述命令将为 MariaDB 创建一个名称为 `user0` 的默认用户，并且授予该用户访问新建的 `user0db` 数据库的权限。chart 中的其他默认配置保持不变。

安装过程中有两种方式传递配置数据：

- `-values` (或 `f`)：使用 YAML 文件覆盖配置。可以指定多次，优先使用最右边的文件。
- `-set`：通过命令行的方式对指定项进行覆盖。

如果同时使用两种方式，则 `--set` 中的值会被合并到 `--values` 中，但是 `--set` 中的值优先级更高。在`--set` 中覆盖的内容会被被保存在 ConfigMap 中。可以通过 `helm get values <release-name>` 来查看指定 release 中 `--set` 设置的值。也可以通过运行 `helm upgrade` 并指定 `--reset-values` 字段来清除 `--set` 中设置的值。

## ‘helm uninstall’：卸载 release

使用 `helm uninstall` 命令从集群中卸载一个 release：

`Plain Text $ helm uninstall happy-panda`

该命令将从集群中移除指定 release。你可以通过 `helm list` 命令看到当前部署的所有 release：

`Plain Text $ helm list NAME VERSION UPDATED STATUS CHART inky-cat 1 Wed Sep 28 12:59:46 2016 DEPLOYED alpine-0.1.0`

从上面的输出中，我们可以看到，`happy-panda` 这个 release 已经被卸载。

在上一个 Helm 版本中，当一个 release 被删除，会保留一条删除记录。而在 Helm 3 中，删除也会移除 release 的记录。 如果你想保留删除记录，使用 `helm uninstall --keep-history`。使用 `helm list --uninstalled` 只会展示使用了 `--keep-history` 删除的 release。

`helm list --all` 会展示 Helm 保留的所有 release 记录，包括失败或删除的条目（指定了 `--keep-history`）：

`Plain Text $ helm list --all NAME VERSION UPDATED STATUS CHART happy-panda 2 Wed Sep 28 12:47:54 2016 UNINSTALLED wordpress-10.4.5.6.0 inky-cat 1 Wed Sep 28 12:59:46 2016 DEPLOYED alpine-0.1.0 kindred-angelf 2 Tue Sep 27 16:16:10 2016 UNINSTALLED alpine-0.1.0`

注意，因为现在默认会删除 release，所以你不再能够回滚一个已经被卸载的资源了。

## ‘helm repo’：使用仓库

Helm 3 不再附带一个默认的 chart 仓库。`helm repo` 提供了一组命令用于添加、列出和移除仓库。

使用 `helm repo list` 来查看配置的仓库：

`Plain Text $ helm repo list NAME URL stable <https://charts.helm.sh/stable> mumoshu <https://mumoshu.github.io/charts`>

使用 `helm repo add` 来添加新的仓库：

`Plain Text $ helm repo add dev <https://example.com/dev-charts`>

因为 chart 仓库经常在变化，在任何时候你都可以通过执行 `helm repo update` 命令来确保你的 Helm 客户端是最新的。

使用 `helm repo remove` 命令来移除仓库。

## 创建你自己的 charts

[chart 开发指南](https://helm.sh/zh/docs/topics/charts) 介绍了如何开发你自己的chart。 但是你也可以通过使用 `helm create` 命令来快速开始：

`Plain Text $ helm create deis-workflow Creating deis-workflow`

现在，`./deis-workflow` 目录下已经有一个 chart 了。你可以编辑它并创建你自己的模版。

在编辑 chart 时，可以通过 `helm lint` 验证格式是否正确。

当准备将 chart 打包分发时，你可以运行 `helm package` 命令：

`Plain Text $ helm package deis-workflow deis-workflow-0.1.0.tgz`

然后这个 chart 就可以很轻松的通过 `helm install` 命令安装：

`Plain Text $ helm install deis-workflow ./deis-workflow-0.1.0.tgz ...`

打包好的 chart 可以上传到 chart 仓库中。查看[Helm chart 仓库](https://helm.sh/zh/docs/topics/chart_repository)获取更多信息。

## 制作helm chart

示例dp

`Plain Text apiVersion: apps/v1 kind: Deployment metadata: name: my-nginx spec: replicas: 3 selector: matchLabels: app: nginx template: metadata: labels: app: nginx spec: containers: - name: nginx image: nginx:latest ports: - containerPort: 80 --- apiVersion: v1 kind: Service metadata: name: ngx-service labels: app: nginx spec: type: NodePort selector: app: nginx ports: - port: 80 targetPort: 80 nodePort: 32500`

### 内置对象

[https://helm.sh/zh/docs/chart_template_guide/builtin_objects/](https://helm.sh/zh/docs/chart_template_guide/builtin_objects/)

`Plain Text apiVersion: apps/v1 kind: Deployment metadata: name: my-nginx annotations: magedu.com/helm-name: {{ .Release.Name }}`

### value对象

该对象提供了传递值到chart的方法，

其内容来自于多个位置：

- chart中的`values.yaml`文件
- 如果是子chart，就是父chart中的`values.yaml`文件
- 使用`f`参数(`helm install -f myvals.yaml ./mychart`)传递到 `helm install` 或 `helm upgrade`的values文件
- 使用`-set` (比如`helm install --set foo=bar ./mychart`)传递的单个参数

以上列表有明确顺序：默认使用`values.yaml`，可以被父chart的`values.yaml`覆盖，继而被用户提供values文件覆盖， 最后会被`--set`参数覆盖，优先级为`values.yaml`最低，`--set`参数最高。

values文件是普通的YAML文件。现在编辑`mychart/values.yaml`然后编辑配置映射ConfigMap模板。

`Plain Text apiVersion: apps/v1 kind: Deployment metadata: name: my-nginx annotations: magedu.com/helm-name: {{ .Release.Name }} spec: replicas: {{ .Value.req }} selector: matchLabels: app: nginx template: metadata: labels: app: nginx spec: containers: - name: nginx image: nginx:{{ .Values.version }} ports: - containerPort: 80`

### 模板函数

[https://helm.sh/zh/docs/chart_template_guide/functions_and_pipelines/](https://helm.sh/zh/docs/chart_template_guide/functions_and_pipelines/)

`Plain Text spec: replicas: {{ .Value.repicas | default 1 }} selector: matchLabels: app: nginx template: metadata: labels: app: nginx spec: containers: - name: nginx image: nginx:latest ports: - containerPort: 80`

### 流程控制

控制结构(在模板语言中称为”actions”)提供给你和模板作者控制模板迭代流的能力。 Helm的模板语言提供了以下控制结构：

- `if`/`else`， 用来创建条件语句
- `with`， 用来指定范围
- `range`， 提供”for each”类型的循环

除了这些之外，还提供了一些声明和使用命名模板的关键字：

- `define` 在模板中声明一个新的命名模板
- `template` 导入一个命名模板
- `block` 声明一种特殊的可填充的模板块

该部分，我们会讨论关于`if`，`with`，和 `range`。其他部门会在该指南的“命名模板”部分说明

`Plain Text apiVersion: apps/v1 kind: Deployment metadata: {{- if eq .Values.custom.name "" }} name: my-nginx {{- else }} name: {{ .Values.custom.name }} {{- end }} annotations: magedu.com/helm-name: {{ .Release.Name }}`

## 发布到自建仓库

使用nginx发布