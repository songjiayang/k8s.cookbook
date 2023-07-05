# k8s.cookbook

个人 k8s 本地实验和使用中遇到一些常见问题以及解决办法。

## 索引
 - Docker
    - [修改根目录](https://github.com/songjiayang/k8s.cookbook#修改根目录)
 - Kind
   - [Node节点拉取镜像忽略tls认证](https://github.com/songjiayang/k8s.cookbook#Node节点拉取镜像忽略tls认证)
 - K8s
   - [如何远程登录含有多个容器的Pod](https://github.com/songjiayang/k8s.cookbook#如何远程登录含有多个容器的Pod)
 - Helm
   - [忽略证书验证](https://github.com/songjiayang/k8s.cookbook#忽略证书验证)
   - [指定 namespace](https://github.com/songjiayang/k8s.cookbook#指定-namespace)
   - [更改配置模版 values](https://github.com/songjiayang/k8s.cookbook#更改配置模版-values)

## Docker

### 修改根目录

打开 Docker 默认配置文件 /etc/docker/daemon.json， 添加 `data-root` 配置，例如：

```
{
     "data-root": "/docker",
}
```

或者在 dockerd 启动命令中添加 data-root 参数 

```
/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --data-root=/docker
```

## Kind

### Node节点拉取镜像忽略tls认证

使用 Kind 创建本地集群的时候，使用 containerdConfigPatches 配置项目添加 containerd 对应配置：

```
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
containerdConfigPatches:
- |-
  [plugins."io.containerd.grpc.v1.cri".registry.configs."registry.k8s.io".tls]
    insecure_skip_verify = true
```

最终会在 Node 节点的 containerd 配置文件 /etc/containerd/config.toml 添加如下内容：

```
[plugins]
    [plugins."io.containerd.grpc.v1.cri".registry]
      [plugins."io.containerd.grpc.v1.cri".registry.configs]
        [plugins."io.containerd.grpc.v1.cri".registry.configs."registry.k8s.io"]
          [plugins."io.containerd.grpc.v1.cri".registry.configs."registry.k8s.io".tls]
            insecure_skip_verify = true
```

## K8s

### 如何远程登录含有多个容器的Pod

这里主要 -c 参数指定 pod 中的容器名，例如

```
kubectl exec -it kube-prometheus-stack-operator-75b7d5b98d-nxkft -c prometheus sh
```

## Helm

### 忽略证书验证

主要使用 `--insecure-skip-tls-verify` 参数，例如：

```
helm repo add kubernetes-dashboard --insecure-skip-tls-verify  https://kubernetes.github.io/dashboard/
helm install kubernetes-dashboard --insecure-skip-tls-verify  kubernetes-dashboard/kubernetes-dashboard
```

### 指定 namespace

```
helm install kubernetes-dashboard -n monitoring --create-namespace
```

### 更改配置模版 values

```
helm show values prometheus-community/kube-prometheus-stack > values.yaml
helm upgrade --install prometheus prometheus-community/kube-prometheus-stack -f values.yaml 
```
