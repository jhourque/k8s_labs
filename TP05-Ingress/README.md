# Kubernetes lab - Ingress

## Use AWS Kubernetes Cluster for this lab

### Install Ingress Controler on Bare Metal
```
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/mandatory.yaml
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/provider/aws/service-l4.yaml
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/provider/aws/patch-configmap-l4.yaml
```

### Create Deployement & Service
deploy_nginx.yml
```yaml
kind: Service
apiVersion: v1
metadata:
  name: svc-nginx
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
---
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

### Get IP address of LoadBalancer
```
$ nslookup <loadbalancer fqdn of ingress address>
```

### Curl ingress
```
$ curl --resolve 'k8straining.com:80:<ip of ingress loadbalancer>'  http://k8straining.com/foo
```
