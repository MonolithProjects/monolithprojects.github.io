---
title: System
weight: 210
menu:
  notes:
    name: System
    identifier: notes-system
    parent: notes-linux
    weight: 10
---
<!-- Forward Traffic-->
{{< note title="Forward from localhost to the outside world" >}}

```bash
sudo socat -d -d TCP-L:8500,bind=<node IP address>,fork TCP:localhost:8500
```

{{< /note >}}

<!-- Fedora Upgrade -->
{{< note title="Fedora Upgrade" >}}

1. First run this command and reboot your computer:

   ```bash
   sudo dnf upgrade --refresh
   ```

2. Install the dnf-plugin-system-upgrade package if it is not currently installed:

   ```bash
   sudo dnf install dnf-plugin-system-upgrade
   ```

3. Download the updated packages:

   ```bash
   sudo dnf system-upgrade download --refresh --releasever=<your destination version>
   ```

4. If some of your packages have unsatisfied dependencies, the upgrade will refuse to continue until you run it again with an extra `--allowerasing` option.  

5. Run the upgrade process:

   ```bash
   sudo dnf system-upgrade reboot
   ```