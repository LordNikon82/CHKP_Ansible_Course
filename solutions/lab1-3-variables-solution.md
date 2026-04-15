# LAB 1.3 — Variables Solution (Trainer Reference)

---

## Directory layout

```
~/ansible/lab1-2/
  variables.yml
  my_vars.yml
  group_vars/
    all.yml
```

---

## group_vars/all.yml

```yaml
cp_external_interface: "eth1"
```

---

## my_vars.yml

```yaml
cp_external_interface: "eth3"
```

---

## variables.yml (complete — all exercises)

```yaml
---
- name: Variables demo
  hosts: mgmt
  gather_facts: false
  vars_files: my_vars.yml          # 1d: loads my_vars.yml

  tasks:
    - name: 1a — Print ansible_facts (empty without gather_facts)
      debug:
        var: ansible_facts

    - name: 1b — Print network_os
      debug:
        var: ansible_network_os

    - name: 1b — Print ansible_host
      debug:
        var: ansible_host

    - name: 1b — Print inventory_hostname
      debug:
        var: inventory_hostname

    - name: 1c/1d — Print cp_external_interface
      debug:
        var: cp_external_interface

    - name: 1e — Build FQDN with set_fact
      set_fact:
        gateway_fqdn: "{{ inventory_hostname }}.cp.lab.internal"

    - name: 1e — Print gateway_fqdn
      debug:
        var: gateway_fqdn
```

---

## Answers to questions

**1a — Why is ansible_facts empty?**
`gather_facts: false` skips the `setup` module. Even with it enabled,
CheckPoint HTTPAPI connections don't populate `ansible_facts` the way Linux
hosts do — the module is not supported over HTTPAPI.

**1d — Which value wins?**
`vars_files` wins over `group_vars/all.yml` because `vars_files` has higher
precedence in Ansible's variable hierarchy. The output will show `"eth3"`.

Precedence order (low → high, simplified):
```
group_vars/all  <  vars_files  <  set_fact  <  extra vars (-e)
```

**1e — Expected output:**
```
ok: [lab_mgmt] => {
    "gateway_fqdn": "lab_mgmt.cp.lab.internal"
}
```

---

## Common mistakes to watch for

| Mistake | Symptom |
|---------|---------|
| `vars_files` as a task instead of a play keyword | Syntax error |
| Forgetting `gather_facts: false` on mgmt play | Long timeout, then error |
| `var: "{{ my_var }}"` in debug | Prints literally `{{ my_var }}` — use `var: my_var` (no braces) |
