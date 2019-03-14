# Kubernetes lab4 - Deployments

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

### Create deployment with 2 nginx pod
deploy_nginx.yml
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: index-html
data:
  index.html: |
      Nginx V1.14.0

---
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 2
  template:
    metadata:
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

### Apply it
```
$ kubectl apply -f deploy_nginx.yml
```

### Create Service (same as TP3)
svc_nginx.yml
```yaml
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

### Apply it
```
$ kubectl apply -f svc_nginx.yml
```

### Curl service
```
$ curl <IP of nginx service>
```

### Scale up deployment
```
$ kubectl scale deploy/nginx --replicas=4
```

### List pods
```
$ kubectl get pod -o wide
```

### Update nginx version in manifest to version 1.15.0
```
# Edit deploy_nginx.yml
```

### Apply new version and quickly monitor pod (kubectl get po -w)
```
$ kubectl apply -f deploy_nginx.yml
$ kubectl get po -w -o wide
```

### Question?
- What happened?

### Repeat upgrade with version 1.15.9
```
...
```

### List change history 
```
$ kubectl rollout history deploy/nginx
```

### Rollback to version 1
```
$ kubectl rollout undo deploy/nginx --to-revision=1
```

## Canary deployment upgrade
### Copy deployment manifests and change replicas to 1
deploy_nginx_canary.yml
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: index-html2
data:
  index.html: |
      Nginx V1.15.0

---
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: nginx2
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.15.0
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

### Deploy it
```
$ kubectl apply -f deploy_nginx_canary.yml
```

### Curl service several time
```
$ curl <IP of nginx service>
$ ...
```

### Increase number of replicas in new deployement and decrease old one
```
$ kubectl scale deploy/nginx2 --replicas=2
$ kubectl scale deploy/nginx --replicas=0
```

### Destroy old deployment
```
$ kubectl delete deploy/nginx
```

## Blue/Green deployment upgrade
### Create Blue and Green version of deployment
deploy_nginx_blue.yml
```yaml
apiVersion: v1
kind: ConfigMap
retadata:
  name: index-html-blue
data:
  index.html: |
      Nginx V1.14.0

---
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: nginx-blue
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx_blue
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
          name: index-html-blue
          items:
            - key: index.html
              path: index.html

```

deploy_nginx_green.yml
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: index-html-green
data:
  index.html: |
      Nginx V1.15.0

---
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: nginx-green
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx_green
    spec:
      containers:
      - name: nginx
        image: nginx:1.15.0
        ports:
        - containerPort: 80
        volumeMounts:
        - mountPath: /usr/share/nginx/html
          readOnly: true
          name: index-html
      volumes:
      - name: index-html
        configMap:
          name: index-html-green
          items:
            - key: index.html
              path: index.html
```

###  Apply both
```
$ kubectl apply -f deploy_nginx_blue.yml
$ kubectl apply -f deploy_nginx_green.yml
```

### Create service targeting app: nginx_blue
svc_nginx.yml
```yaml
kind: Service
apiVersion: v1
metadata:
  name: svc-nginx
spec:
  type: ClusterIP
  selector:
    app: nginx_blue
  ports:
  - protocol: TCP
    port: 80
```

### Apply and test it
```
$ kubectl apply -f svc_nginx.yml
$ kubectl get svc
$ cyyurl ...
```

### Update svc target to app: nginx_green
```
# edit svc_nginx.yml file
$ kubectl apply -f svc_nginx.yml
```

### Curl service
```
$ curl ...
```
