# LAB 1.8 — Loops with CheckPoint: Gateway Interfaces

**Goal:** Combine loops with a `when:` conditional to filter data returned
from the CheckPoint Management API.

Use your inventory from LAB 1.1 (`~/ansible/lab1-1/hosts`).

Create `~/ansible/lab1-8/loops_checkpoint.yml`.

---

## Background

The module `check_point.mgmt.cp_mgmt_simple_gateway_facts` retrieves details
about a gateway object managed by your Management server.

It requires a `name:` argument — the object name as it appears in SmartConsole.

> **Find your gateway object name first:**
> ```bash
> ansible lab_mgmt -i ~/ansible/lab1-1/hosts \
>   -m check_point.mgmt.cp_mgmt_show_gateways_and_servers
> ```
> Look for the `name` field of your gateway in the output.

---

## Exercise 1 — Retrieve gateway facts

Create a play targeting `mgmt`, `gather_facts: false`.

Add a task that calls `check_point.mgmt.cp_mgmt_simple_gateway_facts` with
a `name:` argument set to your gateway object name. Register the result.

Print the full registered output with `debug` to explore the structure.

> **Question:** Under which key path does the list of interfaces live in the
> returned data?

---

## Exercise 2 — Extract the interface list

Add a `set_fact` task to extract the list of interfaces from the registered
output into a variable named `gw_interfaces`.

Print `gw_interfaces` to confirm it contains a list of interface objects,
each with fields like `name`, `ipv4-address`, and `ipv4-network-mask`.

---

## Exercise 3 — Loop with a conditional

Add a task that loops over `gw_interfaces` and prints the `name`,
`ipv4-address`, and `ipv4-network-mask` — but **only for `eth0`**.

All other interfaces should be skipped using a `when:` condition.

Use `loop_control.label` set to `item.name` so the loop header shows the
interface name instead of the full dict.

Expected output:

```
TASK [Print eth0 details] ******************************
skipping: [lab_mgmt] => (item=eth1)
ok: [lab_mgmt] => (item=eth0) => {
    "msg": "Interface: eth0 | IP: <ip> | Mask: <mask>"
}
```

---

## Bonus — Print all interfaces

Remove the `when:` and print details for **all** interfaces.

Then change the condition to print only interfaces with an IPv4 address
assigned (skip any that have an empty `ipv4-address`).

---

---

> **Reference solution available:** A working reference (with your gateway name
> pre-filled) has been deployed to `~/ansible/lab1-8/loops_checkpoint_ref.yml`.
> If you need a starting point:
> ```bash
> cp ~/ansible/lab1-8/loops_checkpoint_ref.yml ~/ansible/lab1-8/loops_checkpoint.yml
> ```

*End of Day 1 — well done!*
