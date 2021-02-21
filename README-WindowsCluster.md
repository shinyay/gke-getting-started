# GKE Getting Started
## Windows Cluster
### Cluster with Linux Node Pool for Add-ons
```
$ gcloud container clusters create shinyay-win-cluster \
    --zone us-central1-c \
    --enable-ip-alias
```

### Windows Server Node Pool
```
$ gcloud container node-pools create win-node-pool \
    --zone us-central1-c \
    --cluster shinyay-win-cluster \
    --image-type WINDOWS_SAC \
    --no-enable-autoupgrade \
    --machine-type n1-standard-2
```

### Webhook for Pod with `kubernetes.io/os: windows`
```
$ kubectl get mutatingwebhookconfigurations
```

### Deploy Windows App
- [deployment-iis.yml](deploying-workloads/windows-app/deployment-iis.yml)
```
$ kubectl apply -f deployment-iis.yml
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iis
  labels:
    app: iis
spec:
  replicas: 3
  selector:
    matchLabels:
      app: iis
  template:
    metadata:
      labels:
        app: iis
    spec:
      nodeSelector:
        kubernetes.io/os: windows
      containers:
      - name: iis-server
        image: mcr.microsoft.com/windows/servercore/iis
        ports:
        - containerPort: 80

```

### Expose Windows App
-[service-iis-lb.yml](deploying-workloads/windows-app/service-iis-lb.yml)
```
$ kubectl apply -f service-iis-lb.yml
```
```
$ kubectl get services iis-lb-service -o json|jq -r .status.loadBalancer.ingress[0].ip
$ open http://(kubectl get services iis-lb-service -o json|jq -r .status.loadBalancer.ingress[0].ip)
```

### Upgrading Windows Server node pools
- [Upgrading Windows Server node pools](https://cloud.google.com/kubernetes-engine/docs/how-to/creating-a-cluster-windows#upgrading_windows_server_node_pools)
  - [Windows container version compatibility](https://docs.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/version-compatibility?tabs=windows-server-1909%2Cwindows-10-1909)
  - [Building Windows Server multi-arch images](https://cloud.google.com/kubernetes-engine/docs/tutorials/building-windows-multi-arch-images)