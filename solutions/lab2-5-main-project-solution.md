# LAB 2.5 — Main Project Solution (Trainer Reference)

---

## Directory structure

```
~/ansible/lab2-5/
├── site.yml
├── group_vars/
│   └── all.yml                          ← vault-encrypted
└── roles/
    ├── handlers/
    │   └── handlers/
    │       └── main.yml
    ├── gaia/
    │   ├── tasks/
    │   │   ├── main.yml
    │   │   ├── static_routes.yml
    │   │   └── dns.yml
    │   ├── vars/
    │   │   └── main.yml
    │   └── meta/
    │       └── main.yml
    ├── mgmt_objects/
    │   ├── tasks/
    │   │   ├── main.yml
    │   │   ├── host_objects.yml
    │   │   ├── network_objects.yml
    │   │   └── dmz_objects.yml
    │   ├── vars/
    │   │   └── main.yml
    │   └── meta/
    │       └── main.yml
    └── fw_policy/
        ├── tasks/
        │   ├── main.yml
        │   ├── mgmt_access.yml
        │   └── dmz_policy.yml
        ├── vars/
        │   └── main.yml
        └── meta/
            └── main.yml
```

---

## `group_vars/all.yml` (vault-encrypted)

Create with:
```bash
ansible-vault create ~/ansible/lab2-5/group_vars/all.yml
```

Contents:
```yaml
ansible_user: admin
ansible_password: "<admin-password>"
```

---

## `roles/handlers/handlers/main.yml`

```yaml
---
- name: publish
  check_point.mgmt.cp_mgmt_publish:

- name: install policy
  check_point.mgmt.cp_mgmt_install_policy:
    policy_package: Training
    install_on_all_cluster_members_or_fail: false
```

---

## Gaia role

### `roles/gaia/meta/main.yml`

```yaml
---
dependencies:
  - role: handlers
```

### `roles/gaia/vars/main.yml`

```yaml
---
static_route:
  address: "172.31.128.0"
  mask_length: 21
  next_hop: "172.31.128.1"

dns:
  primary: "172.31.0.2"
  secondary: "8.8.8.8"
  tertiary: "8.8.4.4"
  domain: "lasthop.io"
```

### `roles/gaia/tasks/main.yml`

```yaml
---
- name: Configure static routes
  ansible.builtin.import_tasks: static_routes.yml

- name: Configure DNS
  ansible.builtin.import_tasks: dns.yml
```

### `roles/gaia/tasks/static_routes.yml`

```yaml
---
- name: Configure static route for {{ static_route.address }}/{{ static_route.mask_length }}
  check_point.gaia.cp_gaia_static_route:
    address: "{{ static_route.address }}"
    mask_length: "{{ static_route.mask_length }}"
    type: gateway
    next_hop:
      - gateway: "{{ static_route.next_hop }}"
        priority: 1
    state: present
```

### `roles/gaia/tasks/dns.yml`

```yaml
---
- name: Configure DNS servers
  check_point.gaia.cp_gaia_dns:
    primary: "{{ dns.primary }}"
    secondary: "{{ dns.secondary }}"
    tertiary: "{{ dns.tertiary }}"

- name: Get current domain name
  check_point.gaia.cp_gaia_hostname_facts:
  register: gaia_facts

- name: Set domain name via clish (only if not already set)
  check_point.mgmt.cp_mgmt_run_script:
    script: 'clish -c "set domainname {{ dns.domain }}"'
    targets:
      - "{{ inventory_hostname }}"
  when: gaia_facts.ansible_facts.domainname | default('') != dns.domain
```

> **Note:** `cp_gaia_hostname_facts` may not expose `domainname` on all R82
> builds. An alternative is `cp_gaia_run_script` with `show domainname` and
> parse the result with `when: "'lasthop.io' not in show_result.response_message"`.

---

## mgmt_objects role

### `roles/mgmt_objects/meta/main.yml`

```yaml
---
dependencies:
  - role: handlers
```

### `roles/mgmt_objects/vars/main.yml`

```yaml
---
host_objects:
  - name: "Ansible Server"
    ipv4_address: "3.125.34.232"
  - name: "Windows SmartConsole"
    ipv4_address: "172.31.12.101"
  - name: "Windows SmartConsole Public"
    ipv4_address: "3.71.9.240"

network_objects:
  - { name: "hq_net_128", subnet: "172.31.128.0", mask_length: 24 }
  - { name: "hq_net_129", subnet: "172.31.129.0", mask_length: 24 }
  - { name: "hq_net_130", subnet: "172.31.130.0", mask_length: 24 }
  - { name: "hq_net_131", subnet: "172.31.131.0", mask_length: 24 }
  - { name: "hq_net_132", subnet: "172.31.132.0", mask_length: 24 }
  - { name: "hq_net_133", subnet: "172.31.133.0", mask_length: 24 }
  - { name: "hq_net_134", subnet: "172.31.134.0", mask_length: 24 }
  - { name: "hq_net_135", subnet: "172.31.135.0", mask_length: 24 }

dmz_objects:
  - name: "Corp Web Server"
    ipv4_address: "172.31.144.220"
```

### `roles/mgmt_objects/tasks/main.yml`

```yaml
---
- name: Configure host objects
  ansible.builtin.import_tasks: host_objects.yml

- name: Configure network objects
  ansible.builtin.import_tasks: network_objects.yml

- name: Configure DMZ objects
  ansible.builtin.import_tasks: dmz_objects.yml
```

