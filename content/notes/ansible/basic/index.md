---
title: Workarounds
weight: 210
menu:
  notes:
    name: Workarounds
    identifier: notes-ansible-workarounds
    parent: notes-ansible
    weight: 10
---

<!-- Variable -->
{{< note title="Serial tasks" >}}

Ansible does not a have out-of-the-box feature which could make possible to run block of tasks in a serial way (the same as you can do with tasks in a playbooks).
To owercome this disadvantage you can include the file with serial tasks like this:

```yaml

- name: Run tasks per node in serial mode
  ansible.builtin.include_tasks: serial_pipe.yml
  with_items: "{{ groups.all }}"
  loop_control:
    loop_var: _host_item
  when: hostvars[_host_item].inventory_hostname == inventory_hostname

```

{{< /note >}}
