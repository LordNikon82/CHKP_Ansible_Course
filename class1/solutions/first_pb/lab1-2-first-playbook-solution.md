# LAB 1.2a — Solution (Trainer Reference)

> **Do not distribute to students.**

---

## Exercises 1a + 1b + 1c + 1d — Complete playbook

`~/ansible/lab1-2/first_pb.yml`

```yaml
---
- name: First playbook
  hosts: local
  gather_facts: false          # 1b: removing fact gathering
  tasks:
    - name: Print hello
      debug:
        msg: "Hello from Ansible"

- name: Query CheckPoint API
  hosts: mgmt
  gather_facts: false
  tasks:
    - name: Show API versions          # 1c
      check_point.mgmt.cp_mgmt_show_api_versions:
      register: api_result

    - name: Print current API version  # 1d
      debug:
        var: api_result["current-version"]
```

**Run:**

```bash
ansible-playbook -i ~/ansible/lab1-1/hosts ~/ansible/lab1-2/first_pb.yml
```

---

## Key teaching points

| Point | Explanation |
|-------|-------------|
| `gather_facts: false` | Network devices (and HTTPAPI connections) don't support the standard fact-gathering mechanism — always disable it for CheckPoint plays |
| `register` | Stores the module return value in a variable for use in later tasks |
| `var:` vs `msg:` in debug | `var:` prints the raw value; `msg:` allows string formatting with `{{ }}` |

---

## Expected output (1c/1d)

```
TASK [Show API versions] *******************************
ok: [lab_mgmt]

TASK [Print current API version] ***********************
ok: [lab_mgmt] => {
    "api_result[\"current-version\"]": "1.9.1"
}
```

If the API is not responding yet (CP still initialising after start), the
error will be `Connection refused` on port 443 — wait ~2 minutes and retry.
