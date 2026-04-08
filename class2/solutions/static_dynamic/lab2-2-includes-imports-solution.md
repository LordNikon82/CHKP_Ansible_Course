# LAB 2.2 — Static vs. Dynamic Solution (Trainer Reference)

> **Do not distribute to students.**

---

## Exercise 1 — include_tasks with a loop

**`~/ansible/lab2-2/subtask1.yml`**

```yaml
---
- name: Print IP address
  ansible.builtin.debug:
    msg: "Processing IP: {{ item }}"
```

**`~/ansible/lab2-2/includes_imports1.yml`**

```yaml
---
- name: include_tasks with loop
  hosts: localhost
  gather_facts: false

  tasks:
    - name: Process each IP address
      ansible.builtin.include_tasks: subtask1.yml
      loop:
        - 10.1.1.1
        - 10.1.1.2
        - 10.1.1.3
        - 10.1.1.4
```

**Expected output** (four iterations):

```
TASK [Print IP address] *****
ok: [localhost] => {
    "msg": "Processing IP: 10.1.1.1"
}
...
```

---

## Exercise 2 — Variable-driven task file name

**`~/ansible/lab2-2/host_vars/localhost.yml`**

```yaml
subtasks_file: "subtask1.yml"
```

**Updated task in `includes_imports1.yml`:**

```yaml
    - name: Process each IP address
      ansible.builtin.include_tasks: "{{ subtasks_file }}"
      loop:
        - 10.1.1.1
        - 10.1.1.2
        - 10.1.1.3
        - 10.1.1.4
```

### Answer — why does this work with include_tasks but not import_tasks?

`include_tasks` is **dynamic** — the file name is resolved at **runtime**,
after variables have been loaded. Using a variable for the file name is
perfectly valid.

`import_tasks` is **static** — Ansible resolves it at **parse time**, before
any variables from inventory or host_vars exist. A variable reference in the
file name would cause a parse error.

---

## Exercise 3 — Tags with include_tasks

**`~/ansible/lab2-2/subtask2.yml`**

```yaml
---
- name: Sub-task A
  ansible.builtin.debug:
    msg: "Sub-task A is running"
  tags: tag_a

- name: Sub-task B
  ansible.builtin.debug:
    msg: "Sub-task B is running"
  tags: tag_b

- name: Sub-task C
  ansible.builtin.debug:
    msg: "Sub-task C is running"
  tags: tag_c
```

**`~/ansible/lab2-2/includes_imports2.yml`** (initial version):

```yaml
---
- name: include_tasks with tags
  hosts: localhost
  gather_facts: false

  tasks:
    - name: Load sub-tasks
      ansible.builtin.include_tasks: subtask2.yml
```

### Answer — Step 2: Why doesn't --tags tag_a work?

`include_tasks` is evaluated at **runtime**. When Ansible parses the playbook,
it sees only the `include_tasks` statement — it cannot know what tasks (or
tags) are inside `subtask2.yml` until execution reaches that point. By the
time the sub-tasks are loaded, the tag filter has already been applied.

Running `--tags tag_a` tells Ansible to skip any task that does *not* have
`tag_a`. Because the `include_tasks` task itself has no tag, Ansible skips it
entirely — the sub-tasks are never loaded and never run.

### Fix — Step 3: apply the tags to the include itself

Add the `apply:` keyword to propagate the relevant tag down to the
`include_tasks` task itself, **and** tag the `include_tasks` call so it is
not skipped:

```yaml
---
- name: include_tasks with tags (fixed)
  hosts: localhost
  gather_facts: false

  tasks:
    - name: Load sub-tasks (tag_a)
      ansible.builtin.include_tasks:
        file: subtask2.yml
        apply:
          tags: tag_a
      tags: tag_a

    - name: Load sub-tasks (tag_b)
      ansible.builtin.include_tasks:
        file: subtask2.yml
        apply:
          tags: tag_b
      tags: tag_b

    - name: Load sub-tasks (tag_c)
      ansible.builtin.include_tasks:
        file: subtask2.yml
        apply:
          tags: tag_c
      tags: tag_c
```

> **Note:** `apply:` injects the listed tags onto every task loaded from the
> included file. The outer `tags:` on the `include_tasks` task ensures the
> include itself is not skipped by the tag filter.

Running `ansible-playbook includes_imports2.yml --tags tag_a` now executes
only Sub-task A.

---

## Key teaching points

| Concept | Explanation |
|---------|-------------|
| `include_tasks` (dynamic) | Resolved at **runtime** — supports variables in the file name and looping |
| `import_tasks` (static) | Resolved at **parse time** — no runtime variables in the file name; tags work transparently |
| Tags + `include_tasks` | The include itself must carry the tag; use `apply:` to propagate tags into included tasks |
| Tags + `import_tasks` | Tags on sub-tasks are visible at parse time — `--tags` works without extra configuration |
| Variable file name | Only possible with `include_tasks`; `import_tasks` will fail at parse time |
