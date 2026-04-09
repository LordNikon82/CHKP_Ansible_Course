# LAB 2.2 — Management Configuration Solution (Trainer Reference)

> **Do not distribute to students.**

---

## Exercise 1 — `host_objects.yml`

```yaml
---
- name: Create DMZ host objects
  hosts: mgmt
  gather_facts: false

  tasks:
    - name: Create host objects
      check_point.mgmt.cp_mgmt_host:
        name: "{{ item.name }}"
        ipv4_address: "{{ item.ip }}"
        state: present
      loop:
        - { name: "DMZ DB1",        ip: "172.31.144.241" }
        - { name: "DMZ Email",      ip: "172.31.144.242" }
        - { name: "DMZ Reporting",  ip: "172.31.144.243" }
      notify: publish

  handlers:
    - name: publish
      check_point.mgmt.cp_mgmt_publish:
```

---

## Exercise 2 — `net_objects.yml`

```yaml
---
- name: Create network objects
  hosts: mgmt
  gather_facts: false

  vars:
    networks:
      - name: Voice Network
        subnet: "10.200.10.0"
        mask_length: 24
      - name: IT Mgmt
        subnet: "10.200.11.0"
        mask_length: 24
      - name: OOB Access Network
        subnet: "198.51.100.0"
        mask_length: 26

  tasks:
    - name: Create network objects
      check_point.mgmt.cp_mgmt_network:
        name: "{{ item.name }}"
        subnet: "{{ item.subnet }}"
        mask_length: "{{ item.mask_length }}"
        state: present
      loop: "{{ networks }}"
      notify: publish

  handlers:
    - name: publish
      check_point.mgmt.cp_mgmt_publish:
```

---

## Exercise 3 — `fw_policy.yml`

```yaml
---
- name: Configure Corp Web Server rule
  hosts: mgmt
  gather_facts: false

  tasks:
    - name: Create Corp Web Server host object
      check_point.mgmt.cp_mgmt_host:
        name: "Corp Web Server"
        ipv4_address: "172.31.144.220"
        state: present
      notify: publish

    - name: Add HTTP/HTTPS rule for Corp Web Server
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
      notify:
        - publish
        - install policy

  handlers:
    - name: publish
      check_point.mgmt.cp_mgmt_publish:

    - name: install policy
      check_point.mgmt.cp_mgmt_install_policy:
        policy_package: Training
        install_on_all_cluster_members_or_fail: false
```

> **Handler order matters:** `publish` must be defined before `install policy`.
> Ansible fires handlers in definition order, so publish always runs first.

---

## Key teaching points

| Concept | Explanation |
|---------|-------------|
| Handler notify | A task only notifies its handler if the task result is `changed`. If the object already exists, no notification is sent — publish is skipped. |
| Handler deduplication | Even if three loop iterations each notify the same handler, the handler fires exactly once at the end of the play. |
| `state: present` idempotency | All `cp_mgmt_*` modules check whether the object already matches the desired state. If yes, result is `ok` (not `changed`). |
| Handler ordering | Handlers execute in the order they are **defined**, not the order they are notified. Always define `publish` before `install policy`. |
| `install_on_all_cluster_members_or_fail: false` | Required on standalone deployments — without it the module errors if there is no cluster. |
