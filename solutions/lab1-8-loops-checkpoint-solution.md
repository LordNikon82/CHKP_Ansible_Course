# LAB 1.8 — Loops with CheckPoint Solution (Trainer Reference)

---

## Finding the gateway object name

```bash
ansible lab_mgmt -i ~/ansible/lab1-1/hosts \
  -m check_point.mgmt.cp_mgmt_show_gateways_and_servers
```

On a fresh standalone deployment the gateway object name is typically the
hostname set during first-time wizard — often something like `gw-<suffix>`.
Tell students to look for the `name` field in the returned list.

---

## Complete playbook — `~/ansible/lab1-8/loops_checkpoint.yml`

```yaml
---
- name: Gateway interface inspection
  hosts: mgmt
  gather_facts: false

  vars:
    fw_name: "gw-<your-gateway-name>"   # students replace with actual name

  tasks:
    - name: Retrieve simple gateway facts
      check_point.mgmt.cp_mgmt_simple_gateway_facts:
        name: "{{ fw_name }}"
      register: gw_result

    - name: Extract interface list
      ansible.builtin.set_fact:
        gw_interfaces: "{{ gw_result.ansible_facts.simple_gateway.interfaces }}"

    - name: Print eth0 details only
      ansible.builtin.debug:
        msg: "Interface: {{ item.name }} | IP: {{ item['ipv4-address'] }} | Mask: {{ item['ipv4-network-mask'] }}"
      loop: "{{ gw_interfaces }}"
      when: item.name == "eth0"
      loop_control:
        label: "{{ item.name }}"
```

---

## Answer to Exercise 1 question

The interface list lives at:
```
gw_result.ansible_facts.simple_gateway.interfaces
```

Always `register` + `debug` the full result first before navigating the
structure — module return schemas differ and the path isn't always obvious.

---

## Bonus solution

```yaml
    # Print all interfaces
    - name: Print all interfaces
      ansible.builtin.debug:
        msg: "{{ item.name }} | {{ item['ipv4-address'] }} | {{ item['ipv4-network-mask'] }}"
      loop: "{{ gw_interfaces }}"
      loop_control:
        label: "{{ item.name }}"

    # Skip interfaces with no IP
    - name: Print only interfaces with an IPv4 address
      ansible.builtin.debug:
        msg: "{{ item.name }} | {{ item['ipv4-address'] }}"
      loop: "{{ gw_interfaces }}"
      when: item['ipv4-address'] | length > 0
      loop_control:
        label: "{{ item.name }}"
```

---

## Key teaching points

| Concept | Explanation |
|---------|-------------|
| `when:` inside a loop | Evaluated per iteration — non-matching items show as `skipping` |
| Bracket notation `item['ipv4-address']` | Required for hyphenated keys — dot notation fails |
| `loop_control.label` | Set to `item.name` here — much cleaner than dumping the full dict per line |
| `register` path discovery | Reinforce the pattern: register → debug full result → navigate the path |

---

## Common mistakes

| Mistake | Symptom |
|---------|---------|
| `item.ipv4-address` (dot notation) | Jinja2 parse error — hyphen is interpreted as minus |
| Missing `gather_facts: false` | Timeout against HTTPAPI host |
| Wrong gateway name in `fw_name` | Module returns error: object not found |
