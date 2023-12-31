---
title: Hacks
weight: 210
menu:
  notes:
    name: Hacks
    identifier: notes-kubernetes-hacks
    parent: notes-kubernetes
    weight: 10
---

<!-- Variable -->
{{< note title="Serial tasks" >}}

Remove a finalizer which got stuck.

```bash
kubectl get ns <namespace name> -ojson | jq '.spec.finalizers = []' | kubectl replace --raw "/api/v1/namespaces/<namespace name>/finalize" -f -

```

{{< /note >}}
