# LAB 1.3a — Gaia Routes

**Goal:** Connect to the Gaia OS via SSH, retrieve the routing table, and
extract the default route using `set_fact` and `debug`.

This lab introduces the `check_point.gaia` collection, which talks to
Gaia over SSH — a different connection method than the Management API (HTTPAPI).

---

## Setup — Install the Gaia collection

```bash
ansible-galaxy collection install check_point.gaia
```

---

## Setup — Extend your inventory

Add a `[gaia]` group and its variables to `~/ansible/lab1-1/hosts`.

The `lab_gaia` host reaches the same gateway IP as `lab_mgmt`, but uses SSH
(port 22) instead of HTTPAPI. The user is `admin`.

> **Tip:** Use a `[gaia:vars]` section. The required variables are:
> - `ansible_network_os` = `check_point.gaia.checkpoint`
> - `ansible_connection` = `network_cli`
> - `ansible_user` = `admin`
> - `ansible_password` = your admin password

---

## Exercise 1a — Retrieve the routing table

Create `~/ansible/lab1-3/gaia_routes.yml`:

- Play targeting `gaia`, `gather_facts: false`
- Task using `check_point.gaia.cp_gaia_routes_facts` — register the result

Run the playbook and inspect the full output. What structure does the
routing table have? How are the routes stored in the returned data?

---

## Exercise 1b — Extract the route table

Add a task using `set_fact` to extract just the list of routes from the
registered output into a variable named `route_table`.

Print `route_table` with `debug` to confirm it contains only the list.

---

## Exercise 1c — Find the default route

Add a task that iterates over `route_table` using a `loop` and uses
`set_fact` to capture the entry where `destination` equals `"0.0.0.0"`.

> **Tip:** Use `when: item.destination == "0.0.0.0"` on a `set_fact` task
> with `loop: "{{ route_table }}"`.

---

## Exercise 1d — Print the result

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

*Next: LAB 1.3b — ansible-lint*
