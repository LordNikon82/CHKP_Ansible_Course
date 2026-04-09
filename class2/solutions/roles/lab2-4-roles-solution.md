# LAB 2.4 — Roles Solution (Trainer Reference)

> **Do not distribute to students.**

---

## Directory structure

```
~/ansible/lab2-3/
├── site.yml
└── roles/
    └── address_range/
        ├── tasks/
        │   ├── main.yml
        │   ├── cfg_address_ranges.yml
        │   └── display_address_ranges.yml
        ├── vars/
        │   └── main.yml
        └── handlers/
            └── main.yml
```

---

## `roles/address_range/vars/main.yml`

```yaml
---
address_ranges:
  - name: "Test Address Range 1"
    ip_address_first: "192.168.201.1"
    ip_address_last: "192.168.201.10"
  - name: "Test Address Range 2"
    ip_address_first: "192.168.202.1"
    ip_address_last: "192.168.202.10"
  - name: "Test Address Range 3"
    ip_address_first: "192.168.203.1"
    ip_address_last: "192.168.203.10"
```

---

## `roles/address_range/tasks/main.yml`

```yaml
---
- name: Configure Address Ranges
  ansible.builtin.import_tasks: cfg_address_ranges.yml

- name: Display Address Ranges
  ansible.builtin.import_tasks: display_address_ranges.yml
```

---

## `roles/address_range/tasks/cfg_address_ranges.yml`

```yaml
---
- name: Create address range objects
  check_point.mgmt.cp_mgmt_address_range:
    name: "{{ item.name }}"
    ip_address_first: "{{ item.ip_address_first }}"
    ip_address_last: "{{ item.ip_address_last }}"
    color: dark green
    state: present
  loop: "{{ address_ranges }}"
  notify: publish
```

---

## `roles/address_range/tasks/display_address_ranges.yml`

```yaml
---
- name: Get address range objects
  check_point.mgmt.cp_mgmt_address_range_facts:
  register: address_range_out

- name: Print address range objects
  ansible.builtin.debug:
    var: address_range_out
```

---

## `roles/address_range/handlers/main.yml`

```yaml
---
- name: publish
  check_point.mgmt.cp_mgmt_publish:
```

---

## `site.yml`

```yaml
---
- name: Configure CheckPoint address ranges
  hosts: lab_mgmt
  gather_facts: false

  roles:
    - address_range
```

---

## Key teaching points

| Concept | Explanation |
|---------|-------------|
| Role directory structure | `tasks/`, `vars/`, `handlers/` are auto-loaded by Ansible when the role is called — no explicit `include` needed |
| `vars/main.yml` vs `defaults/main.yml` | `vars/` has higher precedence than `defaults/` — use `vars/` for values that should not be overridden easily |
| `import_tasks` in `tasks/main.yml` | Static inclusion — good for splitting a large task list into logical files; tags and conditionals on the import apply to all included tasks |
| Handler scoping | Handlers defined in a role are scoped to that role — they fire after all tasks in the play complete, not immediately when notified |
| Idempotency | `state: present` means running the playbook twice produces the same result; the publish handler only fires when a change actually occurs |
