---
title: Prometheus
menu:
  notes:
    name: Prometheus
    identifier: notes-prometheus
    weight: 30
    parent: notes-monitoring
---
# Prometheus Notes

<!-- Forward Traffic-->
{{< note title="PromQL return 0 instead of ‘no data’" >}}

Add following to the end of your query
```promql
OR on() vector(0)
```

Prometheus uses label matching in expressions. If your expression returns anything with labels, it won't match the time series generated by vector(0). In order to make this possible, it's necessary to tell Prometheus explicitly to not trying to match any labels by adding on().