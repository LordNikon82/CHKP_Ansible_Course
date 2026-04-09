# LAB 1.3b — ansible-lint

**Goal:** Run `ansible-lint` against the playbook from LAB 1.3a, understand
the reported issues, and fix them until the linter passes cleanly.

ansible-lint enforces best practices and style rules. Getting familiar with it
early means fewer surprises in production pipelines and code reviews.

---

## Exercise 1a — First run

Run ansible-lint against your playbook from LAB 1.3a:

```bash
ansible-lint ~/ansible/lab1-3/gaia_routes.yml
```

Read each reported issue carefully. Common categories you will likely see:

| Rule tag | Meaning |
|----------|---------|
| `name[missing]` | Task has no `name:` |
| `name[casing]` | Task name does not start with a capital letter |
| `yaml[truthy]` | `yes`/`no` should be `true`/`false` |
| `fqcn[action-core]` | Should use fully qualified module name (`ansible.builtin.debug` not `debug`) |
| `no-free-form` | Avoid free-form module arguments |

---

## Exercise 1b — Fix all issues

Work through each lint violation and fix it in your playbook.

Re-run `ansible-lint` after each set of fixes until you reach:

```
Passed: 0 failure(s), 0 warning(s) on 1 files processed.
```

> **Tip:** Use the FQCN for all built-in modules:
> - `debug` → `ansible.builtin.debug`
> - `set_fact` → `ansible.builtin.set_fact`
> - `loop` stays as a task keyword, not a module

---

## Exercise 1c — Verify the playbook still works

Run the corrected playbook to confirm it still produces the same result
as before linting:

```bash
ansible-playbook -i ~/ansible/lab1-1/hosts ~/ansible/lab1-3/gaia_routes.yml
```

---

*Next: LAB 1.3c — Management API network objects*
