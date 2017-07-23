---
title: kubernetes
tags: kubernetes
categories: 容器技术
---

## Kubernetes

### Kubernetes组件功能介绍：
* kube-apiserver：作为kubernetes系统的入口，封装了核心对象的增删改查操作，以RESTFul接口方式提供给外部客户和内部组件调用。它维护的REST对象将持久化到etcd（一个分布式强一致性的key/value存储）。
* kube-scheduler：负责集群的资源调度，为新建的pod分配机器。这部分工作分出来变成一个组件，意味着可以很方便地替换成其他的调度器。
* kube-controller-manager：负责执行各种控制器，目前有两类：

    * endpoint-controller：定期关联service和pod(关联信息由endpoint对象维护)，保证service到pod的映射总是最新的。
    * replication-controller：定期关联replicationController和pod，保证replicationController定义的复制数量与实际运行pod的数量总是一致的。
* kube-proxy：负责为pod提供代理。它会定期从etcd获取所有的service，并根据service信息创建代理。当某个客户pod要访问其他pod时，访问请求会经过本机proxy做转发。
* kubelet：负责管控docker容器，如启动/停止、监控运行状态等。它会定期从etcd获取分配到本机的pod，并根据pod信息启动或停止相应的容器。同时，它也会接收apiserver的HTTP请求，汇报pod的运行状态。


### Kubernetes安装过程中注意事项：
1.Kubernetes解压之后，进入kubernetes/server目录下，再次解压kubernetes-server-linux-amd64.tar.gz文件，再进入kubernetes/server/kubernetes/server/bin目录下，才能发现kubernetes对应的很多命令。
2.命令list：
Master：
./etcd > /var/log/etcd.log 2>&1 &
./kube-apiserver  --insecure-bind-address=9.21.62.27 --insecure-port=8080 --service-cluster-ip-range=172.17.0.0/16 --etcd_servers=http://127.0.0.1:4001 --logtostderr=true --v=0 --log_dir=/home/kubernetes/logs/kube

./kube-controller-manager --v=0 --logtostderr=false --log_dir=/var/log/kube  --master=9.21.62.27:8080
./kube-scheduler  --master='9.21.62.27:8080'   --v=0    --log_dir=/var/log/kube
Slave:
./kube-proxy  --logtostderr=false    --v=0     --master=http://9.21.62.27:8080
./kubelet  --logtostderr=false --v=0 --allow-privileged=false  --log_dir=/var/log/kube  --address=0.0.0.0  --port=10250 --hostname_override=172.30.13.0  --api_servers=http://9.21.62.27:8080
Test: ./kubectl -s http://9.21.62.27:8080 get services

创建kubernetes Dashboard：./kubectl -s http://9.21.62.27:8080 create -f ../../../../cluster/addons/dashboard/dashboard-service.yaml  --namespace=kube-system
访问地址为： http://9.21.62.27:8080/api/v1/proxy/namespaces/kube-system/services/kubernetes-dashboard/#/dashboard/

基于influxdb,grafana,heapster创建日志分析性能监控：  ./kubectl -s http://9.21.62.27:8080 create -f ../../../../../kubernetes/cluster/addons/cluster-monitoring/influxdb/

显示cluster-info信息：
./kubectl -s http://9.21.62.27:8080 --namespace="kube-system"   cluster-info
Kubernetes master is running at http://9.21.62.27:8080
Heapster is running at http://9.21.62.27:8080/api/v1/proxy/namespaces/kube-system/services/heapster
Grafana is running at http://9.21.62.27:8080/api/v1/proxy/namespaces/kube-system/services/monitoring-grafana
InfluxDB is running at http://9.21.62.27:8080/api/v1/proxy/namespaces/kube-system/services/monitoring-influxdb

## Kubernetes安装：

```bash
cmd/kube-dns
cmd/kube-proxy
cmd/kube-apiserver
cmd/kube-controller-manager
cmd/kubelet
cmd/kubeadm
cmd/hyperkube
cmd/kube-discovery
plugin/cmd/kube-scheduler
```

### 方式一：  通过Docker在本地来安装Kubernetes, Single node Kubernetes cluster using Docker.
```bash
export K8S_VERSION=$(curl -sS https://storage.googleapis.com/kubernetes-release/release/stable.txt)
export ARCH=amd64
docker run -d \
    --volume=/:/rootfs:ro \
    --volume=/sys:/sys:rw \
    --volume=/var/lib/docker/:/var/lib/docker:rw \
    --volume=/var/lib/kubelet/:/var/lib/kubelet:rw \
    --volume=/var/run:/var/run:rw \
    --net=host \
    --pid=host \
    --privileged \
    gcr.io/google_containers/hyperkube-${ARCH}:${K8S_VERSION} \
    /hyperkube kubelet \
        --containerized \
        --hostname-override=127.0.0.1 \
        --api-servers=http://localhost:8080 \
        --config=/etc/kubernetes/manifests \
        --allow-privileged --v=2
```
安装kubectl：
```bash
curl -sSL "http://storage.googleapis.com/kubernetes-release/release/v1.2.0/bin/linux/amd64/kubectl" > /usr/bin/kubectl
chmod +x /usr/bin/kubectl
```
通过kubectl来配置本地kubernetes cluster：
```bash
kubectl config set-cluster test-doc --server=http://localhost:8080
kubectl config set-context test-doc --cluster=test-doc
kubectl config use-context test-doc
```
Test本地Cluster：
```bash
kubectl get nodes
Run an application：
kubectl run nginx --image=nginx --port=80
```

### 方式二：通过Minikube在本地创建Kubernetes Cluster
1. 机器必须支持虚拟化：
```bash
cat /proc/cpuinfo | grep 'vmx\|svm'
```
2.需要安装VirtualBox环境：
ubuntu14.04:   wget http://download.virtualbox.org/virtualbox/5.0.22/virtualbox-5.0_5.0.22-108108~Ubuntu~trusty_amd64.deb
Redhat 7.1: wget http://download.virtualbox.org/virtualbox/5.0.22/VirtualBox-5.0-5.0.22_108108_el7-1.x86_64.rpm
3.本地利用virtualbox搭建Kubernetes Cluster：
```bash
minikube start
kubectl get pods --all-namespaces
minikube dashboard
```
4.Test 搭建的环境：
```bash
kubectl get nodes
```

### 方式三：Installation Kubernetes Locally with No VM：
Requirements:
Docker 1.8.3+ : https://docs.docker.com/engine/installation/#installation
etcd : https://github.com/coreos/etcd/releases
go : https://golang.org/doc/install

etcd安装与Test：
```bash
curl -L  https://github.com/coreos/etcd/releases/download/v2.3.7/etcd-v2.3.7-linux-amd64.tar.gz -o etcd-v2.3.7-linux-amd64.tar.gz
tar xzvf etcd-v2.3.7-linux-amd64.tar.gz
cd etcd-v2.3.7-linux-amd64
./etcd
```
Open another terminal:
```bash
./etcdctl set mykey "this is awesome"
./etcdctl get mykey

docker run --name etcd quay.io/coreos/etcd:v2.3.7
docker exec etcd /etcdctl set foo bar

rkt run --volume data-dir,kind=host,source=/tmp --mds-register=false coreos.com/etcd:v2.3.7
```
