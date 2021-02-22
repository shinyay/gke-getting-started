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