---
title: Fish Variables
weight: 210
menu:
  notes:
    name: Variables
    identifier: notes-fish-variables
    parent: notes-fish
    weight: 10
---

<!-- Variable -->
{{< note title="Environmant Variable" >}}

```fish

set -gx KUBECONFIG ~/.kube/my-cluster

```

{{< /note >}}

<!-- Condition -->
{{< note title="Remove Env variable" >}}

```fish

set -e KUBECONFIG

```

{{< /note >}}