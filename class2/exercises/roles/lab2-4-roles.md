# LAB 2.4 — Roles

**Goal:** Understand how Ansible roles organise tasks, variables, and handlers
into a reusable, self-contained unit. You will build a role that creates
CheckPoint address range objects on the management server.

Use the inventory from LAB 1.1 (`~/ansible/lab1-1/hosts`).

---

## Background

A role is a standardised directory structure that groups related tasks,
variables, handlers, and files together. Instead of putting everything into
one large playbook, you break it into a role and call that role from a short
top-level playbook.

Minimum structure for this lab:

```
roles/
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

## Exercise 1 — Create the role structure

Create the directory tree above under `~/ansible/lab2-3/roles/`.

You can do this manually with `mkdir -p`, or use:

```bash
ansible-galaxy init ~/ansible/lab2-3/roles/address_range
```

(`ansible-galaxy init` creates the full skeleton — you only need the
directories listed above for this lab.)

---

## Exercise 2 — Define the variables

In `roles/address_range/vars/main.yml`, define a list variable
`address_ranges` containing three address range objects:

| Name                 | First IP        | Last IP          |
|----------------------|-----------------|------------------|
| Test Address Range 1 | 192.168.201.1   | 192.168.201.10   |
| Test Address Range 2 | 192.168.202.1   | 192.168.202.10   |
| Test Address Range 3 | 192.168.203.1   | 192.168.203.10   |

Each list item should have the keys `name`, `ip_address_first`, and
`ip_address_last`.

---

## Exercise 3 — Write the tasks

**`tasks/main.yml`** should contain exactly two tasks that delegate to the
other task files using `import_tasks`:

```yaml
- name: Configure Address Ranges
  ansible.builtin.import_tasks: cfg_address_ranges.yml
- name: Display Address Ranges
  ansible.builtin.import_tasks: display_address_ranges.yml
```

**`tasks/cfg_address_ranges.yml`** — loop over `address_ranges` and create
each one using `check_point.mgmt.cp_mgmt_address_range` with:

- `color: dark green`
- `state: present`
- `ip_address_first` and `ip_address_last` from the variable
- Notify the publish handler if the task results in a change

The module signature for reference:

```yaml
- name: Add address ranges
  check_point.mgmt.cp_mgmt_address_range:
    color: dark green
    ip_address_first: 10.1.1.1
    ip_address_last: 10.1.1.10
    name: My Address Range
    state: present
```

**`tasks/display_address_ranges.yml`** — retrieve all configured address
ranges using `check_point.mgmt.cp_mgmt_address_range_facts`, register the
output, and print it with `ansible.builtin.debug`.

The module signature for reference:

```yaml
- name: Get address range objects
  check_point.mgmt.cp_mgmt_address_range_facts:
  register: address_range_out
```

---

## Exercise 4 — Write the handler

In `handlers/main.yml`, create a handler that publishes the CheckPoint
management session when triggered (i.e. when a configuration change occurs).

> **Tip:** Use `check_point.mgmt.cp_mgmt_publish` — it takes no required
> arguments.

---

## Exercise 5 — Write the top-level playbook

Create `~/ansible/lab2-3/site.yml` that:

- Targets `lab_mgmt`
- Uses `gather_facts: false`
- Calls the `address_range` role

Run it and verify that:
1. All three address ranges are created (or reported as unchanged on a second run)
2. The facts task returns the address range objects
3. The publish handler fires on first run but is skipped on subsequent runs

---

*Next: LAB 2.4 — When / Conditionals*
