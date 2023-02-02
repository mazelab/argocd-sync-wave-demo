# ArgoCD Sync-Wave Demo

The health assessment of Application CRD has been removed in version 1.8. If you want to use the app-of-app pattern in your project, you need to understand the following implications.

## Unpatched

![unpatched](images/wavetest-unpatched.png)

- All Application CRs are always in the healthy state if the child is also an application.
- Sync-Wave configurations in Applications do not take effect in terms of waiting.
- The Sync-Wave order only pauses the default configured wave-to-wave duration.

## Patched

![patched](images/wavetest-patched.png)

- Application CRs get the health state from there childrens.
- Sync-Wave configurations in Applications take effect in term of waiting.
- An errored wave blocks the following waves.

## Installing

Add an Application like the following example in your ArgoCD project.

Checklist:

- namespace in the example is `argocd` => must exist

```yaml
---
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: argo-cd-wave-test
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: default
  source:
    repoURL: https://github.com/mazelab/argocd-sync-wave-demo.git
    path: argocd
    targetRevision: main
    helm:
      values: |
        repo:
          revision: main
          repoURL: https://github.com/mazelab/argocd-sync-wave-demo.git
  destination:
    namespace: argocd
    server: https://kubernetes.default.svc
  syncPolicy:
    syncOptions:
      - CreateNamespace=true
      - PruneLast=true
    automated:
      selfHeal: true
      prune: true
    retry:
      limit: 2
```
