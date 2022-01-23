---
title: "Ansible templating"
date: 2022-01-23T01:50:25+02:00
description: Ansible templating
menu:
  sidebar:
    name: Ansible templating
    identifier: ansibletemplating
    parent: tech-sk
    weight: 10
hero: title.png
tags: ["tech", "slovak"]
categories: ["tech"]
---

V tomto blogu popíšem základné postupy pri generovaní (nie len) konfiguračných súborov pomocou Ansible a Jinja2 templatovacieho jazyka. Článok počíta s tým, že máš aspoň základné znalosti Ansible.

### Ako konfigurovať servis

Pri automatizovanej konfiguracii servisov pomocou Ansible som sa stretol so 4 spôsobmi ako deploynuť servis na server ( ajkeď priznávam, že by sa našli aj nejaké iné/šialené spôsoby :-) ).  

1. Jednoducho nainštalovať servis s defaultnou konfiguraciou a teda s konfiguračným súborom servisu nerobiť nič.
2. Nakopírovať na server už hotový konfigurák pre daný servis, použitím modulu `copy`.
3. Editovať konfigurák na vzdialenom serveri napríklad pomocou modulu `lineinfile`.
4. A nakoniec s použi modulu `template`.

### Prečo použivať template

Načo použivať Ansible modul `template` a učiť sa syntax Jinja2, ak môžem použiť už hotový konfigurák? Fungovalo by to iba za predpokladu že je konfiguračný súbor statický a nič sa v ňom meniť nebude. V prípade, že sú v súbore jedna/dve dynamické premenné, dá sa ešte využiť Ansible modul `lineinfile` a miesto v konfiguračnom súbore zmeniť priamo na serveri. No niekedy može byť `lineinfile` trocha tricky. Najmä ak by si mal vytvárať krkolomné `regexp` výrazy. Lenže čo v prípade, ak sa dynamicky menia celé časti konfiguračného súboru a ešte k tomu na základe rôznych podmienok?

Ansible modul `template` je fajn v tom, že ti dovolí meniť hodnoty v konfiguračnom súbore pomocou dynamickych vyrazov a celé bloky pomocou príkazov. Dosadzovať hodnoty, upravovať ich, vytvárať loopy a podmienky na základe ktorých sa rozhoduje či daná čast vo výstupnom súbore bude alebo nie.

### Ako na to

Zákládna iformácia je asi tá, že templatovanie sa deje na Ansible kontrolery. Jinja2 teda nieje potrebná na cieľovom serveri. Tam sa posiela už vygenerovaný súbor.

Príklad ako pracovať s `template` modulom (viac info [tu](https://docs.ansible.com/ansible/latest/modules/template_module.html)). Zdrojový Jinja2 súbor/template má príponu `j2`:

```yaml
- name: Template a file to /etc/service.conf
  template:
    src: templates/myservice.conf.j2
    dest: /etc/myservice.conf
```

Jinja2 pozná 4 druhy oddeľovačov od textu, ktorý je statický a bude vo výslednom súbore (najdôležitejšie sú prvé dva a im sa budem venovať):

```jinja
{% raw %}
{{ ... }} pre výrazy, ktorý bude vo výstupe vyrenderovaný
{% ... %} pre príkazy
{# ... #} pre komenty - nebudú vo výstupe
#  ...    pre celoriadkové príkazy
{% endraw %}
```

Jednotlivé postupy popíšem na príklade. Použijem naň konfigurák pre keepalived. Najskor si predstav, že máme iba cluster s jednym serverom (oxymoron?) v Ansible inventory groupe `ha_cluster`.

Statická konfigurácia pre keepalived master server:

```jinja
{% raw %}
vrrp_instance VI_1 {
        interface eth0
        state MASTER
        priority 100
        virtual_router_id 51
        authentication {
                auth_type PASS
                auth_pass SomePassword
        }
        virtual_ipaddress {
                192.168.1.70
        }
}
{% endraw %}
```

---

### Výrazy

Teraz chcem parametrizovať časť s virtuálnou IP a nahradiť statickú hodnotu `192.168.160.70` premennou tak, aby som vedel ip adresu doplniť z Ansible variable v playbooku, inventory alebo Ansible Facts. Na to použijem Jinja2 výraz s premonnou `virt_ip`. Template bude vyzerať takto:

```jinja
{% raw %}
vrrp_instance VI_1 {
        interface eth0
        state MASTER
        priority 100
        virtual_router_id 51
        authentication {
                auth_type PASS
                auth_pass SomePassword
        }
        virtual_ipaddress {
               {{ virt_ip }}
        }
}
{% endraw %}
```

Ako by vyzeral Ansible playbook s premennou:

```yaml
---
- name: Configure keepalived servers
  hosts: ha_cluster
  become: yes
  vars:
      virt_ip: 192.168.1.70

  tasks:
      - name: Template for Keepalived
        template:
            src: templates/keepalived.conf.j2
            dest: /etc/keepalived.conf
```

Jinja2 obsahuje množstvo filtrov pomocou ktorých sa dáta vo výrazoch daju spracovať. Viac o filtroch nájdeš [tu](https://docs.ansible.com/ansible/latest/user_guide/playbooks_filters.html#filters).

---

### Príkazy

V ďalšom kroku chcem pridať druhý server do clustra. Jeho konfigurácia ale musí byť trocha iná. `state` musí byť `BACKUP` a hodnota `priority` nižšia ako má `MASTER`. Pridám ho do Ansible inventory do grupy `ha_cluster`. Využijem fakt, že tento server je druhý v poradí v inverntory groupe `ha_cluster`. Použijem `if` podmienku v oddeľovači pre príkazy ktorá hovorí, že ak generujem súbor pre prvý server v `ha_cluster` grupe, riadky so `state` a `priority` budú iné ako pre ďalší server. 

```jinja
{% raw %}
vrrp_instance VI_1 {
        interface eth0
{% if groups['ha_cluster'][0] == "host1" %}
        state MASTER
        priority 100
{% else %}
        state BACKUP
        priority 99
{% endif %}
        virtual_router_id 51
        authentication {
                auth_type PASS
                auth_pass SomePassword
        }
        virtual_ipaddress {
               {{ virt_ip }}
        }
}
{% endraw %}
```

---

Lenže čo ak chcem pridať ďalších n serverov a možno ani neviem ich konečný počet? Môžem použit `for` loop. Loop pobeží toľko krát, koľko je serverov v inventory grupe `ha_cluster`. `priority` je odstránená z `if` podmienky a jej hodnota je vypočítaná v `{{ 100 - loop.index0 }}`. Cize každým loopom je hodnota o 1 menšia. To znamená, že MASTER server bude mať prioritu 100 a každy ďalší server o 1 menej.

```jinja
{% raw %}
vrrp_instance VI_1 {
        interface eth0
{% for host in groups['ha_cluster'] %}
{% if groups['ha_cluster'][0] %}
        state MASTER
{% else %}
        state BACKUP
{% endif %}
        priority {{ 100 - loop.index0 }}
{% endfor %}
        virtual_router_id 51
        authentication {
                auth_type PASS
                auth_pass SomePassword
        }
        virtual_ipaddress {
               {{ virt_ip }}
        }
}
{% endraw %}
```

Viac k prpíkazom v Jinja2 [tu](https://jinja.palletsprojects.com/en/2.11.x/templates/#list-of-control-structures).

---

Referencie:

[Kompletná dokumentácia k Jinja2](https://jinja.palletsprojects.com/en/2.11.x/templates)

[Dokumentácia k Ansible Templating](https://docs.ansible.com/ansible/latest/user_guide/playbooks_templating.html)

---
