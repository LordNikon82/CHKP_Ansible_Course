# Class 2 — Exercises

> **Class 1 reference solutions** have been deployed to your workspace as `*_ref.yml`
> files in each lab directory (`lab1-2` through `lab1-8`). If you missed an exercise
> or need a working starting point, copy the reference:
> ```bash
> cp ~/ansible/lab1-4/gaia_routes_ref.yml ~/ansible/lab1-4/gaia_routes.yml
> ```
> All labs in Class 2 assume the inventory from LAB 1.1 is in place.
> If `~/ansible/lab1-1/hosts` is missing, it was also restored for you — check with
> `ansible-inventory -i ~/ansible/lab1-1/hosts --graph`.

---

## Module 1: Gaia Configuration

| Lab | Topic |
|-----|-------|
| [Lab 2.1](exercises/gaia_cfg/lab2-1-gaia-config.md) | Static routes, DNS configuration, route verification |

## Module 2: Management Configuration

| Lab | Topic |
|-----|-------|
| [Lab 2.2](exercises/mgmt_cfg/lab2-2-mgmt-config.md) | Host objects, network objects, access rules, handlers, policy install |

## Module 3: Static vs. Dynamic Includes

| Lab | Topic |
|-----|-------|
| [Lab 2.3](exercises/static_dynamic/lab2-3-includes-imports.md) | include_tasks / import_tasks — loops, variables, tag behaviour |

## Module 4: Roles

| Lab | Topic |
|-----|-------|
| [Lab 2.4](exercises/roles/lab2-4-roles.md) | Role structure, vars/, handlers/, import_tasks, idempotency |

## Module 5: Main Project

| Lab | Topic |
|-----|-------|
| [Lab 2.5](exercises/main_project/lab2-5-main-project.md) | Full project — Gaia config, mgmt objects, FW policy, shared handler role, vault, tags |
