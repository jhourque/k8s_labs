# Kubernetes lab2 - Pods

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
    su vagrant -c "kubectl apply -f https://docs.projectcalico.org/v3.10/manifests/calico.yaml"
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

### Create manifest for a single nginx pod
pod_nginx.yml:
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx-1.14.0
spec:
  containers:
  - name: nginx
    image: nginx:1.14.0
    ports:
    - containerPort: 80
```

### Apply manifest
```
$ kubectl apply -f pod_nginx.yml
```

### List pods
```
$ kubectl get pods
```

### Delete pod
```
$ kubectl delete -f pod_nginx.yml
```

### Restart pod
```
$ kubectl apply -f pod_nginx.yml
```

### change nginx version to 1.15.9
```
# change version in pod_nginx.yml
$ kubectl apply -f pod_nginx.yml
```

### Question?
- what happened?

### Run sh in nginx pod
```
$ kubectl exec -it nginx sh
```

### Check process running
```
nginx$ /usr/bin/apt update
nginx$ /usr/bin/apt install -y procps
nginx$ ps aux
```

### Check process on master
```
$ ps aux |grep nginx
```

### Kill nginx process
```
$ sudo kill <nginx process>
```

### Question?
- Check process, what happens?

### Create namespace nstest
```
$ kubectl create namespace nstest
```

### Apply nginx pod manifest in nstest namespace
```
$ kubectl apply -f pod_nginx.yml -n nstest
```

### List pods in each namespace
```
$ kubectl get po -n nstest
$ kubectl get po -n default
```

### Use configmap to customize your pod
pod_nginx2.yml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: index-html
data:
  index.html: |
      Nginx V1.14.0

---
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
      app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.0
    ports:
    - containerPort: 80
    volumeMounts:
    - mountPath: /usr/share/nginx/html
      readOnly: true
      name: index-html
  volumes:
  - name: index-html
    configMap:
      name: index-html
      items:
        - key: index.html
          path: index.html
```

### test pod_nginx2.yml
```
$ kubectl delete -f pod_nginx.yml
$ kubectl apply -f  pod_nginx2.yml
```
