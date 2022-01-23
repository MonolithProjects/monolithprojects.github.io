---
title: "Docker Part 1."
date: 2022-01-23T01:50:25+02:00
description: Ako Docker vytvara kontajnery
menu:
  sidebar:
    name: Docker part 1
    identifier: docker1
    parent: tech-sk
    weight: 10
hero: title.png
tags: ["tech", "slovak"]
categories: ["tech"]
---

V tomto seriali by som chcel popísať, ako Docker vytvára kontainery.

### Čo je to kontajner

Dnes existujú rôzne technológie, ktoré sa zaoberajú vytváraním kontajnerov. Dá sa dokonca povedať, že je okolo nich celkom pekný hype. Určite si už videl nejaké cool prezentácie o niečom čo sa volá Docker kde ti bolo jednoducho povedané, že Docker využíva namespaces, cgroups, chroot, atď. na vytváranie kontajnerov. Ale načo je toto všetko potrebné na vytvorenie kontajnera?
Prečo to nieje jednoducho systémove volanie a hotovo? Pretože realita je taká, že kontajnery neexistujú - sú vymyslené. Nič také ako "linux container" v kerneli nieje. Kontajner patri do oblasti user space a teda mimo kernelu.  

### Namespaces

Na začiatok si povieme niečo o Linux namespace aby bolo jasné ako sú použité v rámci Dockeru. A neskor sa pozrieme na to, ako sú namespaces kombinované so cgroups a izolovanými filsystémami na vytvorenie niečoho užitočného.  

Najskor by by som mal povedať podstatu namespaces a načo sú užitočné. Namespace je funkcia Linux kernelu rozparticiovať rôzne resourcy do separatneho "priestoru", kde skupina resourcov (napríklad procesov) v jednom priestore/namespace vidi iba resourcy patriace do toho istého namespacu. Linux kernel pozná niekoľko typov namespacov. Kukneme sa na ne.

### NET Namespace

Sieťový namespace poskytuje vlastný pohľad na network stack tvojho sytému. Do toho môže byť zahrnuý aj `localhost`. V tomto príklade je pohlad z vnútra docker kontajnera bežiaceho s `--net host` a teda schopného vidieť všetky sieťové rozhrania hostu.

```bash
root@virt01:/containers# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:00:00:00:00:00 brd ff:ff:ff:ff:ff:ff
    inet 192.168.160.101/24 brd 192.168.160.255 scope global enp1s0
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fefd:43c0/64 scope link
       valid_lft forever preferred_lft forever
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ab:3d:e1:a2 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:abff:fe3d:e1a2/64 scope link
       valid_lft forever preferred_lft forever
```

Namespacy sú primárne ovládané pomocou clone flags. Pri systemovom volaní "clone" na vytvorenie nového sieťového namespace pre subproces je použitá variabla `CLONE_NEWNET` a tento namespace je neskôr spárovaný s `veth` takže kontajner dostane vlastnú IP alokovanú na bridge, väčšinou `docker0`.

### MNT Namespace

Mount namespace ti dáva špecifikovaný pohľad na mounty na tvojom filesysteme. Ľudia si často tento namespace zamieňajú s jailingom procesu vnútri `chroot` a podobne. Lenže to nieje pravda. Mount namespace nieje to isté ako filesystem jail. `chroot` je fajn ale ten neposkytuje kompletnú izoláciu a jeho efekt je obmedzený iba na root mountpoint. Samostatný mount namespace docoľuje aby každý z izolovaných procesov mal vlastný pohľad na pôvodny mountpoint sysému.

Na vytvorenie nového mount namespace sa používa variable `CLONE_NEWNS`. Wait... čože? Nie CLONE_NEWMOUNT alebo CLONE_NEWMNT? Nie, pretože mount namespace bol úplne prvý namespace a tato variabla mu jednoducho ostala.  

### UTS Namespace

Ďalší namespace je UTS namespace. Ten je zodpovedný za identifikáciu systému. To zahŕňa `hostname` a `domainname`. Jednoducho umožňuje kontajneru mať svoj vlastný hostname, nezávisle na hostovi a ostatných kontajneroch. Tu je ako flag použitý `CLONE_NEWUTS`. Štandardne je jeho hodnota zdedená z hostu.

### IPC Namespace

IPC namespace slúži na izolovanie komunikácie medzi procesmi, ako SysV alebo message queues. Tu sa používa flag `CLONE_NEWIPC`

### PID Namespace

Tento namespace zabezpečuje s ktorými proces ID možes komunikovať a vidieť ich. Keď vytvoríš novy PID namespace, prvý proces bude PID 1. Ak tento proces existuje, kernel killne všetky ostatné v rámci tohto namespace.  

Clone flag pre tento namespace je `CLONE_NEWPID`. Ak sa teraz pozriem na jednoduchý kontajner s `CMD  ["bash"]` uvidím toto:

```bash
root@virt01:/containers# ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.1  20236  3108 pts/0    Ss   15:11   0:00 bash
root        11  0.0  0.0  17492  2060 pts/0    R+   16:19   0:00 ps aux
```

### USER Namespace

Aby sa zabezpečila prevencia proti `privilege-escalation` útokom, je vhodné aplikáciu v kontajnery nakonfigurovať tak, aby používala neprivilegovaného usera. Ak to nieje možné a proces v kontajneri musí bežať ako `root` použi user namespace. Tento namespace zabezpečí, že môžeš mať užívateľov v rámci jedneho namespace ktorý niesú tí istí užívatelia mimo tohto namespace. To znamená, že vďaka user namespace, môže byť user `root` vramci kontajnera ale zároveň ako "neroot" na hoste. Toto je docielené vďaka UID a GID mapingu.  

Clone flag je použitý `CLONE_NEWUSER`.

### A na záver

Takže pre rekapituláciu prešiel som cez Network, Mount, IPC, UTS, PID a User namespaces. Niekedy nabudúce sa pozriem ako sa jailne proces kontajnera na root filesysteme (docker image) a na používanie cgroups.

---
