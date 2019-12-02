# Kubernetes lab - Ingress

### Install Ingress Controler on Bare Metal
```
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/mandatory.yaml
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/baremetal/service-nodeport.yaml
```

### Create Deployement & Service
[deploy_nginx.yml](deploy_nginx.yml)
[deploy_nginx2.yml](deploy_nginx2.yml)

```
$ kubectl apply -f deploy_nginx.yml
$ kubectl apply -f deploy_nginx2.yml
```

### Create Ingress
[ingress.yml](ingress.yml)

```
$ kubectl apply -f ingress.yml
```

### Wait for ingress address
```
$ kubectl get ingress -w
```

### Curl ingress
```
$ curl --resolve 'k8straining.com:80:<ip of ingress loadbalancer>'  http://k8straining.com/foo
$ curl --resolve 'k8straining.com:80:<ip of ingress loadbalancer>'  http://k8straining.com/bar
```
