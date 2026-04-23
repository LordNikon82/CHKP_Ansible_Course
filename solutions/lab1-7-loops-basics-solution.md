# LAB 1.7 — Loops Basics Solution (Trainer Reference)

---

## Complete playbook — `~/ansible/lab1-7/loops_basics.yml`

```yaml
---
- name: Loops basics
  hosts: localhost
  gather_facts: false

  vars:
    my_firewalls:
      - fw1
      - fw2
      - fw3
      - fw4

    firewall_dict:
      name: fw1
      location: Munich
      ip_address: 1.1.1.1
      sw_version: R82

    fw_list:
      - { name: fw1, location: Munich,  ip_address: 1.1.1.1,  sw_version: R82    }
      - { name: fw2, location: Munich,  ip_address: 1.1.1.2,  sw_version: R82    }
      - { name: fw3, location: Cologne, ip_address: 10.1.1.1, sw_version: R81.10 }
      - { name: fw4, location: Cologne, ip_address: 10.1.1.2, sw_version: R81.10 }

  tasks:
    # Exercise 1 — inline list
    - name: Simple loop
      ansible.builtin.debug:
        msg: "Firewall: {{ item }}"
      loop:
        - fw1
        - fw2
        - fw3
        - fw4

    # Exercise 2 — loop over variable
    - name: Loop over my_firewalls variable
      ansible.builtin.debug:
        msg: "Firewall: {{ item }}"
      loop: "{{ my_firewalls }}"

    # Exercise 3 — dict2items
    - name: Loop over firewall_dict
      ansible.builtin.debug:
        msg: "{{ item.key }}: {{ item.value }}"
      loop: "{{ firewall_dict | dict2items }}"

    # Exercise 4 — list of dicts with loop_control
    - name: Loop over fw_list
      ansible.builtin.debug:
        msg: "{{ item.name }} | {{ item.location }} | {{ item.ip_address }} | {{ item.sw_version }}"
      loop: "{{ fw_list }}"
      loop_control:
        label: " "
```

---

## Key teaching points

| Concept | Explanation |
|---------|-------------|
| `loop:` with inline list | Quickest form; `item` holds the current value |
| `loop: "{{ var }}"` | Loops over a list variable — identical behaviour |
| `dict2items` filter | Converts a dict to a list of `{key, value}` pairs for looping |
| `loop_control.label` | Replaces the item representation in task header — critical for readability with large dicts |
| Exercise 1 vs 2 output | Functionally identical — teaches that inline loop and variable loop are interchangeable |

---

## Answer to Exercise 2 question

The output is identical — Ansible makes no distinction between an inline
list and a list variable. The `loop:` keyword always expects a list; the
variable is just a reference to one.
