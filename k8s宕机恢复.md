# K8s 宕机恢复

> 前言 当k8s所在的电脑宕机了



![img](https://dl4.weshineapp.com/gif/20220218/e5e406ef3c1837a0a04f8e7b59bc335d.gif?f=micro_)

此时你所有的指令都无效

```
[root@k8smaster ~]# kubectl get po
The connection to the server 192.168.206.130:6443 was refused - did you specify the right host or port?
```



> 原因

由于断电停机，kubernetes集群挂掉，使用任意kubectl 命令会报错：The connection to the server ip:6443 was refused - did you specify the right host or port，重启kubelet也不能恢复，etcd读取数据报错，数据文件损坏



```shell
[root@k8smaster ~]# systemctl status kubelet.service
● kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; vendor preset: disabled)
  Drop-In: /usr/lib/systemd/system/kubelet.service.d
           └─10-kubeadm.conf
   Active: active (running) since 六 2024-05-18 16:16:11 CST; 8s ago
     Docs: https://kubernetes.io/docs/
 Main PID: 2535 (kubelet)
    Tasks: 13
   Memory: 44.7M
   CGroup: /system.slice/kubelet.service
           └─2535 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --...

5月 18 16:16:19 k8smaster kubelet[2535]: E0518 16:16:19.216072    2535 kubelet.go:2461] "Error getting node" err="node \"k8smaster\" not found"
5月 18 16:16:19 k8smaster kubelet[2535]: E0518 16:16:19.317759    2535 kubelet.go:2461] "Error getting node" err="node \"k8smaster\" not found"
5月 18 16:16:19 k8smaster kubelet[2535]: E0518 16:16:19.418196    2535 kubelet.go:2461] "Error getting node" err="node \"k8smaster\" not found"
5月 18 16:16:19 k8smaster kubelet[2535]: E0518 16:16:19.518424    2535 kubelet.go:2461] "Error getting node" err="node \"k8smaster\" not found"
5月 18 16:16:19 k8smaster kubelet[2535]: E0518 16:16:19.620122    2535 kubelet.go:2461] "Error getting node" err="node \"k8smaster\" not found"

```



### **查看apiserver是否挂掉**

注意：如果你使用的是docker,执行这个docker ps -a| grep kube-apiserver



### **查看etcd是否挂掉**

注意：如果你使用的是docker,执行这个docker ps -a| grep etcd

ectd 在读取数据时发生了错误，导致启动失败。继而api-server也无法启动

etcd的数据文件损坏了，要做数据恢复，而我这是实验环境，没搞etcd备份就只能重置集群了

**注意，线上使用etcd一定要做高可用和定期备份，否则就悲催了**



## 重置k8s集群

需要在每台机器上执行

```bash
kubeadm reset
```

删除$HOME/.kube

```bash
rm -rf $HOME/.kube
```

初始化master

```
kubeadm init \
	--apiserver-advertise-address=192.168.206.130 \
	--image-repository registry.aliyuncs.com/google_containers \
	--kubernetes-version=v1.23.6 \
	--service-cidr=10.96.0.0/12 \
	--pod-network-cidr=10.244.0.0/16
```



创建必要文件

```bash
 mkdir -p $HOME/.kube
 sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
 sudo chown $(id -u):$(id -g) $HOME/.kube/config
```



> 其余node拜master 码头

```
[root@k8snode1 ~]# kubeadm join 192.168.206.130:6443 --token 4wdr7f.7qdbs17sozs24dln \
> --discovery-token-ca-cert-hash sha256:a9c341457a3c7c3572fdba72e557d03da13e21136a1970956e54ef1ec13caa24
```



欧克就恢复了