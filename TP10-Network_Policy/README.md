# Kubernetes lab - Network Policy

## Use AWS Kubernetes Cluster for this lab

## Create a Netpol which accept traffic only from namespace labelled “production” and is not accessible otherwise.
### Create namespace dev & prod
```
$ kubectl create ns dev
$ kubectl create ns prod
```

### Add purpose label to namespaces
```
$ kubectl label namespace/dev purpose=dev
$ kubectl label namespace/prod purpose=prod
```

### Run nginx instance (in default namespace)
```
$ kubectl run web --image=nginx --labels=app=web --expose --port 80
```

### Run ubuntu in dev & prod
```
$ kubectl run ubuntu --image ubuntu -n dev  -- sleep 3600
$ kubectl run ubuntu --image ubuntu -n prod -- sleep 3600
```

### Install curl in ubuntu (prod) and test nginx
```
$ kubectl get po -n prod |grep ubuntu
$ kubectl exec -it <ubuntu pod name> -n prod -- bash
ubuntu$ apt-get update && apt-get install -y curl
ubuntu$ curl web.default
```

### Install curl in ubuntu (dev) and test nginx
```
$ kubectl get po -n dev |grep ubuntu
$ kubectl exec -it <ubuntu pod name> -n dev -- bash
ubuntu$ apt-get update && apt-get install -y curl
ubuntu$ curl web.default
```

### Create network policy
netpol.yml
```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: only-allow-namespace-prod
spec:
  podSelector:
    matchLabels:
      app: web
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          purpose: prod
```

### Apply it
```
$ kubectl apply -f netpol.yml
```

### Test curl again from each ubuntu pod
