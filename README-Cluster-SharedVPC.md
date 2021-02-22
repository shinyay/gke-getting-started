# GKE Getting Started
- [README - Top](README.md)

## Clusters with Shared VPC
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