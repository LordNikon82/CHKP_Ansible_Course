# CheckPoint Firewall Automation with Ansible

Internal training course covering CheckPoint firewall automation using Ansible
and the `check_point.mgmt` collection.

## Structure

```
class1/
  exercises/
    inventory/      Lab 1.1 — INI inventory, groups, variables
  solutions/
    inventory/      Trainer reference solutions (spoilers!)
class2/             (upcoming)
```

## Prerequisites

- Access to the shared jump host (details in your personal handout)
- Ansible pre-installed on the jump host — nothing to install locally

## Collections used

- [`check_point.mgmt`](https://galaxy.ansible.com/ui/repo/published/check_point/mgmt/) — Management API automation
- [`check_point.gaia`](https://galaxy.ansible.com/ui/repo/published/check_point/gaia/) — Gaia OS (SSH-based)

## Training environment

Each student has an isolated pod in Azure:

```
[Jump Host] ──SSH──► [Student Workspace]
                            │
                            │ HTTPAPI (443)
                            ▼
                    [CheckPoint R82 Gateway+Mgmt]
                            │
                    ┌───────┴───────┐
                    ▼               ▼
               [Lab VM 1]      [Lab VM 2]
              (nginx)          (nginx)
```
