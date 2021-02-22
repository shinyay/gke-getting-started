# GKE Getting Started
- [README - Top](README.md)

## Clusters with Shared VPC
### Multiple Projects for Shared VPC
Create projects:

- `Host Project`
  - shinyay-works
- `Service Project`
  - shinyay-service-1
  - shinyay-service-2

```
$ gcloud projects list --filter=shinyay --format='value(PROJECT_ID)'

shinyay-works
shinyay-service-1
shinyay-service-2
```

### Network and Subnet
Create networks:

- **Network** at `Host Project`
  - shared-net
- **Subnet** at `Service Project`
  - tier-1
  - tier-2

```
$ gcloud compute networks create shared-net \
    --subnet-mode custom \
    --project shinyay-works
```

```
$ gcloud compute networks subnets create tier-1 \
    --network shared-net \
    --range 10.0.4.0/22 \
    --region us-central1 \
    --secondary-range tier-1-services=10.0.32.0/20,tier-1-pods=10.4.0.0/14 \
    --project shinyay-works

$ gcloud compute networks subnets create tier-2 \
    --network shared-net \
    --range 172.16.4.0/22 \
    --region us-central1 \
    --secondary-range tier-2-services=172.16.16.0/20,tier-2-pods=172.20.0.0/14 \
    --project shinyay-works
```