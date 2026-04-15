# LAB 1.4 — Gaia Routes

**Goal:** Connect to the Gaia OS, retrieve the routing table, and
extract the default route using `set_fact` and `debug`.

This lab introduces the `check_point.gaia` collection, which talks to
the **Gaia REST API** (`/gaia_api/`) — a different API endpoint than the
Management API (`/web_api/`), even though both use HTTPAPI over port 443.

Use the inventory from LAB 1.1 (`~/ansible/lab1-1/hosts`).

---

## Exercise 1 — Retrieve the routing table

Create `~/ansible/lab1-3/gaia_routes.yml`:

- Play targeting `gaia`, `gather_facts: false`
- Task using `check_point.gaia.cp_gaia_routes_facts` — register the result
- A `debug` task to print the registered output

Run the playbook and inspect the full output. What structure does the
routing table have? How are the routes stored in the returned data?

---

## Exercise 2 — Extract the route table

Add a task using `set_fact` to extract just the list of routes from the
registered output into a variable named `route_table`.

Print `route_table` with `debug` to confirm it contains only the list.

---

## Exercise 3 — Find the default route

Add a task that iterates over `route_table` using a `loop` and uses
`set_fact` to capture the entry where `address` equals `"0.0.0.0"`.

> **Tip:** Use `when: item.address == "0.0.0.0"` on a `set_fact` task
> with `loop: "{{ route_table }}"`.

---

## Exercise 4 — Print the result

Add a final `debug` task to print the default route prefix and its next-hop
(the `gateways` field in the route entry).

Expected output shape:

```
TASK [Display default route] ***********************************
ok: [lab_gaia] => {
    "msg": "Prefix: 0.0.0.0  Next Hop: [{'address': '<gateway_ip>', 'interface': 'eth0'}]"
}
```

---

*Next: LAB 1.5 — ansible-lint*
