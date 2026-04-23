# LAB 1.3 — Variables

**Goal:** Understand the different ways Ansible loads variables — inventory
variables, group_vars, vars_files, and set_fact — using your CheckPoint
gateway as the target.

Use the inventory from LAB 1.1 (`~/ansible/lab1-1/hosts`).

---

## Exercise 1 — Print ansible_facts

Create `~/ansible/lab1-3/variables.yml` with:

- A play targeting `lab_mgmt`
- `gather_facts: false`
- A single task using `debug` to print `ansible_facts`

Run it. What do you get? Why is `ansible_facts` empty?

---

## Exercise 2 — Print inventory variables

Add tasks to print the following variables for `lab_mgmt`:

- `ansible_network_os`
- `ansible_host`
- `inventory_hostname`

> **Tip:** These come directly from your inventory — no extra config needed.
> Use the `debug` module with `var:` or `msg:`.

---

## Exercise 3 — group_vars

Create a directory `~/ansible/lab1-3/group_vars/` and inside it a file
`all.yml` containing:

```yaml
cp_external_interface: "eth1"
```

Add a task to your playbook to print the value of `cp_external_interface`.

Run it. Where does Ansible look for `group_vars`? Does it find the file?

---

## Exercise 4 — vars_files and variable precedence

In `~/ansible/lab1-3/`, create a file `my_vars.yml` containing:

```yaml
cp_external_interface: "eth3"
```

Add `vars_files: my_vars.yml` to your play and re-run.

Which value wins — `"eth1"` from `group_vars/all.yml` or `"eth3"` from
`my_vars.yml`? Why?

> **Reference:** [Ansible variable precedence](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#understanding-variable-precedence)

---

## Exercise 5 — set_fact

Add a task using `set_fact` to create a new variable:

- Name: `gateway_fqdn`
- Value: combine `inventory_hostname` with the suffix `".cp.lab.internal"`

In a final task, print `gateway_fqdn`.

Expected debug output example:

```
ok: [lab_mgmt] => {
    "gateway_fqdn": "lab_mgmt.cp.lab.internal"
}
```

---

---

> **Reference solution available:** A working reference has been deployed to
> `~/ansible/lab1-3/variables_ref.yml` (with vars already inline). If you need a starting point:
> ```bash
> cp ~/ansible/lab1-3/variables_ref.yml ~/ansible/lab1-3/variables.yml
> ```

*Next: LAB 1.4 — Gaia Routes*
