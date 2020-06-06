---
layout: post
title:  "Ansible Templating s použitím Jinja2"
categories: [slovak]
---

V tomto blogu popíšem možnosti generovania (nie len) konfiguračných súborov Linux/Unix servisov pomocou Ansible a Jinja2 templatovacieho jazyka. Článok počíta s tým, že čitatel má aspoň základné znalosti Ansible.

### Akoy spôsobom konfigurovať servis

Pri automatizovanej konfiguracii servisov pomocou Ansible som sa stretol so 4 spôsobmi ako deploynuť servis na server ( ajkeď priznávam, že by sa našli aj nejaké iné/šialené spôsoby :-) ).  

1. Jednoducho nainštalovať servis s defaultnou konfiguraciou a teda s konfiguračným súborom servisu nerobiť nič.
2. Nakopírovať na server už hotový konfigurák pre daný servis, použitím modulu `copy`.
3. Editovať konfigurák na vzdialenom serveri napríklad pomocou modulu `lineinfile`.
4. A nakoniec s použi modulu `template`.

### Prečo použivať template

Ak si niekto povie načo použivať modul `template` a učiť sa syntax Jinja2, ak môžem použiť už existujúci konfigurák, toto by fungovalo iba za predpokladu že je konfiguračný súbor statický a nič sa v ňom meniť nebude. V prípade, že je v súbore jedno/dve dynamické premenné, je v poriadku keď použiješ modul `lineinfile` a miesto v konfiguračnom súbore zmeníš priamo na serveri. Ale niekedy aj v tomto prípade može byť `lineinfile` trocha tricky. Najmä ak by si mal vytvárať nejaké krkolomné `regexp` výrazy. Lenže čo v prípade, ak sa dynamicky menia celé časti konfiguračného súboru?

Ansible modul `template` je fajn v tom, že ti dovolí meniť hodnoty v konfiguračnom súbore pomocou premenných a rôznych dynamickych vyrazov. Dosadzovať hodnoty, vytvárať loopy a podmienky na základe ktorých sa rozhoduje či daná čast vo výstupnom súbore bude alebo nie.

### Ako na to

Zákládna ńformácia je asi tá, že templatovanie sa deje na Ansible kontrolery. Jinja2 teda nieje potrebná na cieľoovom serveri. Tam sa posiela už vygenerovaný súbor.

Príklad ako pracovať s `template` modulom. Zdrojový Jinja2 súbor/template má príponu `j2`:

```yaml
- name: Template a file to /etc/service.conf
  template:
    src: templates/myservice.conf.j2
    dest: /etc/myservice.conf
```

Jinja2 pozná 4 druhy oddeľovačov (najdôležitejšie sú prvé dva a im sa budem venovať):

- `{{ ... }}` pre výrazy, ktorý bude vo výstupe vyrenderovaný
- `{% ... %}` pre príkazy
- `{# ... #}` pre komenty (nebudú vo výstupe)
- `#  ... ##` pre celoriadkové príkazy

