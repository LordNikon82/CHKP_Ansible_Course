# LAB 1.2 — Solution (Trainer Reference)

---

## Exercises 1 + 2 — Complete playbook

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
