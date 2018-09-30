# kube
由于不能从google container上直接pull镜像，所以这里通过docker hub的Automated Builds功能从项目的dockerfile中Build到docker的官方服务器上，然后再从它们上面拉取.

##	kube 1.5.3需要的镜像:
```
REPOSITORY                                               TAG
gcr.io/google_containers/kube-apiserver-amd64            v1.5.3
gcr.io/google_containers/kube-controller-manager-amd64   v1.5.3
gcr.io/google_containers/kube-proxy-amd64                v1.5.3
gcr.io/google_containers/kube-scheduler-amd64            v1.5.3
gcr.io/google_containers/kubernetes-dashboard-amd64      v1.5.0
gcr.io/google_containers/etcd-amd64                      3.0.14-kubeadm
gcr.io/google_containers/kubedns-amd64                   1.9
gcr.io/google_containers/dnsmasq-metrics-amd64           1.0
gcr.io/google_containers/kube-dnsmasq-amd64              1.4
gcr.io/google_containers/kube-discovery-amd64            1.0
gcr.io/google_containers/exechealthz-amd64               1.2
gcr.io/google_containers/pause-amd64                     3.0
```

## docker hub上设置
由于docker hub不能后期更改一个image的tag，所以每次更新kubernetes时，都在build settings中，手动增加一个版本对应文件

## 更改tag
```
images=(kube-proxy-amd64:v1.5.3 kube-scheduler-amd64:v1.5.3 kube-controller-manager-amd64:v1.5.3 kube-apiserver-amd64:v1.5.3 etcd-amd64:3.0.14-kubeadm kube-discovery-amd64:1.0 pause-amd64:3.0 kubedns-amd64:1.9 dnsmasq-metrics-amd64:1.0 kube-dnsmasq-amd64:1.4 exechealthz-amd64:1.2)
for imageName in ${images[@]} ; do
  docker pull ist0ne/$imageName
  docker tag ist0ne/$imageName gcr.io/google_containers/$imageName
  docker rmi ist0ne/$imageName
done

images=(heapster:canary heapster_grafana:v3.1.1 heapster_influxdb:v0.7)
for imageName in ${images[@]} ; do
  docker pull ist0ne/$imageName
  docker tag ist0ne/$imageName kubernetes/$imageName
done
```

## 安装必要的软件包
```
yum install -y ebtables socat
rpm -ivh kubernetes-el7-x86_64/567600102f687e0f27bd1fd3d8211ec1cb12e71742221526bb4e14a412f4fdb5-kubernetes-cni-0.3.0.1-0.07a8a2.x86_64.rpm
rpm -ivh kubernetes-el7-x86_64/._8a299eb1db946b2bdf01c5d5c58ef959e7a9d9a0dd706e570028ebb14d48c42e-kubelet-1.5.1-0.x86_64.rpm
rpm -ivh kubernetes-el7-x86_64/8a299eb1db946b2bdf01c5d5c58ef959e7a9d9a0dd706e570028ebb14d48c42e-kubelet-1.5.1-0.x86_64.rpm
rpm -ivh kubernetes-el7-x86_64/._93af9d0fbd67365fa5bf3f85e3d36060138a62ab77e133e35f6cadc1fdc15299-kubectl-1.5.1-0.x86_64.rpm
rpm -ivh kubernetes-el7-x86_64/93af9d0fbd67365fa5bf3f85e3d36060138a62ab77e133e35f6cadc1fdc15299-kubectl-1.5.1-0.x86_64.rpm
```

## 通过kubeadm安装
```
kubeadm init --use-kubernetes-version v1.5.3
```
### 当通过kubeadm安装后，还需要安装网络
由于 pod 可能运行在不同的机器上，所以为了能让 pod 互相通信，就需要安装 pod 网络。这里使用的方案就是 weave net:
```
kubectl apply -f https://git.io/weave-kube
```
因为之前的 kube-dns addon 是依赖 pod 网络的，所以在没有部署 pod 网络之前，kube-dns 都会报错，因此只需要检查 kube-dns 是否成功就知道 pod 网络有没有成功了。
```
kubectl get pods --all-namespaces
```

## 如果docker hub也不能访问
如果docker hub也不能访问，那么可以通过daocloud的加速，它会在docker的配置--registry-mirro中加一个镜像服务器，但是通过它还是不能访问google container的镜像，所以还是需要上面在docker hub中配置
