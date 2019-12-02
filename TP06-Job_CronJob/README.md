# Kubernetes lab - Job/CronJob

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

### Create simple job manifest: pi compute
job.yml
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
```

### Apply it
```
$ kubectl apply -f job.yml
```

### Watch execution
```
$ kubectl get po -w
```

### Check result when execution is completed
```
$ kubectl logs <pi job pod>
```

### Create simple cronjob manifest: echo hello with date each minute
cronjob.yml
```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```

### Apply it
```
$ kubectl apply -f cronjob.yml
```

### Watch execution
```
$ kubectl get po -w
```

### Check result when execution is completed
```
$ kubectl logs <hello cronjob pod>
```
