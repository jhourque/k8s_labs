# Kubernetes lab1 - Kubeadm

## Vagrantfile with docker and kubernetes install
Vagrantfile:
```
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"
  config.vm.network "private_network", type: "dhcp"
  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    vb.name = "UBU18-K8S"
    vb.customize ["modifyvm", :id, "--memory", 2048]
    vb.customize ["modifyvm", :id, "--cpus", 2]
  end
  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt install -y docker.io
    systemctl enable docker.service
    usermod -aG docker vagrant
    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
    apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
    apt install -y kubeadm 
    swapoff -a
  SHELL
end
```

Create & Start VM:
```
>vagrant up
```

Login to VM with
```
>vagrant ssh
```

### Setup master
```
$ sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

### Setup kubeconfig
```
$ rm -rf ~vagrant/.kube
$ mkdir -p ~vagrant/.kube
$ sudo cp -i /etc/kubernetes/admin.conf ~vagrant/.kube/config
$ sudo chown vagrant:vagrant ~vagrant/.kube/config
```

### List nodes
```
$ kubectl get nodes
```

### Allow Pods run on master
```
$ kubectl taint node ubuntu-bionic node-role.kubernetes.io/master:NoSchedule-
```

### List pod running in all namespaces
```
$ kubectl get pod --all-namespaces
```

### Question?
- What is pods status?
- is it normal?

### Install flannel CNI (cf: https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/)
```
$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/a70459be0084506e4ec919aa1c114638878db11b/Documentation/kube-flannel.yml
```

### Check pod running in all namespaces
```
$ kubectl get pod --all-namespaces
```

### reset kubernetes config
```
$ sudo kubeadm reset
$ sudo iptables -F && sudo iptables -t nat -F && sudo iptables -t mangle -F && sudo iptables -X
```

### Reconfigure for callico
```
$ sudo kubeadm init --pod-network-cidr=192.168.0.0/16
$ rm -rf ~vagrant/.kube
$ mkdir -p ~vagrant/.kube
$ sudo cp -i /etc/kubernetes/admin.conf ~vagrant/.kube/config
$ sudo chown vagrant:vagrant ~vagrant/.kube/config
$ kubectl taint node ubuntu-bionic node-role.kubernetes.io/master:NoSchedule-
```

### Install callico CNI (cf: https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/)
```
$ kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/rbac-kdd.yaml
$ kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml
```
