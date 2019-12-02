# Kubernetes lab - Scheduling

## Use AWS Kubernetes Cluster for this lab

### Label each node with node1 and node2
```
$ kubectl get nodes
$ kubectl label nodes <node name 1> nodename=node1
$ kubectl label nodes <node name 2> nodename=node2
```

### Create deployment with affinity to node1
deploy_nginx.yml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: nodename
                operator: In
                values:
                - node1
      containers:
      - name: nginx
        image: nginx:1.12.1
        ports:
        - containerPort: 80
```

### Check pods placement
```
$ kubectl get po -o wide
```

### Taint node1 NoSchedule and check pods
```
$ kubectl taint nodes <node name 1> key=value:NoSchedule
$ kubectl get po -o wide
```

### Question?
- What's happening?

### Delete deployment and reapply it
```
$ kubectl delete -f deploy_nginx.yml
$ kubectl apply -f deploy_nginx.yml
$ kubectl get po -o wide
```

### Question?
- What's happening?

### Remove NoSchedule taint from node1
```
$ kubectl taint nodes <node name 1> key:NoSchedule-
$ kubectl get po -o wide
```

### Question?
- What's happening?
