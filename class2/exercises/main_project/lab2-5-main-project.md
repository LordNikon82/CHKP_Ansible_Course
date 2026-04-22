# LAB 2.5 — Main Project (1h)

**Goal:** Bring together everything from class 1 and class 2 into a single,
production-style Ansible project. You will configure Gaia system settings,
create management objects, and build a firewall policy — all using roles,
handlers, tags, and vault-encrypted credentials.

Use the inventory from LAB 1.1 (`~/ansible/lab1-1/hosts`).

---

## Overview

You will create `~/ansible/lab2-5/` with the following role structure:

```
lab2-5/
├── site.yml
├── group_vars/
│   └── all.yml          ← vault-encrypted credentials
└── roles/
    ├── handlers/         ← shared handlers (publish, install policy)
    ├── gaia/             ← Gaia OS configuration
    ├── mgmt_objects/     ← management server objects
    └── fw_policy/        ← firewall access rules
```

The `handlers` role is a **shared dependency** — the other three roles declare
it as a required role in their `meta/main.yml` so its handlers are available
across the whole play.

---

## Part 1 — Vault Credentials

Store your CheckPoint admin credentials in `group_vars/all.yml` and encrypt
the file with Ansible Vault:

```bash
ansible-vault create ~/ansible/lab2-5/group_vars/all.yml
```

The file should contain:

```yaml
ansible_user: admin
ansible_password: "<your-admin-password>"
```

> You will need to pass `--ask-vault-pass` (or use a vault password file) when
> running `ansible-playbook`.

Remove `ansible_user` and `ansible_password` from the inventory hosts entry
for `lab_mgmt` — they will now come from the vault-encrypted vars file.

---

## Part 2 — Shared Handlers Role

Create `roles/handlers/` with only a `handlers/main.yml` file:

```
roles/handlers/
└── handlers/
    └── main.yml
```

Define two handlers:

| Handler name     | Module                                    |
|------------------|-------------------------------------------|
| `publish`        | `check_point.mgmt.cp_mgmt_publish`        |
| `install policy` | `check_point.mgmt.cp_mgmt_install_policy` — install the `Training` policy package with `install_on_all_cluster_members_or_fail: false` |

> **Handler order matters:** `publish` must be listed before `install policy`.

---

## Part 3 — Gaia Role

Create `roles/gaia/` with this structure:

```
roles/gaia/
├── tasks/
│   ├── main.yml
│   ├── static_routes.yml
│   └── dns.yml
├── vars/
│   └── main.yml
└── meta/
    └── main.yml
```

### `meta/main.yml` — declare dependency on the handlers role

```yaml
dependencies:
  - role: handlers
```

### `vars/main.yml` — store all data here, not in the task files

Define variables for:
- A static route: network `172.31.128.0/21`, interface `eth1`, next hop `172.31.128.1`
- DNS settings: primary `172.31.0.2`, secondary `8.8.8.8`, tertiary `8.8.4.4`, domain `lasthop.io`

### `tasks/main.yml` — delegate to sub-task files

Use `import_tasks` to include `static_routes.yml` and `dns.yml`.

### `tasks/static_routes.yml`

Configure the static route using the `check_point.gaia.cp_gaia_static_route` module.

Module reference:

```yaml
- name: Configure static route
  check_point.gaia.cp_gaia_static_route:
    destination: "172.31.128.0/21"
    next_hop:
      - address: "172.31.128.1"
        priority: 1
    interface: "eth1"
    state: present
```

### `tasks/dns.yml`

**DNS server configuration** — use `check_point.gaia.cp_gaia_dns`:

```yaml
- name: Configure DNS servers
  check_point.gaia.cp_gaia_dns:
    primary: "172.31.0.2"
    secondary: "8.8.8.8"
    tertiary: "8.8.4.4"
    state: present
```

**DNS domain name** — use `check_point.mgmt.cp_mgmt_run_script` (via the
`clish -c "set domainname ..."` command). To make this task **idempotent**:

1. First retrieve the current domain name from Gaia facts or via a `show`
   command.
2. Only run `set domainname` if the current value is not already `lasthop.io`.

Use `when:` to conditionally execute the set task.

---

## Part 4 — Management Objects Role

Create `roles/mgmt_objects/` with:

