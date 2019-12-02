# Kubernetes lab3 - Services

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

### Reuse pod_nginx2.yml from TP2
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

### Create clusterip Service manifest targetting label app: nginx
svc_nginx.yml
```
kind: Service
apiVersion: v1
metadata:
  name: svc-nginx
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
```

### Apply pods and service manifests
```
$ kubectl apply -f pod_nginx2.yml
$ kubectl apply -f svc_nginx.yml
```

### List Services
```
$ kubectl get services
```

### Curl nginx service IP
```
$ curl <IP>
```

### Create new nginx pod manifest on other version
pod_nginx3.yml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: index-html2
data:
  index.html: |
      Nginx V1.15.9

---
apiVersion: v1
kind: Pod
metadata:
  name: nginx2
  labels:
      app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.15.9
    ports:
    - containerPort: 80
    volumeMounts:
    - mountPath: /usr/share/nginx/html
      readOnly: true
      name: index-html
  volumes:
  - name: index-html
    configMap:
      name: index-html2
      items:
        - key: index.html
          path: index.html
```

### Apply it
```
$ kubectl apply -f pod_nginx3.yml
```

### Curl nginx service IP several time
```
$ curl <IP>
$ ...
```

### Question?
- What is the result?

### Change label in pod_nginx3.yml "app: nginx" -> "app: nginx2" and apply
```
# edit pod_nginx3.yml file
$ kubectl apply -f pod_nginx3.yml
```

### Curl nginx service IP several time
```
$ curl <IP>
$ ...
```

### Question?
- What is the result?
