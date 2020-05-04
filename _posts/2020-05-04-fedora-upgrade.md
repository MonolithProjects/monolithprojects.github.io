---
layout: post
title:  "Fedora upgrade"
categories: [english]
---

Since the Fedora upgrade is a ceremony which i am doing every 6 months, and always forgetting how i did it, it's probably good idea to write it down and create kind of `cheat sheet`.

## DNF System Upgrade

Not so long time ago, theere were only two ways how to upgrade Fedora distro. Eather to reinstall it from the scratch, or use a special script and pray that everything will go right.  
But these days we can do it more or less in the safe way and use DNF System Upgrade plugin. Anyway... every system upgrade is a potentially risky.
So maybe it is not a bad idea to backup your data (i am not doing this coz i don't care).

## Performing System Upgrade

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

6. Success and profit :-)

---
