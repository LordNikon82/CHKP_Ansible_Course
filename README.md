# CheckPoint Firewall Automation with Ansible

Internal training course covering CheckPoint firewall automation using Ansible
and the `check_point.mgmt` and `check_point.gaia` collections.

## Structure

```
CHKP_Ansible_Course/
├── class1/
│   └── exercises/       Day 1 labs (LAB 1.1 – 1.8)
├── class2/
│   └── exercises/       Day 2 labs (LAB 2.1 – 2.4)
└── solutions/           Trainer reference solutions for all labs
```

## Prerequisites

- Access to the shared jump host (details in your personal handout)
- Ansible pre-installed on the jump host — nothing to install locally

## Collections used

- [`check_point.mgmt`](https://galaxy.ansible.com/ui/repo/published/check_point/mgmt/) — Management API automation
- [`check_point.gaia`](https://galaxy.ansible.com/ui/repo/published/check_point/gaia/) — Gaia OS REST API

## Training environment

Each student has an isolated pod in Azure:

```
[Jump Host] ──SSH──► [Student Workspace]
                            │
                            │ HTTPAPI (443)
                            ▼
                    [CheckPoint R82 Gateway+Mgmt]
```
