# Kubernetes lab - StatefulSets

## Use AWS Kubernetes Cluster for this lab

### Small fix on EBS StorageClass
sc-ebs.yml
```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: slow
provisioner: kubernetes.io/aws-ebs
parameters:
  type: io1
  iopsPerGB: "10"
  fsType: ext4
allowedTopologies:
- matchLabelExpressions:
  - key: failure-domain.beta.kubernetes.io/zone
    values:
    - eu-west-1b
    - eu-west-1c
```

### Create StatefulSet with EBS StorageClass
sts-ebs.yml
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx
  serviceName: "nginx"
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: slow
      resources:
        requests:
          storage: 1Gi
```

### Apply and check
```
$ kubectl apply -f sc-ebs.yml
$ kubectl apply -f sts-ebs.yml
$ kubectl get pvc
```

### Scale up
```
$ kubectl scale sts web --replicas=2
```

###
```
```

```
```
