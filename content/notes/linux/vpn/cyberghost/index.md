---
title: CyberGhost
weight: 210
menu:
  notes:
    name: CyberGhost
    identifier: cyberghost
    parent: notes-vpn
    weight: 30
---

{{< note title="CyberGhost in Fedora" >}}

Currently the installation script for Cyber Ghost VPN client does not support Fedora 32. Since the `cyberghostvpn` client works
on Fedora 32 just fine, let see how we can make the installation to work.  

### How to do it

Go to [Cyber Ghost](https://my.cyberghostvpn.com/en_US/devices/desktop/add-linux) web page and download the installation package for Fedora 31.
Unpack the Zip file. Open the `install.sh` script. Go to the line number 63 and paste the code snippet right after the `elif` condition which is checking if you are running Fedora 31. With this new `elif` condition you will tell the script that it is totally OK to have Fedora 32 and glibc v2.31 :-).

```bash
        elif [ "$distroVersion" == "32" ]; then

                if [ "$glibcVersion" == "2.31" ]; then

                        echo "The glibc version is compatible, continue..."

                else

                        echo "The glibc version is incompatible, exiting setup..."
                        exit

                fi
```

Then run the install script:

```bash
╭─mono@ntb ~
╰─➤  sudo ./install.sh
```

Load the `wireguard` kernel modul and make sure it will be loaded also after the reboot:

```bash
╭─mono@ntb ~
╰─➤   sudo modprobe wireguard

╭─mono@ntb ~
╰─➤  echo "wireguard" | sudo tee /etc/modprobe.d/cyberghost.conf
```

If you updated also the kernel, reboot your machine.  
After that you can test your VPN connection:

```bash
╭─mono@ntb ~
╰─➤  sudo cyberghostvpn --country-code us --city atlanta --connect
Prepare OpenVPN connection ...
Select server ... atlanta-s408-i20
Connecting ...
VPN connection established.

╭─mono@ntb ~  
╰─➤  sudo cyberghostvpn --status
VPN connections found.
```
{{< /note >}}
