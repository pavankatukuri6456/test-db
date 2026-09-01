# ResourceQuota restart demonstration

This example demonstrates that an existing pod is not evicted when a
`ResourceQuota` is reduced, but a replacement pod can be rejected after the
Deployment is restarted.

The Deployment requests `100m` CPU and `64Mi` memory. The initial quota allows
those requests. The restricted quota permits only `50m` CPU and `32Mi` memory,
so Kubernetes admission rejects the replacement pod.

## 1. Start the pod under the initial quota

Run these commands from this directory:

```bash
kubectl apply -f namespace.yaml
kubectl apply -f resource-quota.yaml
kubectl apply -f deployment.yaml

kubectl rollout status deployment/quota-demo -n resource-quota-demo
kubectl get pods -n resource-quota-demo
kubectl get resourcequota workload-quota -n resource-quota-demo
```

The pod should show `Running`.

## 2. Reduce the quota

```bash
kubectl apply -f resource-quota-restricted.yaml
kubectl get resourcequota workload-quota -n resource-quota-demo
```

The existing pod remains running because changing a quota does not evict
existing pods.

## 3. Restart the Deployment

```bash
kubectl rollout restart deployment/quota-demo -n resource-quota-demo
kubectl get pods -n resource-quota-demo --watch
```

This Deployment intentionally uses the `Recreate` strategy. Kubernetes removes
the old pod before attempting to create its replacement. The replacement is
rejected because its resource requests exceed the restricted quota, so no new
pod reaches `Running`.

Press `Ctrl+C`, then inspect the rejection:

```bash
kubectl describe deployment quota-demo -n resource-quota-demo
kubectl get events -n resource-quota-demo \
  --sort-by=.metadata.creationTimestamp
```

Look for a `FailedCreate` event containing `exceeded quota`.

Kubernetes may show no pending pod at all. This is expected: ResourceQuota is
enforced by API admission, so the rejected Pod object is never created.

## 4. Restore the quota

```bash
kubectl apply -f resource-quota.yaml
kubectl rollout status deployment/quota-demo -n resource-quota-demo
kubectl get pods -n resource-quota-demo
```

The Deployment controller retries automatically and the replacement pod should
become `Running` after the quota is restored.

## Cleanup

```bash
kubectl delete namespace resource-quota-demo
```