```
roles/mgmt_objects/
├── tasks/
│   ├── main.yml
│   ├── host_objects.yml
│   ├── network_objects.yml
│   └── dmz_objects.yml
├── vars/
│   └── main.yml
└── meta/
    └── main.yml
```

Declare the `handlers` role as a dependency in `meta/main.yml`.

### Host objects (`host_objects.yml`)

| Name                      | IPv4 address    |
|---------------------------|-----------------|
| Ansible Server            | 3.125.34.232    |
| Windows SmartConsole      | 172.31.12.101   |
| Windows SmartConsole Public | 3.71.9.240    |

Use a loop. Notify the `publish` handler on any change.

### Network objects (`network_objects.yml`)

Create the following networks with **NAT auto-hide behind the gateway** enabled.

| Name         | Network           |
|--------------|-------------------|
| hq_net_128   | 172.31.128.0/24   |
| hq_net_129   | 172.31.129.0/24   |
| hq_net_130   | 172.31.130.0/24   |
| hq_net_131   | 172.31.131.0/24   |
| hq_net_132   | 172.31.132.0/24   |
| hq_net_133   | 172.31.133.0/24   |
| hq_net_134   | 172.31.134.0/24   |
| hq_net_135   | 172.31.135.0/24   |

Use a loop. Notify the `publish` handler on any change.

The `cp_mgmt_network` module supports NAT via the `nat_settings` parameter:

```yaml
nat_settings:
  auto_rule: true
  hide_behind: gateway
  method: hide
```

### DMZ objects (`dmz_objects.yml`)

Create a single host object: `Corp Web Server`, IP `172.31.144.220`.
Notify the `publish` handler on any change.

### `tasks/main.yml`

Use `import_tasks` to include all three task files in order:
`host_objects.yml` → `network_objects.yml` → `dmz_objects.yml`.

---

## Part 5 — Firewall Policy Role

Create `roles/fw_policy/` with:

```
roles/fw_policy/
├── tasks/
│   ├── main.yml
│   ├── mgmt_access.yml
│   └── dmz_policy.yml
├── vars/
│   └── main.yml
└── meta/
    └── main.yml
```

Declare `handlers` as a dependency in `meta/main.yml`.

### `tasks/mgmt_access.yml` — Management access rules

Add rules in the `Training Network` layer that allow the following hosts to
reach the firewall/management server (`cp_builtin_policy` or your gateway
object) with **any** service:

- Ansible Server
- Windows SmartConsole
- Windows SmartConsole Public

Also add a rule allowing **any** source to reach the firewall with **SSH** —
so you can SSH directly into clish/expert mode.

Use separate tasks or a loop. Notify `publish` and `install policy` on change.

### `tasks/dmz_policy.yml` — DMZ HTTP/HTTPS rule

Add a rule at position `1` allowing **Any** source to reach `Corp Web Server`
on HTTP and HTTPS. Notify `publish` and `install policy` on change.

### `tasks/main.yml`

Use `import_tasks` to include `mgmt_access.yml` then `dmz_policy.yml`.

---

## Part 6 — Top-level Playbook

Create `site.yml` with two plays:

```yaml
---
# Play 1: Gaia OS configuration (targets the gaia group)
- name: Configure Gaia OS
  hosts: gaia
  gather_facts: false
  tags: gaia
  roles:
    - gaia

# Play 2: Management objects and firewall policy (targets mgmt group)
- name: Configure management objects and policy
  hosts: mgmt
  gather_facts: false
  roles:
    - role: mgmt_objects
      tags: objects
    - role: fw_policy
      tags: policy
```

---

## Tags

You should be able to run individual parts of the project independently:

```bash
# Only Gaia configuration
ansible-playbook site.yml --tags gaia --ask-vault-pass

# Only management objects
ansible-playbook site.yml --tags objects --ask-vault-pass

# Only firewall policy
ansible-playbook site.yml --tags policy --ask-vault-pass

# Full run
ansible-playbook site.yml --ask-vault-pass
```

---

## Idempotency

Run `site.yml` twice. On the second run every task should report `ok` or
`skipped` — no `changed`. The only exception allowed is tasks that cannot be
made idempotent by the module (e.g. `cp_mgmt_run_script` for the domainname —
handle this with an explicit `when:` guard).

---

*This is the final exercise of Class 2.*

