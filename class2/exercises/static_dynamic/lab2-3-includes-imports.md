# LAB 2.3 — Static vs. Dynamic: include_tasks / import_tasks

**Goal:** Understand the difference between dynamic (`include_tasks`) and static
(`import_tasks`) task inclusion — and how that difference affects loops, variables,
and tag behaviour.

All tasks in this lab run against `localhost` — no CheckPoint connection needed.

Create a working directory `~/ansible/lab2-2/` for all files in this lab.

---

## Exercise 1 — include_tasks with a loop

Create a main playbook `~/ansible/lab2-2/includes_imports1.yml` with:

- A play targeting `localhost` with `gather_facts: false`
- A task that uses `include_tasks` to load `subtask1.yml`
- A `loop:` on that task that iterates over four IP addresses:
  `10.1.1.1`, `10.1.1.2`, `10.1.1.3`, `10.1.1.4`

Create `~/ansible/lab2-2/subtask1.yml` containing:

- A single `ansible.builtin.debug` task that prints the current IP address
  (`item`) with a message such as `"Processing IP: <ip_address>"`

Run the playbook and verify that all four IP addresses are printed.

---

## Exercise 2 — Variable-driven task file name

Add the name of your sub-tasks file into
`~/ansible/lab2-2/host_vars/localhost.yml`:

```yaml
subtasks_file: "subtask1.yml"
```

Update your main playbook to replace the hard-coded `subtask1.yml` filename
with the variable `subtasks_file`.

Re-run the playbook — the output should be identical to Exercise 1.

> **Question:** Why does using a variable here work with `include_tasks` but
> would **not** work with `import_tasks`?

---

## Exercise 3 — Tags with include_tasks

Create `~/ansible/lab2-2/subtask2.yml` with three `ansible.builtin.debug`
tasks. Each task should:

- Print a slightly different message (e.g. `"Sub-task A"`, `"Sub-task B"`,
  `"Sub-task C"`)
- Have its own unique tag (`tag_a`, `tag_b`, `tag_c`)

Create a second main playbook `~/ansible/lab2-2/includes_imports2.yml` that
uses `include_tasks: subtask2.yml` — no loop is needed.

**Step 1 — Run without tags:**

```bash
ansible-playbook includes_imports2.yml
```

Verify that all three sub-tasks execute.

**Step 2 — Run with a tag:**

```bash
ansible-playbook includes_imports2.yml --tags tag_a
```

Does only Sub-task A execute? Why or why not?

> **Think about it:** When does Ansible evaluate `include_tasks` versus
> `import_tasks`? How does that affect tag resolution?

**Step 3 — Fix it:**

Still using `include_tasks`, modify `includes_imports2.yml` so that passing
`--tags tag_a` causes **only** Sub-task A to execute.

> **Hint:** Split `subtask2.yml` into three separate files — one per sub-task.
> Include each file with its own `tags:` on the `include_tasks` statement.
> Also add `apply: tags: tag_X` matching the same tag — in Ansible ≥ 2.17,
> dynamically included tasks are also subject to tag filtering, so inner tasks
> need the tag injected via `apply:` or they will be skipped.

---

*Next: LAB 2.4 — Roles*
