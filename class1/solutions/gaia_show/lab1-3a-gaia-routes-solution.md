# LAB 1.3a — Gaia Routes Solution (Trainer Reference)

> **Do not distribute to students.**

---

## Inventory additions (`~/ansible/lab1-1/hosts`)

```ini
[gaia]
lab_gaia ansible_host=<CP_PUBLIC_IP>

[gaia:vars]
ansible_network_os=check_point.gaia.checkpoint
ansible_connection=network_cli
ansible_user=admin
ansible_password=<PASSWORD>
```

---

## Complete playbook — `~/ansible/lab1-3/gaia_routes.yml`

```yaml
---
- name: Inspect Routes
  hosts: gaia
  gather_facts: false

  tasks:
    - name: Retrieve routing table from Gaia
      check_point.gaia.cp_gaia_routes_facts:
      register: routes_result

    - name: Extract the route_table list
      ansible.builtin.set_fact:
        route_table: "{{ routes_result.ansible_facts.cp_gaia_routes.objects }}"

    - name: Find default route entry (destination 0.0.0.0)
      ansible.builtin.set_fact:
        default_route: "{{ item }}"
      loop: "{{ route_table }}"
      when: item.destination == "0.0.0.0"

    - name: Display prefix and next_hop for the default route
      ansible.builtin.debug:
        msg:
          - "Prefix: {{ default_route.destination }}"
          - "Next Hop: {{ default_route.gateways }}"
```

---

## Expected output

```
TASK [Retrieve routing table from Gaia] ************************
ok: [lab_gaia]

TASK [Extract the route_table list] ****************************
ok: [lab_gaia]

TASK [Find default route entry (destination 0.0.0.0)] **********
skipping: [lab_gaia] => (item=...) [non-default routes skipped]
ok: [lab_gaia] => (item={'destination': '0.0.0.0', 'gateways': [{'address': '...', 'interface': 'eth0'}], ...})

TASK [Display prefix and next_hop for the default route] *******
ok: [lab_gaia] => {
    "msg": [
        "Prefix: 0.0.0.0",
        "Next Hop: [{'address': '10.x.1.1', 'interface': 'eth0'}]"
    ]
}
```

---

## Key teaching points

| Point | Explanation |
|-------|-------------|
| `network_cli` vs `httpapi` | Gaia uses SSH (`network_cli`); Management API uses HTTPAPI — same VM, two different access paths |
| `register` structure | Always print the full registered result first to discover the key path before using `set_fact` |
| `loop` + `when` | Iterates every route, `set_fact` only fires (overwrites) when the condition is true — last match wins |
| `ansible_facts` key | The `cp_gaia_routes_facts` module returns data under `ansible_facts`, not directly in the result |

---

## Common mistakes

| Mistake | Symptom |
|---------|---------|
| Wrong `ansible_connection` (using `httpapi` for gaia) | Connection refused or wrong protocol error |
| Accessing `routes_result.objects` directly | `undefined variable` — data is nested under `ansible_facts` |
| `when: item.destination == "0.0.0.0/0"` | No match — the field is `"0.0.0.0"`, prefix length is separate |
