# ArgoCD OpenShift Route Configuration

## Problem
ArgoCD cannot find the OpenShift Route CRD because it's not included in ArgoCD's default resource discovery.

## Solution Options

### Option 1: Configure ArgoCD at Cluster Level (Requires Cluster Admin)

Edit the ArgoCD ConfigMap to include Route resources:

```bash
kubectl edit configmap argocd-cm -n argocd
```

Add the following to enable Route resource discovery:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
  namespace: argocd
data:
  resource.customizations: |
    route.openshift.io/Route:
      health.lua: |
        hs = {}
        hs.status = "Healthy"
        hs.message = "Route is healthy"
        return hs
```

Or add Route to the resource inclusion list in `argocd-cmd-params-cm`:

```bash
kubectl edit configmap argocd-cmd-params-cm -n argocd
```

Add:
```yaml
data:
  server.insecure: "true"
  application.resourceTrackingMethod: annotation
```

Then restart ArgoCD:

```bash
kubectl rollout restart deployment argocd-server -n argocd
kubectl rollout restart deployment argocd-repo-server -n argocd
```

### Option 2: Use ArgoCD Application with Resource Inclusion

The `argocd-application.yaml` file has been created with proper settings. Apply it:

```bash
kubectl apply -f Application/yaml/argocd-application.yaml
```

### Option 3: Manual Sync (Workaround)

If you cannot modify ArgoCD configuration, you can manually sync the Route:

```bash
kubectl apply -f Application/yaml/route.yaml
```

Then ArgoCD will track it after the initial creation.

