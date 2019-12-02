# Kubernetes lab1 - Cluster multi nodes

## Use AWS Kubernetes Cluster for this lab

### Setup master
Add cloud provider option in kubelet
```
master$ sudo sed -i 's/KUBELET_EXTRA_ARGS.*/KUBELET_EXTRA_ARGS=--cloud-provider=aws/' /etc/default/kubelet
```

Start kubeadm init and force node name
```
master$ sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --node-name=`hostname -f`
```

Stop kubelet
```
master$ sudo systemctl stop kubelet
```

Add cloud provider option in /etc/kubernetes/manifests/kube-apiserver.yaml 
```
master$ sudo sed -i '/    - kube-apiserver/ a \    - --cloud-provider=aws' /etc/kubernetes/manifests/kube-apiserver.yaml
```

Add cloud provider and configure-cloud-routes option in /etc/kubernetes/manifests/kube-controller-manager.yaml 
```
master$ sudo sed -i '/    - kube-controller-manager/ a \    - --cloud-provider=aws\n    - --configure-cloud-routes=false' /etc/kubernetes/manifests/kube-controller-manager.yaml
```

Restart kubelet
```
master$ sudo systemctl start kubelet
```

### Setup kubeconfig
```
$ rm -rf ~ubuntu/.kube
$ mkdir -p ~ubuntu/.kube
$ sudo cp -i /etc/kubernetes/admin.conf ~ubuntu/.kube/config
$ sudo chown ubuntu:ubuntu ~ubuntu/.kube/config
```

### Install callico CNI (cf: https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/)
```
master$ kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/rbac-kdd.yaml
master$ kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml
```

### List nodes
```
master$ kubectl get nodes
```

### Create token for join command
```
master$ sudo kubeadm token create --print-join-command
```

### Setup node (on each node)
Add cloud provider option in kubelet
```
nodeX$ sudo sed -i 's/KUBELET_EXTRA_ARGS.*/KUBELET_EXTRA_ARGS=--cloud-provider=aws/' /etc/default/kubelet
```

Start kubeadm join command and force node name to hostname
```
nodeX$ sudo kubeadm join <...> --node-name=`hostname -f`
```

### List nodes
```
master$ kubectl get nodes
```

## Remove node1 from cluster
### Cordon node1
```
master$ kubectl cordon <node1>
```

### Drain node1
```
master$ kubectl drain <node1> --ignore-daemonsets
```

### Reset node1
```
node1$ sudo kubeadm reset
```

### Check nodes
```
master$ kubectl get nodes
```

### Create token (with join command)
```
master$ sudo kubeadm token create  --print-join-command
```

### Join node1 and force node name
```
node1$ sudo kubeadm join <...> --node-name=`hostname -f`
```

### Uncordon node1
```
master$ kubectl uncordon <node1>
```
