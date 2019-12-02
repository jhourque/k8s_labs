# Kubernetes lab - Ingress

### Install Ingress Controler on Bare Metal
```
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/mandatory.yaml
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/baremetal/service-nodeport.yaml
```

### Create Deployement & Service
[deploy_nginx.yml](deploy_nginx.yml)
[deploy_nginx2.yml](deploy_nginx2.yml)

### Create Ingress
ingress.yml
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-nginx
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: k8straining.com
    http:
      paths:
      - path: /foo
        backend:
          serviceName: svc-nginx
          servicePort: 80
      - path: /bar
        backend:
          serviceName: svc-nginx2
          servicePort: 80
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
