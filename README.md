# Name

Overview

## Description

## Demo
```
$ gcloud container clusters create shinyay-cluster --scopes cloud-platform \
    --num-nodes 1 \
    --enable-stackdriver-kubernetes \
    --zone us-central1-c
```

### ClusterIP
```
$ kubectl get service -o json|jq -r .items[0].spec.clusterIP
```

```
$ kubectl exec -it POD-NAME -- sh
$ apk add --no-cache curl
$ curl CLUSTER-IP:80
```

### NodePort
```
$ kubectl get nodes --output wide
$ gcloud compute firewall-rules create test-node-port --allow tcp:31271
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
