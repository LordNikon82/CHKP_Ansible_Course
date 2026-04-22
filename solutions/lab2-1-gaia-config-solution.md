# LAB 2.1 — Gaia Configuration Solution (Trainer Reference)

---

## Exercise 1 — `static_route.yml`

```yaml
---
- name: Configure static route on Gaia
  hosts: gaia
  gather_facts: false

  tasks:
    - name: Configure static route 172.31.128.0/21
      check_point.gaia.cp_gaia_static_route:
        destination: "172.31.128.0/21"
        next_hop:
          - address: "172.31.128.1"
            priority: 1
        interface: "eth1"
        state: present
      register: route_result

    - name: Print result
      ansible.builtin.debug:
        var: route_result
```

---

## Exercise 2 — `dns.yml`

```yaml
---
- name: Configure DNS on Gaia
  hosts: gaia
  gather_facts: false

  tasks:
    - name: Configure DNS servers
      check_point.gaia.cp_gaia_dns:
        primary: "172.31.0.2"
        secondary: "8.8.8.8"
        tertiary: "8.8.4.4"
        state: present
      register: dns_result

    - name: Print result
      ansible.builtin.debug:
        var: dns_result
```

---

## Exercise 3 — `verify_routes.yml`

```yaml
---
- name: Verify static route on Gaia
  hosts: gaia
  gather_facts: false

  tasks:
    - name: Retrieve routing table
      check_point.gaia.cp_gaia_routes_facts:
      register: routes_result

    - name: Extract route list
      ansible.builtin.set_fact:
        route_table: "{{ routes_result.ansible_facts.objects }}"

    - name: Find configured static route
      ansible.builtin.set_fact:
        static_route: "{{ item }}"
      loop: "{{ route_table }}"
      when: item.address == "172.31.128.0"

    - name: Assert route is present with correct next hop
      ansible.builtin.assert:
        that:
          - static_route is defined
          - static_route.next_hop.gateways[0].address == "172.31.128.1"
        success_msg: "Static route verified successfully"
        fail_msg: "Static route not found or next hop incorrect"

    - name: Print route summary
      ansible.builtin.debug:
        msg: "Route {{ static_route.address }}/{{ static_route.mask_length }} → next hop: {{ static_route.next_hop.gateways[0].address }} via {{ static_route.next_hop.gateways[0].interface }}"
```

---

## Exercise 4 — `gaia_config.yml`

```yaml
---
- name: Configure and verify Gaia OS settings
  hosts: gaia
  gather_facts: false

  tasks:
    - name: Configure static route 172.31.128.0/21
      check_point.gaia.cp_gaia_static_route:
        destination: "172.31.128.0/21"
        next_hop:
          - address: "172.31.128.1"
            priority: 1
        interface: "eth1"
        state: present

    - name: Configure DNS servers
      check_point.gaia.cp_gaia_dns:
        primary: "172.31.0.2"
        secondary: "8.8.8.8"
        tertiary: "8.8.4.4"
        state: present

    - name: Retrieve routing table
      check_point.gaia.cp_gaia_routes_facts:
      register: routes_result

    - name: Extract route list
      ansible.builtin.set_fact:
        route_table: "{{ routes_result.ansible_facts.objects }}"

    - name: Find configured static route
      ansible.builtin.set_fact:
        static_route: "{{ item }}"
      loop: "{{ route_table }}"
      when: item.address == "172.31.128.0"

    - name: Assert static route is present
      ansible.builtin.assert:
        that:
          - static_route is defined
          - static_route.next_hop.gateways[0].address == "172.31.128.1"
        success_msg: "Static route 172.31.128.0/21 verified"
        fail_msg: "Static route not found or next hop incorrect"

    - name: Print route summary
      ansible.builtin.debug:
        msg: "Route {{ static_route.address }}/{{ static_route.mask_length }} → next hop: {{ static_route.next_hop.gateways[0].address }} via {{ static_route.next_hop.gateways[0].interface }}"
```

---

## Key teaching points

| Point | Explanation |
|-------|-------------|
| `check_point.gaia` vs `check_point.mgmt` | Both use HTTPAPI on port 443 but hit different endpoints: Gaia REST (`/gaia_api/`) for OS config, Management API (`/web_api/`) for policy objects |
| `state: present` idempotency | Configuration modules apply the setting only if it differs — second run reports `changed=0` |
| Route field names | Routes use `address` + `mask_length` (not `destination`); next hop is under `next_hop.gateways` |
| `assert` for verification | Use `ansible.builtin.assert` to fail the play explicitly when expected state is not present — better than silent success |

---

## Common mistakes

| Mistake | Symptom |
|---------|---------|
| `when: item.destination == "172.31.128.0/21"` | No match — field is `address` (without mask), mask is separate `mask_length` |
| Accessing `routes_result.objects` directly | `undefined variable` — data is nested under `ansible_facts.objects` |
| Missing `priority` in `next_hop` list | Module may reject or silently ignore the next hop entry |
| Running against `mgmt` group instead of `gaia` | Module connects but hits `/web_api/` instead of `/gaia_api/` — wrong collection for the endpoint |
