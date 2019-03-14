# Kubernetes lab - DaemonSet

## Vagrantfile with docker and kubernetes install and kubeadm setup with callico
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
    kubeadm init --pod-network-cidr=192.168.0.0/16
    mkdir -p ~vagrant/.kube
    cp -i /etc/kubernetes/admin.conf ~vagrant/.kube/config
    chown vagrant:vagrant ~vagrant/.kube/config
    su vagrant -c "kubectl taint node ubuntu-bionic node-role.kubernetes.io/master:NoSchedule-"
    su vagrant -c "kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/rbac-kdd.yaml"
    su vagrant -c "kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml"
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

### Check DaemonSet already running
```
$ kubectl get ds -n kube-system
```

### Create simple daemonset manifest: perdiodic check /proc/meminfo (memfree) in container
>>>
tips: add spec: "hostNetwork: true" & "hostPID: true" to access /proc/meminfo
>>>

daemonset.yml
```yaml
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: proc-daemon
spec:
  template:
    metadata:
      labels:
        name: proc-daemon
    spec:
      hostNetwork: true
      hostPID: true
      containers:
        - name: busybox
          image: busybox
          args:
            - /bin/sh
            - -c
            - while true; do grep MemFree /proc/meminfo; sleep 10; done
          securityContext:
            privileged: true
```

### Apply it
```
$ kubectl apply -f daemonset.yml
```

### Check result when execution is completed
```
$ kubectl logs <daemonset pod>
```

