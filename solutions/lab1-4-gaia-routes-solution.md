# LAB 1.4 — Gaia Routes Solution (Trainer Reference)

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

    - name: Print full routes result
      ansible.builtin.debug:
        var: routes_result

    - name: Extract the route_table list
      ansible.builtin.set_fact:
        route_table: "{{ routes_result.ansible_facts.objects }}"

    - name: Find default route entry (address 0.0.0.0)
      ansible.builtin.set_fact:
        default_route: "{{ item }}"
      loop: "{{ route_table }}"
      when: item.address == "0.0.0.0"

    - name: Display prefix and next_hop for the default route
      ansible.builtin.debug:
        msg:
          - "Prefix: {{ default_route.address }}/{{ default_route.mask_length }}"
          - "Next Hop: {{ default_route.next_hop.gateways }}"
```

---

## Expected output

```
TASK [Retrieve routing table from Gaia] ************************
ok: [lab_gaia]

TASK [Extract the route_table list] ****************************
ok: [lab_gaia]

TASK [Find default route entry (address 0.0.0.0)] **************
skipping: [lab_gaia] => (item=...) [non-default routes skipped]
ok: [lab_gaia] => (item={'address': '0.0.0.0', 'mask_length': 0, 'next_hop': {'gateways': [...]}, ...})

TASK [Display prefix and next_hop for the default route] *******
ok: [lab_gaia] => {
    "msg": [
        "Prefix: 0.0.0.0/0",
        "Next Hop: [{'address': '10.x.1.1', 'interface': 'eth0'}]"
    ]
}
```

> **Data structure note (collection v7.0.0):** The httpapi-based module returns routes under
> `ansible_facts.objects` (not `ansible_facts.cp_gaia_routes.objects`). Each route uses
> `address`/`mask_length` (not `destination`) and `next_hop.gateways` (not `gateways`).

---

## Key teaching points

| Point | Explanation |
|-------|-------------|
| Gaia API vs Management API | Both use HTTPAPI but hit different endpoints: Gaia REST (`/gaia_api/`) and Management API (`/web_api/`) — same VM, two separate services |
| `register` structure | Always print the full registered result first to discover the key path before using `set_fact` |
| `loop` + `when` | Iterates every route, `set_fact` only fires (overwrites) when the condition is true — last match wins |
| `ansible_facts` key | The `cp_gaia_routes_facts` module returns data under `ansible_facts`, not directly in the result |

---

## Common mistakes

| Mistake | Symptom |
|---------|---------|
| Using `ansible_connection=network_cli` for gaia | `network os check_point.gaia.checkpoint is not supported` — collection v3+ requires httpapi |
| Accessing `routes_result.objects` directly | `undefined variable` — data is nested under `ansible_facts` |
| `when: item.destination == "0.0.0.0/0"` | No match — the field is `"0.0.0.0"`, prefix length is separate |
