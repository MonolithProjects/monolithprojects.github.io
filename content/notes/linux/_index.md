---
title: Linux Notes
menu:
  notes:
    name: Linux
    identifier: notes-linux
    weight: 10
---
# Linux Notes

<!-- Forward Traffic-->
{{< note title="Forward from localhost to the outside world" >}}

```bash
sudo socat -d -d TCP-L:8500,bind=<node IP address>,fork TCP:localhost:8500
```
