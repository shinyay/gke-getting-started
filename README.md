# GKE Getting Started

Overview

## Description
### Creating Clusters
![cluster-architecture.](https://cloud.google.com/kubernetes-engine/images/cluster-architecture.svg)

The types of available clusters include: `zonal` (single-zone or multi-zonal) and `regional`.

- Zonal clusters
  - Single-zone clusters
  - Multi-zonal clusters
- Regional Clusters

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

### Configuring Cluster Network
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


## Demo




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
