# LAB 1.1 — Solution (Trainer Reference)

> **Do not distribute to students.**
> Replace `<CP_PUBLIC_IP>` and `<PASSWORD>` with actual values from the handout.

---

## Exercise 1a — `~/ansible/lab1-1/hosts`

```ini
[all:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_user=admin
ansible_password=<PASSWORD>
ansible_httpapi_use_ssl=true
ansible_httpapi_validate_certs=false
ansible_httpapi_port=443

[local]

[gaia]

[mgmt]
```

**Expected `--list` output (abbreviated):**

```json
{
    "_meta": {
        "hostvars": {}
    },
    "all": {
        "children": ["local", "gaia", "mgmt", "ungrouped"]
    },
    "gaia": {},
    "local": {},
    "mgmt": {}
}
```

---

## Exercise 1b — Add hosts

```ini
[all:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_user=admin
ansible_password=<PASSWORD>
ansible_httpapi_use_ssl=true
ansible_httpapi_validate_certs=false
ansible_httpapi_port=443

[local]
localhost ansible_connection=local

[gaia]
lab_gaia ansible_host=<CP_PUBLIC_IP>

[mgmt]
lab_mgmt ansible_host=<CP_PUBLIC_IP>
```

**Expected `--graph` output:**

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

## Exercise 1c — Add network OS (group vars sections)

```ini
[all:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_user=admin
ansible_password=<PASSWORD>
ansible_httpapi_use_ssl=true
ansible_httpapi_validate_certs=false
ansible_httpapi_port=443

[local]
localhost ansible_connection=local

[gaia]
lab_gaia ansible_host=<CP_PUBLIC_IP>

[gaia:vars]
ansible_network_os=check_point.gaia.checkpoint

[mgmt]
lab_mgmt ansible_host=<CP_PUBLIC_IP>

[mgmt:vars]
ansible_network_os=check_point.mgmt.checkpoint
ansible_connection=httpapi
```

**Expected `--host lab_mgmt` output (key fields):**

```json
{
    "ansible_connection": "httpapi",
    "ansible_host": "<CP_PUBLIC_IP>",
    "ansible_httpapi_port": "443",
    "ansible_httpapi_use_ssl": "true",
    "ansible_httpapi_validate_certs": "false",
    "ansible_network_os": "check_point.mgmt.checkpoint",
    "ansible_password": "<PASSWORD>",
    "ansible_python_interpreter": "/usr/bin/python3",
    "ansible_user": "admin"
}
```

---

## Common mistakes to watch for

| Mistake | Symptom | Fix |
|---------|---------|-----|
| `ansible_httpapi_use_ssl=True` (capital T) | Ansible ignores it | Use lowercase `true` |
| Network OS in host line instead of `[group:vars]` | Works but hard to scale | Move to group vars section |
| `ansible_connection=httpapi` in `[all:vars]` | localhost also tries httpapi | Put it only in `[mgmt:vars]` |
| Password with special chars unquoted | Parsing errors | Wrap in quotes or use vault |

---

## Bonus answer

```bash
ansible lab_mgmt \
  -i ~/ansible/lab1-1/hosts \
  -m check_point.mgmt.cp_mgmt_show_api_versions
```

Returns JSON with `api_versions` list, e.g. `["1", "1.1", ..., "1.9.1"]`.
If API is not ready yet (CP still initialising), you get a connection refused error — wait and retry.
