# LAB 2.1 — Gaia Configuration

**Goal:** Use the `check_point.gaia` collection to configure Gaia OS settings
— static routes and DNS — and verify the changes using facts modules.

Use the inventory from LAB 1.1 (`~/ansible/lab1-1/hosts`). All playbooks
target the `gaia` host group.

> **Missing inventory?** It was restored as part of the Day 2 preparation.
> Verify with: `ansible-inventory -i ~/ansible/lab1-1/hosts --graph`

Create a working directory `~/ansible/lab2-1/` for all files in this lab.

---

## Background

The `check_point.gaia` collection talks to the **Gaia REST API** (`/gaia_api/`)
on port 443. This is a different API endpoint and collection from
`check_point.mgmt` — even though both run on the same standalone VM.

Key modules used in this lab:

| Module | Purpose |
|--------|---------|
| `check_point.gaia.cp_gaia_static_route` | Add/update/remove a static route |
| `check_point.gaia.cp_gaia_dns` | Configure DNS servers |
| `check_point.gaia.cp_gaia_routes_facts` | Retrieve the full routing table |

---

## Exercise 1 — Configure a Static Route (20 min)

Create `~/ansible/lab2-1/static_route.yml`.

The playbook should configure the following static route on the Gaia OS:

| Setting      | Value             |
|--------------|-------------------|
| Destination  | `172.31.128.0/21` |
| Gateway      | your pod's default gateway (see tip below) |
| Priority     | `1`               |
| Type         | `gateway`         |

> **Important:** The gateway IP must be reachable from the CP VM. Use the
> default gateway of your pod — you can find it by running `cp_gaia_routes_facts`
> and looking at the next hop of the `0.0.0.0` (default) route.

Requirements:

- Target the `gaia` group, `gather_facts: false`
- Use `check_point.gaia.cp_gaia_static_route`
- Register the result and print it with `debug`

Module reference:

```yaml
- name: Configure static route
  check_point.gaia.cp_gaia_static_route:
    address: "172.31.128.0"
    mask_length: 21
    next_hop:
      - gateway: "<your-default-gateway-ip>"
        priority: 1
    type: gateway
    state: present
```

Run the playbook. Then run it a second time and verify `changed=0`
(idempotency).

---

## Exercise 2 — Configure DNS (20 min)

Create `~/ansible/lab2-1/dns.yml`.

Configure the following DNS settings on the Gaia OS:

| Setting   | Value         |
|-----------|---------------|
| Primary   | `172.31.0.2`  |
| Secondary | `8.8.8.8`     |
| Tertiary  | `8.8.4.4`     |

Requirements:

- Target the `gaia` group, `gather_facts: false`
- Use `check_point.gaia.cp_gaia_dns`
- Register and print the result

Module reference:

```yaml
- name: Configure DNS servers
  check_point.gaia.cp_gaia_dns:
    primary: "172.31.0.2"
    secondary: "8.8.8.8"
    tertiary: "8.8.4.4"
```

Run the playbook twice and verify idempotency.

---

## Exercise 3 — Verify the Static Route (20 min)

Create `~/ansible/lab2-1/verify_routes.yml`.

The playbook should:

1. Retrieve the full routing table using `check_point.gaia.cp_gaia_routes_facts`
   and register the result.

2. Use `set_fact` to extract the list of routes into a variable `route_table`.

3. Use a `set_fact` task with `loop` and `when: item.address == "172.31.128.0"`
   to find the static route you just configured and store it in a variable
   `static_route`.

4. Use `ansible.builtin.assert` to verify:
   - The route exists (the variable `static_route` is defined)
   - The next hop gateway matches the IP you configured

5. Print a final summary with `debug`:
   ```
   "Route 172.31.128.0/21 → next hop: <gateway>"
   ```

> **Tip:** Inspect the `cp_gaia_routes_facts` output first — print the full
> registered variable and identify the correct key path for the routes list
> and each route's fields. (You already did this in LAB 1.4.)

---

## Exercise 4 — Combine into one playbook (10 min)

Create `~/ansible/lab2-1/gaia_config.yml` that runs all three tasks in a
single play:

1. Configure static route
2. Configure DNS
3. Retrieve routes and assert the static route is present

Run the combined playbook. On a second run every task should report `ok` —
no `changed`.

---

*Next: LAB 2.2 — Management Configuration: Objects & Policy*
