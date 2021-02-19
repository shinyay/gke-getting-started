# GKE Getting Started

Overview

## Description
### Before Clusters
#### VPC Network
```
$ gcloud compute networks create shinyay-network \
    --subnet-mode custom
```

#### Subnet
##### Primary IP Range
```
$ gcloud compute networks subnets create shinyay-us-central-192 \
    --network shinyay-network \
    --region us-central1 \
    --range 192.168.1.0/24
```

##### Secondary IP Range
```
$ gcloud compute networks subnets update shinyay-us-central-192 \
    --region us-central1 \
    --add-secondary-ranges=shinyay-pod=10.0.0.0/16
```


### Creating Clusters
![cluster-architecture.](https://cloud.google.com/kubernetes-engine/images/cluster-architecture.svg)

The types of available clusters include: `zonal` (single-zone or multi-zonal) and `regional`.

- Zonal Cluster
  - Single-zone Cluster
  - Multi-zonal Cluster
- Regional Cluster
- Private Cluster
- Windows Cluster

#### Single-zone clusters
A single-zone cluster has a single control plane running in one [zone](https://cloud.google.com/compute/docs/regions-zones#available).

```
$ gcloud container clusters create shinyay-cluster \
    --zone us-central1-c
```

#### Multi-zone clusters
A multi-zonal cluster has a single replica of the control plane running in a single zone, and has nodes running in multiple zones. 

```
$ gcloud container clusters create shinyay-cluster \
    --zone us-central1-c \
    --node-locations us-central1-a,us-central1-b,us-central1-c
```

#### Regional clusters
A regional cluster has multiple replicas of the control plane, running in multiple zones within a given [region](https://cloud.google.com/compute/docs/regions-zones#available).

```
$ gcloud container clusters create shinyay-cluster \
    --region us-west1
```

#### Private Cluster
- Public endpoint access disabled
- Public endpoint access enabled, authorized networks enabled
- Public endpoint access enabled, authorized networks disabled

##### Public endpoint access disabled
```
$ gcloud container clusters create shinyay-cluster \
    --zone us-central1-c \
    --master-ipv4-cidr 172.16.0.0/28 \
    --enable-ip-alias \
    --enable-private-nodes \
    --enable-private-endpoint
```

##### Public endpoint access enabled, authorized networks enabled
```
$ gcloud container clusters create shinyay-cluster \
    --zone us-central1-c \
    --master-ipv4-cidr 172.16.0.0/28 \
    --enable-ip-alias \
    --enable-private-nodes \
    --enable-master-authorized-networks \
    --master-authorized-networks [AUTHORIZED_IPV4_ADDRESS]/32
```

We can get your local ip:
```
$ curl -s ip4only.me/api/ |awk -F , '{print $2}'
```

We can also update `Master Authorized Networks`
```
$ gcloud container clusters update shinyay-cluster \
    --enable-master-authorized-networks \
    --master-authorized-networks [AUTHORIZED_IPV4_ADDRESS]/32
```

##### Public endpoint access enabled, authorized networks disabled
```
$ gcloud container clusters create shinyay-cluster \
    --zone us-central1-c \
    --master-ipv4-cidr 172.16.0.0/28 \
    --enable-ip-alias \
    --enable-private-nodes \
    --no-enable-master-authorized-networks
```

##### Outbound Traffic with Cloud NAT
1-CASE: Create subnet with GKE Cluster
```
$ gcloud container clusters create [CLUSTER]] \
    --create-subnetwork name=[SUBNET] \
    :
    :
```

1-CASE: Use existing subnet with GKE Cluster

1-1. Create VPC
```
$  gcloud compute networks create [NETWORK] \
     --subnet-mode custom
```

1-2. Create Subnet
```
$ gcloud compute networks subnets create subnet-us-east-192 \
   --network shinyay-network \
   --region [REGION:us-central1] \
   --range [SUBNET_RANGE:192.168.1.0/24]
```
1-3. Use subnet
```
$ gcloud container clusters create [CLUSTER] \
    --subnetwork [SUBNET] \
    :
    :
```

2. Cloud Router
```
$ gcloud compute routers create shinyay-router \
    --network default \
    --region us-central1
```

3. Cloud NAT
```
$ gcloud compute routers nats create shinyay-nat \
    --router shinyay-router \
    --router-region us-central1 \
    --auto-allocate-nat-external-ips \
    --nat-all-subnet-ip-ranges \
    --enable-logging
```

4. Confirm Outbound Traffic
```
$ kubectl apply -f exposing-with-services/loadbalancer/
$ kubectl exec -it [POD] -- apk add --no-cache curl
$ kubectl exec -it [POD] -- curl httpbin.org/ip
```

#### GKE Version (Release Channel / Static Version)
##### Release Channel
- `rapid`
- `regular`
- `stable`
- `None`

```
$ gcloud container clusters create shinyay-cluster \
    --release-channel rapid
    --zone us-central1-c
```

Available GKE versions in Channel
```
$ gcloud container get-server-config --format "yaml(channels)" \
    --zone us-central1-c
```

### Windows Cluster
#### Cluster with Linux Node Pool for Add-ons
```
$ gcloud container clusters create shinyay-win-cluster \
    --zone us-central1-c \
    --enable-ip-alias
```

#### Windows Server Node Pool
```
$ gcloud container node-pools create win-node-pool \
    --zone us-central1-c \
    --cluster shinyay-win-cluster \
    --image-type WINDOWS_SAC \
    --no-enable-autoupgrade \
    --machine-type n1-standard-2
```

#### Webhook for Pod with `kubernetes.io/os: windows`
```
$ kubectl get mutatingwebhookconfigurations
```

#### Deploy Windows App
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

#### Expose Windows App
-[service-iis-lb.yml](deploying-workloads/windows-app/service-iis-lb.yml)
```
$ kubectl apply -f service-iis-lb.yml
```
```
$ kubectl get services iis-lb-service -o json|jq -r .status.loadBalancer.ingress[0].ip
$ open http://(kubectl get services iis-lb-service -o json|jq -r .status.loadBalancer.ingress[0].ip)
```

#### Upgrading Windows Server node pools
- [Upgrading Windows Server node pools](https://cloud.google.com/kubernetes-engine/docs/how-to/creating-a-cluster-windows#upgrading_windows_server_node_pools)
  - [Windows container version compatibility](https://docs.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/version-compatibility?tabs=windows-server-1909%2Cwindows-10-1909)
  - [Building Windows Server multi-arch images](https://cloud.google.com/kubernetes-engine/docs/tutorials/building-windows-multi-arch-images)

### Configuring Cluster Network
#### Network Configuration
##### Routes-based Cluster
Routes-based cluster uses custom static routes in a VPC network.
- [Static routes](https://cloud.google.com/vpc/docs/routes#static_routes)

```
$ gcloud container clusters create shinyay-cluster-route-based \
    --no-enable-ip-alias \
    --zone us-central1-c
```
##### VPC-native Cluster
VPC-native cluster uses alias IP address ranges.
- [Alias IP](https://cloud.google.com/vpc/docs/alias-ip)

Benefits of VPC-native clusters:
- Pod IP addresses are natively routable within the cluster's VPC network
- Pod IP addresses are reserved in the VPC before the Pods are created
- Pod IP address ranges do not depend on custom static routes
  - automatically-generated [subnet routes](https://cloud.google.com/vpc/docs/routes#subnet-routes)
- You can create firewall rules for Pod IP address ranges

|Subnet Range|Description|
|------------|-----------|
|Node|Node IP addresses are assigned from **Primary IP** range of subnet of cluster|
|Pod|Pod IP addresses are taken from subnet's **Secondary IP address range for Pods**|
|Service|Service (cluster IP) addresses are taken from subnet's **Secondary IP address range for Services**|

```
$ gcloud container clusters create shinyay-cluster-vpc-native \
    --zone us-central1-c \
    --enable-ip-alias \
    --create-subnetwork name=cluster-subnet,range=10.0.0.0/24
    --cluster-ipv4-cidr /16 \
    --services-ipv4-cidr /22
```

![vpc-native](https://user-images.githubusercontent.com/3072734/108294188-bd6f6180-71d8-11eb-8991-a7cdac45ceae.png)

#### Load balancing
##### Exposing App with Services
###### ClusterIP

Create the Deployment and Service
```
$ kubectl apply -f deployment-cip.yml
$ kubectl apply -f service-cip.yml
```

Retrieve ClusterIP
```
$ kubectl get service -o json|jq -r .items[0].spec.clusterIP
```

Access Service
```
$ kubectl get pods
$ kubectl exec -it POD-NAME -- sh
$ apk add --no-cache curl
$ curl CLUSTER-IP:80
```

###### NodePort

Create the Deployment and Service
```
$ kubectl apply -f deployment-np.yml
$ kubectl apply -f service-np.yml
```

Retrieve NodePort
```
$ kubectl get services np-service -o json|jq -r .spec.ports[].nodePort
```

Create Firewall Rule
```
$ gcloud compute firewall-rules create test-node-port --allow tcp:NODEPORT
```

Show External IP
```
$ kubectl get nodes -o wide
```

Access Service
```
$ curl EXTERNAL-IP:NODEPORT
```

###### LoadBalancer

Create the Deployment and Service
```
$ kubectl apply -f deployment-lb.yml
$ kubectl apply -f service-lb.yml
```

Retrieve Port
```
$ kubectl get services np-service -o json|jq -r .spec.ports[].port
```

Show External IP
```
$ kubectl get services -o wide
```

Access Service
```
$ curl EXTERNAL-IP:PORT
```

#### Ingress for GKE
##### Container-native Load balancing
**Container-native load balancing** enables load balancers to target Pods directly and to distribute traffic to Pods.

![neg](https://user-images.githubusercontent.com/3072734/108304619-bef65500-71eb-11eb-8d35-ec7a61b03c7a.png)

This configicuration model is **[Network Endpoint Group](https://cloud.google.com/load-balancing/docs/negs)(NEG)**


The  Service's annotation, `cloud.google.com/neg: '{"ingress": true}'`, enables container-native load balancing.
```yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    cloud.google.com/neg: '{"ingress": true}'
```

Deploy app and service:
```
$ kubectl apply -f ingress-for-gke/container-native-loadbalancing/deloyment-neg.yml
$ kubectl apply -f ingress-for-gke/container-native-loadbalancing/service-neg.yml
$ kubectl apply -f ingress-for-gke/container-native-loadbalancing/ingress-neg.yml
```

Confirm NEG app
```
$ curl (kubectl get ingress -o json|jq -r .items[0].status.loadBalancer.ingress[0].ip)
```

## Demo
### Clean up
```
$ gcloud container clusters delete NAME [--region=REGION | --zone=ZONE]
```



## Features

- feature:1
- feature:2

## Requirement

## Usage

## Installation

## References

## Licence

Released under the [MIT license](https://gist.githubusercontent.com/shinyay/56e54ee4c0e22db8211e05e70a63247e/raw/34c6fdd50d54aa8e23560c296424aeb61599aa71/LICENSE)

## Author

[shinyay](https://github.com/shinyay)
