# LAB 1.1 — Ansible Inventory (INI Format)

**Goal:** Build an INI-format inventory file step by step and learn how Ansible
resolves variables and groups.

Your CP Management IP and credentials are in your handout.

---

## Exercise 1a — Skeleton inventory with variables

Create the directory and file:

```bash
mkdir -p ~/ansible/lab1-1
```

Create `~/ansible/lab1-1/hosts` with the following structure:

- An `[all:vars]` section containing:
  - `ansible_python_interpreter` set to the system Python 3 path
  - `ansible_user` set to the CheckPoint admin username
  - `ansible_password` set to your admin password (see handout)
  - `ansible_httpapi_use_ssl = true`
  - `ansible_httpapi_validate_certs = false`
  - `ansible_httpapi_port = 443`
- Three empty groups: `[local]`, `[gaia]`, `[mgmt]`

**Validate:**

```bash
ansible-inventory -i ~/ansible/lab1-1/hosts --list
```

Expected: you should see the variables under `_meta` → `hostvars` and the
three groups (empty for now).

---

## Exercise 1b — Add hosts to groups

Add one host to each group:

| Group   | Host name  | Required variable                             |
|---------|-----------|-----------------------------------------------|
| `local` | `localhost` | `ansible_connection=local`                   |
| `gaia`  | `lab_gaia` | `ansible_host=<your CP public IP>`            |
| `mgmt`  | `lab_mgmt` | `ansible_host=<your CP public IP>`            |

**Validate with graph view:**

```bash
ansible-inventory -i ~/ansible/lab1-1/hosts --graph
```

Expected output shape:

```
@all:
  |--@local:
  |  |--localhost
  |--@gaia:
  |  |--lab_gaia
  |--@mgmt:
  |  |--lab_mgmt
  |--@ungrouped:
```

---

## Exercise 1c — Add network OS to groups

Add a **group variable** (not a host variable) to each network group so
Ansible knows which connection plugin to use:

| Group  | Variable              | Value                              |
|--------|-----------------------|------------------------------------|
| `gaia` | `ansible_network_os`  | `check_point.gaia.checkpoint`      |
| `mgmt` | `ansible_network_os`  | `check_point.mgmt.checkpoint`      |
| `mgmt` | `ansible_connection`  | `httpapi`                          |

Group variables in INI format use a `[groupname:vars]` section.

**Validate:**

```bash
ansible-inventory -i ~/ansible/lab1-1/hosts --host lab_mgmt
```

Expected: the output JSON should show `ansible_network_os` equal to
`check_point.mgmt.checkpoint` and `ansible_connection` equal to `httpapi`
(inherited from the group).

---

## Bonus — Test a live API call

Once the inventory looks correct, try a real API call against your gateway:

```bash
ansible lab_mgmt \
  -i ~/ansible/lab1-1/hosts \
  -m check_point.mgmt.cp_mgmt_show_api_versions
```

You should get back a JSON response listing available Management API versions.

---

*Next: LAB 1.2 — Your first CheckPoint playbook*
