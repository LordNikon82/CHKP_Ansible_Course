# LAB 1.6 — Management API: Network Objects

**Goal:** Use the Management API to retrieve network objects from your
CheckPoint gateway, extract specific fields, and print them cleanly.

This lab combines everything from Day 1: inventory, HTTPAPI connection,
`register`, `set_fact`, and `debug`.

Use the inventory from LAB 1.1 (`~/ansible/lab1-1/hosts`).

---

## Exercise 1 — Retrieve network objects

Create `~/ansible/lab1-6/mgmt_net_objects.yml`:

- Play targeting `mgmt`, `gather_facts: false`
- Task using `check_point.mgmt.cp_mgmt_network_facts` to retrieve all
  network objects — register the result
- A `debug` task to print the registered output

Run the playbook and inspect the full returned structure.

> **Question:** What is the top-level key that contains the list of objects?
> How many objects are returned by default on a freshly deployed gateway?

---

## Exercise 2 — Extract the first object

Add a `set_fact` task to extract the first network object from the returned
list into a variable named `first_object`.

Print `first_object` with `debug` to see its full structure.

---

## Exercise 3 — Extract specific fields

Add another `set_fact` task to extract these three fields from `first_object`
into separate variables:

| Variable | Source field |
|----------|-------------|
| `obj_name` | `name` |
| `obj_subnet` | `subnet4` |
| `obj_mask` | `subnet-mask` |

---

## Exercise 4 — Print a clean summary

Add a final `debug` task using `msg:` to print all three variables in a
readable format:

Expected output shape:

```
TASK [Print object info] *******************************
ok: [lab_mgmt] => {
    "msg": "Name: <name>  Network: <subnet>/<mask>"
}
```

---

## Bonus — Loop over all objects

First, extend your playbook to create three network objects so there is
something meaningful to loop over. Add these tasks **before** the
`cp_mgmt_network_facts` call:

| Name        | Subnet          | Mask length |
|-------------|-----------------|-------------|
| Lab Net 1   | 10.10.1.0       | 24          |
| Lab Net 2   | 10.10.2.0       | 24          |
| Lab Net 3   | 10.10.3.0       | 24          |

Use a single loop task with `check_point.mgmt.cp_mgmt_network` and a
`cp_mgmt_publish` handler that fires if anything changed.

Then re-run `cp_mgmt_network_facts` (already in the play) and add a loop
task that prints the name and subnet of **every IPv4 object** in the list.

> **Tips:**
> - Some built-in objects may be IPv6-only and will not have a `subnet4` field — skip those with `when: item.subnet4 is defined`
> - Use `loop_control.label` set to `item.name` to keep the output readable

---

*Next: LAB 1.7 — Loops: Basics*
