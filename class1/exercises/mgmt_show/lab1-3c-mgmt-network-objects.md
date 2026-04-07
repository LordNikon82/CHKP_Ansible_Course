# LAB 1.3c — Management API: Network Objects

**Goal:** Use the Management API to retrieve network objects from your
CheckPoint gateway, extract specific fields, and print them cleanly.

This lab combines everything from Day 1: inventory, HTTPAPI connection,
`register`, `set_fact`, and `debug`.

---

## Exercise 1a — Retrieve network objects

Create `~/ansible/lab1-3/mgmt_net_objects.yml`:

- Play targeting `mgmt`, `gather_facts: false`
- Task using `check_point.mgmt.cp_mgmt_network_facts` to retrieve all
  network objects — register the result

Run the playbook and inspect the full returned structure.

> **Question:** What is the top-level key that contains the list of objects?
> How many objects are returned by default on a freshly deployed gateway?

---

## Exercise 1b — Extract the first object

Add a `set_fact` task to extract the first network object from the returned
list into a variable named `first_object`.

Print `first_object` with `debug` to see its full structure.

---

## Exercise 1c — Extract specific fields

Add another `set_fact` task to extract these three fields from `first_object`
into separate variables:

| Variable | Source field |
|----------|-------------|
| `obj_name` | `name` |
| `obj_subnet` | `subnet4` |
| `obj_mask` | `subnet-mask` |

---

## Exercise 1d — Print a clean summary

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

Instead of just the first object, add a task that loops over **all** returned
network objects and prints the name and subnet of each one.

> **Tip:** Use `loop` with `loop_control.label` to keep the output readable.

---

*End of Day 1 — well done!*
