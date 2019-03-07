# Quotas and Limits

[Quota and Limit Docs](https://docs.openshift.com/container-platform/3.11/dev_guide/compute_resources.html)

## Quota - CPU Requests

If you want to limit how much CPU an entire project can consume, *cpu requests, but not limits*:

```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-resources
spec:
  hard:
    requests.cpu: "1" 
    requests.memory: 1Gi 
```
With this quota set, each container will have to set a cpu **request**, but it will not be required to have a **limit**, so it will be burstable.

## Limit Ranges

If you don't want to set *limits and requests* on each container, you can create defaults for the project:

```
apiVersion: "v1"
kind: "LimitRange"
metadata:
  name: "core-resource-limits" 
spec:
  limits:
    - type: "Container"
      default:
        memory: "512Mi"
      defaultRequest:
        cpu: "100m" 
        memory: "512Mi" 
```
This will set default CPU and Memory *requests* to `100m` and `512Mi` respectively.  It will also default the memory *limit* to `512Mi`.