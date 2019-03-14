# Kubernetes lab0 - Docker

## Vagrantfile with docker install
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
    usermod -aG docker vagrant
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

### Pull nginx container
```
$ docker pull nginx
```

### Start nginx container (map port 80 to 8000)
```
docker run --name nginx -d -p 8000:80 nginx
```

### Curl nginx from inside (in container)
```
$ docker exec -it nginx sh
nginx$ /usr/bin/apt-get update
nginx$ /usr/bin/apt-get install -y curl
nginx$ curl localhost
```

### Curl nginx from outside (from host)
```
$ curl localhost:8000
```

### Kill nginx container
```
$ docker kill nginx
```

### List container stopped
```
$ docker ps -a
```

### Restart a container stopped
```
$ docker start nginx
```

### Destroy container
```
$ docker kill nginx
$ docker rm nginx
```
Or
```
$ docker rm -f nginx
```

###List images
```
$ docker images
```

### Remove image
```
$ docker rmi nginx
```

### Question ?
A new version of the web site is available:
- How do you update your website container without rebuild your container? 
- How would you Manage scaling?
- How would you deploy a new version with zero down-time?

## Dockerfile manipulation
Dockerfile:
```
FROM nginx
RUN echo "hello from nginx" > /usr/share/nginx/html/index.html
```

### Build image
```
$ docker build . -t my-nginx:latest
```

### Create tag v1 and tag latest on new container
```
$ docker tag my-nginx:latest my-nginx:v1
```

### Share your docker image with other developers in your team (use external registry)
User1:
```
$ docker tag my-nginx:v1 <my-registry>:<port>/my-nginx:v1
$ docker push <my-registry>:<port>/my-nginx:v1
```

User2:
```
$ docker pull <my-registry>:<port>/my-nginx:v1
```

### Run container with source network and check container IP
```
$ docker run -d --name=bb1 busybox sleep 3600
$ docker run -d --name=bb2 --net=container:bb1 busybox sleep 3600
$ docker exec -it bb1 ifconfig
$ docker exec -it bb2 ifconfig
```

### Question?
- What do you observe ?
