# LAB 1.3b — ansible-lint Solution (Trainer Reference)

> **Do not distribute to students.**

---

## Lint-clean version of gaia_routes.yml

The most common issues ansible-lint flags in beginner playbooks:

```yaml
---
- name: Inspect routes                        # name[casing]: capital letter
  hosts: gaia
  gather_facts: false                         # yaml[truthy]: not "no" or "False"

  tasks:
    - name: Retrieve routing table from Gaia  # name[missing]: every task needs a name
      check_point.gaia.cp_gaia_routes_facts:  # fqcn: already correct (collection module)
      register: routes_result

    - name: Extract the route_table list
      ansible.builtin.set_fact:              # fqcn[action-core]: not just "set_fact"
        route_table: "{{ routes_result.ansible_facts.cp_gaia_routes.objects }}"

    - name: Find default route entry
      ansible.builtin.set_fact:
        default_route: "{{ item }}"
      loop: "{{ route_table }}"
      when: item.destination == "0.0.0.0"

    - name: Display prefix and next_hop for the default route
      ansible.builtin.debug:                 # fqcn[action-core]: not just "debug"
        msg:
          - "Prefix: {{ default_route.destination }}"
          - "Next Hop: {{ default_route.gateways }}"
```

---

## Typical lint output before fixes

```
WARNING  Listing 4 violation(s) that are fatal
yaml[truthy] ... gather_facts: no  →  use true/false
name[missing] ... task has no name
fqcn[action-core] ... use ansible.builtin.debug
fqcn[action-core] ... use ansible.builtin.set_fact
```

## After fixes

```
Passed: 0 failure(s), 0 warning(s) on 1 files processed.
Last profile that met the validation criteria was 'production'.
```

---

## Key teaching points

| Rule | Why it matters |
|------|---------------|
| FQCN for built-in modules | Future-proofs playbooks; unambiguous when multiple collections define same name |
| `true`/`false` over `yes`/`no` | YAML 1.2 spec; avoids confusion across tools |
| Task names | Essential for readable output and debugging in CI/CD pipelines |
| Play names | Same — always name your plays |
