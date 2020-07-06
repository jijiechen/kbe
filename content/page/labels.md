+++
title = "Labels"
subtitle = "Kubernetes labels by example"
date = "2019-02-27"
url = "/labels/"
+++

标记是一种可用于组织 Kubernetes 对象的机制。标记是一种键值对，在长度和允许的值方面需要遵循一些[限制](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#syntax-and-character-set)，不过标记的含义并没有预先的定义。因而，我们可以根据需要使用标记


concerning length and allowed values but without any pre-defined meaning.
So you're free to choose labels as you see fit, for example, to express
environments such as 'this pod is running in production' or ownership,
like 'department X owns that pod'.

Let's create a [pod](https://github.com/openshift-evangelists/kbe/blob/master/specs/labels/pod.yaml) that initially has one label (`env=development`):


```bash
$ kubectl apply -f https://raw.githubusercontent.com/openshift-evangelists/kbe/master/specs/labels/pod.yaml

$ kubectl get pods --show-labels
NAME       READY     STATUS    RESTARTS   AGE    LABELS
labelex    1/1       Running   0          10m    env=development
```
In above `get pods` command note the `--show-labels` option that output the
labels of an object in an additional column.

You can add a label to the pod as:

```bash
$ kubectl label pods labelex owner=michael

$ kubectl get pods --show-labels
NAME        READY     STATUS    RESTARTS   AGE    LABELS
labelex     1/1       Running   0          16m    env=development,owner=michael
```

To use a label for filtering, for example to list only pods that have an
`owner` that equals `michael`, use the `--selector` option:

```bash
$ kubectl get pods --selector owner=michael
NAME      READY     STATUS    RESTARTS   AGE
labelex   1/1       Running   0          27m
```

The `--selector` option can be abbreviated to `-l`, so to select pods that are
labelled with `env=development`, do:

```bash
$ kubectl get pods -l env=development
NAME      READY     STATUS    RESTARTS   AGE
labelex   1/1       Running   0          27m
```

Oftentimes, Kubernetes objects also support set-based selectors.
Let's launch [another pod](https://github.com/openshift-evangelists/kbe/blob/master/specs/labels/anotherpod.yaml)
that has two labels (`env=production` and `owner=michael`):

```bash
$ kubectl apply -f https://raw.githubusercontent.com/openshift-evangelists/kbe/master/specs/labels/anotherpod.yaml
```

Now, let's list all pods that are either labelled with `env=development` or with
`env=production`:

```bash
$ kubectl get pods -l 'env in (production, development)'
NAME           READY     STATUS    RESTARTS   AGE
labelex        1/1       Running   0          43m
labelexother   1/1       Running   0          3m
```

Other verbs also support label selection, for example, you could
remove both of these pods with:

```bash
$ kubectl delete pods -l 'env in (production, development)'
```

Beware that this will destroy any pods with those labels.

You can also delete them directly, via their names, with:

```bash
$ kubectl delete pods labelex

$ kubectl delete pods labelexother
```

Note that labels are not restricted to pods. In fact you can apply them to
all sorts of objects, such as nodes or services.

[上一页](/pods) | [下一页](/deployments)
