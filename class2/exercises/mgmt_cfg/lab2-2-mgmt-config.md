# LAB 2.2 — Management Configuration: Objects & Policy

**Goal:** Use Ansible modules to create host objects, network objects, and a
firewall access rule on the CheckPoint management server. You will use loops,
handlers, and the `cp_mgmt_access_rule` module to build a simple DMZ policy.

Use the inventory from LAB 1.1 (`~/ansible/lab1-1/hosts`). All playbooks
target the `mgmt` host group.

---

## Background

The CheckPoint management collection provides dedicated modules for common
object types:

| Module | Purpose |
|--------|---------|
| `check_point.mgmt.cp_mgmt_host` | Create/update host objects |
| `check_point.mgmt.cp_mgmt_network` | Create/update network objects |
| `check_point.mgmt.cp_mgmt_access_rule` | Create/update access rules in a policy layer |
| `check_point.mgmt.cp_mgmt_publish` | Publish the current session |
| `check_point.mgmt.cp_mgmt_install_policy` | Install a policy package to targets |

All configuration modules use `state: present` to be idempotent — running the
playbook twice produces no further changes.

Changes are staged in a private session and only take effect when published.
A handler is the correct pattern: it publishes once at the end of the play,
even if multiple tasks reported changes.

---

## Exercise 1 — Host Objects (20 min)

Create `~/ansible/lab2-2/host_objects.yml`.

The playbook should create the following host objects on the management server:

| Name            | IPv4 address    |
|-----------------|-----------------|
| DMZ DB1         | 172.31.144.241  |
| DMZ Email       | 172.31.144.242  |
| DMZ Reporting   | 172.31.144.243  |

Requirements:

- Use a single task with a `loop` to create all three objects
- Each object should have `state: present`
- Notify a `publish` handler so the session is published if any object changed
- The handler should use `check_point.mgmt.cp_mgmt_publish`

Module reference:

```yaml
- name: Create host object
  check_point.mgmt.cp_mgmt_host:
    name: "My Host"
    ipv4_address: "10.0.0.1"
    state: present
```

Run the playbook and verify all three objects appear in SmartConsole.
Run it a second time — verify `changed=0` (idempotency).

---

## Exercise 2 — Network Objects (20 min)

Create `~/ansible/lab2-2/net_objects.yml`.

The playbook should create the following network objects:

| Name               | Network          |
|--------------------|-----------------|
| Voice Network      | 10.200.10.0/24  |
| IT Mgmt            | 10.200.11.0/24  |
| OOB Access Network | 198.51.100.0/26 |

Requirements:

- Use a single task with a `loop`
- Each object should have `state: present`
- Notify a `publish` handler on any change

The `cp_mgmt_network` module takes `subnet` and `mask_length` (integer) or
`subnet_mask` (dotted decimal). The cleanest approach is to split the CIDR in
your variables:

```yaml
vars:
  networks:
    - name: Voice Network
      subnet: "10.200.10.0"
      mask_length: 24
```

Module reference:

```yaml
- name: Create network object
  check_point.mgmt.cp_mgmt_network:
    name: "{{ item.name }}"
    subnet: "{{ item.subnet }}"
    mask_length: "{{ item.mask_length }}"
    state: present
```

---

## Exercise 3 — Firewall Access Rule (30 min)

Create `~/ansible/lab2-2/fw_policy.yml`.

The playbook should:

1. Create a host object `Corp Web Server` with IPv4 address `172.31.144.220`,
   and publish via handler if changed.

2. Add an access rule at position `1` in the `Training Network` layer (the
   layer created in the previous policy push) with:
   - Source: `Any`
   - Destination: `Corp Web Server`
   - Service: `http`, `https`
   - Action: `accept`
   - Track: log

   Use handlers to both **publish** and **install the policy** after the rule
   is added.

Module reference for the access rule:

```yaml
- name: Add access rule
  check_point.mgmt.cp_mgmt_access_rule:
    layer: "Training Network"
    position: "1"
    name: "Allow HTTP to Corp Web Server"
    source:
      - Any
    destination:
      - Corp Web Server
    service:
      - http
      - https
    action: accept
    track:
      type: log
    state: present
```

Module reference for policy install:

```yaml
- name: Install policy
  check_point.mgmt.cp_mgmt_install_policy:
    policy_package: Training
    install_on_all_cluster_members_or_fail: false
```

> **Tip:** Handlers run in the order they are defined. Define `publish` before
> `install policy` — you must publish before you can install.

Run the playbook and verify in SmartConsole that the rule appears at position 1
and the policy is installed.

---

*Next: LAB 2.3 — Static vs. Dynamic: include_tasks / import_tasks*
