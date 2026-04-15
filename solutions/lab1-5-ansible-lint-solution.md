# LAB 1.5 — ansible-lint Solution (Trainer Reference)

---

## Faulty playbook (`gaia_routes_lint.yml`) with lint issues annotated

```yaml
---
- name: inspect Routes          # ← name[casing]: must start with capital letter
  hosts: gaia
  gather_facts: no              # ← yaml[truthy]: use true/false, not yes/no

  tasks:
    - check_point.gaia.cp_gaia_routes_facts:   # ← name[missing]: no task name
      register: routes_result

    - name: Extract the route_table list
      set_fact:                 # ← fqcn[action-core]: use ansible.builtin.set_fact
        route_table: "{{ routes_result.ansible_facts.objects }}"

    - name: Find default route entry (address 0.0.0.0)
      ansible.builtin.set_fact:
        default_route: "{{ item }}"
      loop: "{{ route_table }}"
      when: item.address == "0.0.0.0"

    - name: Display prefix and next_hop for the default route
      debug:                    # ← fqcn[action-core]: use ansible.builtin.debug
        msg:
          - "Prefix: {{ default_route.address }}/{{ default_route.mask_length }}"
          - "Next Hop: {{ default_route.next_hop.gateways }}"
```

---

## Lint-clean version

```yaml
---
- name: Inspect Routes                        # name[casing]: capital letter
  hosts: gaia
  gather_facts: false                         # yaml[truthy]: not "no"

  tasks:
    - name: Retrieve routing table from Gaia  # name[missing]: task now has a name
      check_point.gaia.cp_gaia_routes_facts:
      register: routes_result

    - name: Extract the route_table list
      ansible.builtin.set_fact:              # fqcn[action-core]: fully qualified
        route_table: "{{ routes_result.ansible_facts.objects }}"

    - name: Find default route entry (address 0.0.0.0)
      ansible.builtin.set_fact:
        default_route: "{{ item }}"
      loop: "{{ route_table }}"
      when: item.address == "0.0.0.0"

    - name: Display prefix and next_hop for the default route
      ansible.builtin.debug:                 # fqcn[action-core]: fully qualified
        msg:
          - "Prefix: {{ default_route.address }}/{{ default_route.mask_length }}"
          - "Next Hop: {{ default_route.next_hop.gateways }}"
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
