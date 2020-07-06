+++
title = "容器组"
subtitle = "在实例中学习 Kubernetes 容器组"
date = "2019-03-23"
url = "/pods/"
+++

容器组（Pod）是使用相同的网络、能挂载相同存储卷的一组容器，它是 Kubernetes 中的最基本部署单元。在容器组中的所有容器都会被调度到同一节点上。

执行下面的命令可以使用容器[镜像](https://quay.io/repository/openshiftlabs/simpleservice/) `quay.io/openshiftlabs/simpleservice:0.5.0` 启动一个容器组，并公开一个 `9876` 端口作为 HTTP 

```bash
$ kubectl run sise --image=quay.io/openshiftlabs/simpleservice:0.5.0 --port=9876
```

现在，可以确保容器组已处于运行状态：

```bash
$ kubectl get pods
NAME                      READY     STATUS    RESTARTS   AGE
sise-3210265840-k705b     1/1       Running   0          1m

$ kubectl describe pod sise-3210265840-k705b | grep IP:
IP:                     172.17.0.3
```

在执行上面的 `kubectl describe` 命令后，我们发现，在集群内部（比如，通过调用 [`oc rsh`](https://docs.openshift.com/container-platform/latest/cli_reference/openshift_cli/developer-cli-commands.html#rsh)），这一容器组可使用容器组 IP 地址 `172.17.0.3` 来访问：

```bash
[cluster] $ curl 172.17.0.3:9876/info
{"host": "172.17.0.3:9876", "version": "0.5.0", "from": "172.17.0.1"}
```
请注意，`kubecctl run` 会创建一个[部署](/deployments/)。所以，如果想删除容器组，就需要执行 `kubectl delete deployment sise`。


#### 使用配置文件

我们还可以用配置文件创建容器组。

下面的例子里，[容器组](https://github.com/openshift-evangelists/kbe/blob/master/specs/pods/pod.yaml)中运行了一个从前面过程中用过的 `simpleservice` 镜像，以及一个常见的 `CentOS` 镜像：

```bash
$ kubectl apply -f https://raw.githubusercontent.com/openshift-evangelists/kbe/master/specs/pods/pod.yaml

$ kubectl get pods
NAME                      READY     STATUS    RESTARTS   AGE
twocontainers             2/2       Running   0          7s
```

现在，我们可以连接进入 `CentOS` 容器，并以 localhost 的方式访问 `simpleservice`：

```bash
$ kubectl exec twocontainers -c shell -i -t -- bash
[root@twocontainers /]# curl -s localhost:9876/info
{"host": "localhost:9876", "version": "0.5.0", "from": "127.0.0.1"}
```

为[容器组](https://github.com/openshift-evangelists/kbe/blob/master/specs/pods/constraint-pod.yaml)指定 `resources` 字段来影响其中的容器可用的 CPU 及/或 内存资源：

```bash
$ kubectl create -f https://raw.githubusercontent.com/openshift-evangelists/kbe/master/specs/pods/constraint-pod.yaml

$ kubectl describe pod constraintpod
...
Containers:
  sise:
    ...
    Limits:
      cpu:      500m
      memory:   64Mi
    Requests:
      cpu:      500m
      memory:   64Mi
...
```

可以在[此处](https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-ram-container/)和[此处](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/)的文档继续学习 Kubernetes 中的资源限制。

如果要删除上面创建的容器组，只需执行：

```bash
$ kubectl delete pod twocontainers

$ kubectl delete pod constraintpod
```

总之，在 Kubernetes 中（一同）启动一个或多个容器非常简单，但是从上面的演示可以看出直接启动容器组的做法有一个明显的缺陷：在出现失败时，我们需要手工确保它的运行状态。一个更好的方法，是使用[部署](/deployments)来管理容器组，这样能让我们对它的生命周期，包括部署版本等操作拥有更好的控制能力。

[下一页](/labels)
