# Kubernetes lab - PV/PVC

## Use AWS Kubernetes Cluster for this lab

### Label each node with node1 and node2
```
$ kubectl get nodes
$ kubectl label nodes <node name 1> nodename=node1
$ kubectl label nodes <node name 2> nodename=node2
```

### Create PV with StorageClass Local
sc-local.yml
```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

### Create /local path in node1
```
node1$ sudo mkdir /local
node1$ sudo chmod 777 /local
```

### Create PV in /local path
pv-local.yml
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-storage
  local:
    path: /local
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: nodename
          operator: In
          values:
          - node1
```

### Create PVC from local storage
pvc-local.yml
```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: local-claim
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: local-storage
  resources:
    requests:
      storage: 1Gi
```

### Test it with pod
pod-local.yml
```yaml
kind: Pod
apiVersion: v1
metadata:
  name: ubuntu
spec:
  containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep"]
      args: ["3600"]

      volumeMounts:
      - mountPath: "/local"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: local-claim
```

### Apply and check
```
$ kubectl apply -f sc-local.yml
$ kubectl apply -f pv-local.yml
$ kubectl apply -f pvc-local.yml
$ kubectl get pvc
$ kubectl apply -f pod-local.yml
$ kubectl get pvc
```

### Create StorageClass AWSElasticBlockStore
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
```

### Create PVC 
pvc-ebs.yml
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: claim1
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: slow
  resources:
    requests:
      storage: 8Gi
```

### Apply and check
```
$ kubectl apply -f sc-ebs.yml
$ kubectl apply -f pvc-ebs.yml
$ kubectl get pvc
```