### `roles/mgmt_objects/tasks/host_objects.yml`

```yaml
---
- name: Create host objects
  check_point.mgmt.cp_mgmt_host:
    name: "{{ item.name }}"
    ipv4_address: "{{ item.ipv4_address }}"
    state: present
  loop: "{{ host_objects }}"
  notify: publish
```

### `roles/mgmt_objects/tasks/network_objects.yml`

```yaml
---
- name: Create network objects with NAT auto-hide
  check_point.mgmt.cp_mgmt_network:
    name: "{{ item.name }}"
    subnet: "{{ item.subnet }}"
    mask_length: "{{ item.mask_length }}"
    nat_settings:
      auto_rule: true
      hide_behind: gateway
      method: hide
    state: present
  loop: "{{ network_objects }}"
  notify: publish
```

### `roles/mgmt_objects/tasks/dmz_objects.yml`

```yaml
---
- name: Create DMZ host objects
  check_point.mgmt.cp_mgmt_host:
    name: "{{ item.name }}"
    ipv4_address: "{{ item.ipv4_address }}"
    state: present
  loop: "{{ dmz_objects }}"
  notify: publish
```

---

## fw_policy role

### `roles/fw_policy/meta/main.yml`

```yaml
---
dependencies:
  - role: handlers
```

### `roles/fw_policy/vars/main.yml`

```yaml
---
policy_layer: "Training Network"
policy_package: "Training"
gateway_name: "vm-cp-student1"     # replace with actual gateway object name

mgmt_access_sources:
  - "Ansible Server"
  - "Windows SmartConsole"
  - "Windows SmartConsole Public"
```

### `roles/fw_policy/tasks/main.yml`

```yaml
---
- name: Configure management access rules
  ansible.builtin.import_tasks: mgmt_access.yml

- name: Configure DMZ policy rules
  ansible.builtin.import_tasks: dmz_policy.yml
```

### `roles/fw_policy/tasks/mgmt_access.yml`

```yaml
---
- name: Allow management hosts any access to firewall
  check_point.mgmt.cp_mgmt_access_rule:
    layer: "{{ policy_layer }}"
    position: "1"
    name: "Allow mgmt access from {{ item }}"
    source:
      - "{{ item }}"
    destination:
      - "{{ gateway_name }}"        # actual gateway object name (e.g. vm-cp-student1)
    service:
      - Any
    action: accept
    track:
      type: log
    state: present
  loop: "{{ mgmt_access_sources }}"
  notify:
    - publish
    - install policy

- name: Allow SSH from Any to firewall
  check_point.mgmt.cp_mgmt_access_rule:
    layer: "{{ policy_layer }}"
    position: "1"
    name: "Allow SSH to firewall"
    source:
      - Any
    destination:
      - "{{ gateway_name }}"
    service:
      - ssh
    action: accept
    track:
      type: log
    state: present
  notify:
    - publish
    - install policy
```

> **Note on `gateway_name`:** `CpmiGatewayNetobj` (a built-in object representing
> the local gateway) is not present on all R82 standalone builds. Use the actual
> gateway object name (e.g. `vm-cp-student1`) instead — find it in SmartConsole
> or via `show-gateways-and-servers` in the Management API.

### `roles/fw_policy/tasks/dmz_policy.yml`

```yaml
---
- name: Allow HTTP/HTTPS from Any to Corp Web Server
  check_point.mgmt.cp_mgmt_access_rule:
    layer: "{{ policy_layer }}"
    position: "1"
    name: "Allow HTTP/HTTPS to Corp Web Server"
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
```

---

## `site.yml`

```yaml
---
- name: Configure Gaia OS
  hosts: gaia
  gather_facts: false
  tags: gaia
  roles:
    - gaia

- name: Configure management objects and firewall policy
  hosts: mgmt
  gather_facts: false
  roles:
    - role: mgmt_objects
      tags: objects
    - role: fw_policy
      tags: policy
```

---

## Running the playbook

```bash
# Full run
ansible-playbook site.yml --ask-vault-pass

# Selective runs
ansible-playbook site.yml --tags gaia --ask-vault-pass
ansible-playbook site.yml --tags objects --ask-vault-pass
ansible-playbook site.yml --tags policy --ask-vault-pass
```

---

## Key teaching points

| Concept | Explanation |
|---------|-------------|
| Shared handler role | Handlers defined in a role are scoped to that play. By declaring the `handlers` role as a `meta` dependency, its handlers are available to all tasks in the play — no need to duplicate handler definitions. |
| Handler deduplication | Even if ten tasks notify `publish`, it fires exactly once at the end of the play. |
| Handler ordering | `publish` must be defined before `install policy` — Ansible fires handlers in definition order. |
| `meta/main.yml` dependencies | Listed roles are always applied before the current role. This is how the `handlers` role is loaded before any tasks run. |
| `import_tasks` vs `include_tasks` | `import_tasks` is static — tags applied to the import statement apply to all included tasks. Good choice when you want `--tags objects` to cover the entire objects task file. |
| Vault | `ansible-vault create/edit` encrypts the file. `--ask-vault-pass` decrypts at runtime. Never commit unencrypted credentials. |
| Idempotency with `run_script` | `cp_mgmt_run_script` always reports `changed`. Add a read-then-compare `when:` guard so the script only runs when the value actually needs to change. |
