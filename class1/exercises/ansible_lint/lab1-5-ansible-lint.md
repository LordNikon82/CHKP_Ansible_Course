# LAB 1.5 — ansible-lint

**Goal:** Run `ansible-lint` against the playbook from LAB 1.4, understand
the reported issues, and fix them until the linter passes cleanly.

ansible-lint enforces best practices and style rules. Getting familiar with it
early means fewer surprises in production pipelines and code reviews.

Use the inventory from LAB 1.1 (`~/ansible/lab1-1/hosts`).

---

## Exercise 1 — First run

Instead of using your own playbook from LAB 1.4, use the faulty version below.
Save it as `~/ansible/lab1-5/gaia_routes_lint.yml`:

```yaml
---
- name: inspect Routes
  hosts: gaia
  gather_facts: no

  tasks:
    - check_point.gaia.cp_gaia_routes_facts:
      register: routes_result

    - name: Extract the route_table list
      set_fact:
        route_table: "{{ routes_result.ansible_facts.objects }}"

    - name: Find default route entry (address 0.0.0.0)
      ansible.builtin.set_fact:
        default_route: "{{ item }}"
      loop: "{{ route_table }}"
      when: item.address == "0.0.0.0"

    - name: Display prefix and next_hop for the default route
      debug:
        msg:
          - "Prefix: {{ default_route.address }}/{{ default_route.mask_length }}"
          - "Next Hop: {{ default_route.next_hop.gateways }}"
```

> **Important:** Always run `ansible-lint` from the playbook's directory on
> this jump host. When run from a different directory, ansible-lint falls back
> to a shared `/tmp` cache that can cause permission conflicts between users.

```bash
cd ~/ansible/lab1-3
ansible-lint gaia_routes_lint.yml
```

Read each reported issue carefully. Common categories you will likely see:

| Rule tag | Meaning |
|----------|---------|
| `name[missing]` | Task has no `name:` |
| `name[casing]` | Task name does not start with a capital letter |
| `yaml[truthy]` | `yes`/`no` should be `true`/`false` |
| `fqcn[action-core]` | Should use fully qualified module name (`ansible.builtin.debug` not `debug`) |
| `no-free-form` | Avoid free-form module arguments |

---

## Exercise 2 — Fix all issues

Work through each lint violation and fix it in `gaia_routes_lint.yml`.

Re-run `ansible-lint` after each set of fixes until you reach:

```
Passed: 0 failure(s), 0 warning(s) on 1 files processed.
```

> **Tip:** Use the FQCN for all built-in modules:
> - `debug` → `ansible.builtin.debug`
> - `set_fact` → `ansible.builtin.set_fact`
> - `loop` stays as a task keyword, not a module

---

## Exercise 3 — Verify the playbook still works

Run the corrected playbook to confirm it still produces the same result
as before linting:

```bash
ansible-playbook -i ~/ansible/lab1-1/hosts ~/ansible/lab1-5/gaia_routes_lint.yml
```

---

*Next: LAB 1.6 — Management API: Network Objects*
