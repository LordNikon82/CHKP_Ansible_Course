# LAB 1.7 — Loops: Basics

**Goal:** Learn the different ways to loop in Ansible — over simple lists,
variables, dictionaries, and lists of dictionaries.

All tasks in this lab run against `localhost` — no CheckPoint connection needed.

Create `~/ansible/lab1-7/loops_basics.yml` for all exercises below.

---

## Exercise 1 — Simple inline loop

Create a play targeting `localhost` with `gather_facts: false`.

Add a task named `"Simple loop"` that uses `ansible.builtin.debug` with a
`loop:` to print the names of four firewalls inline:
`fw1`, `fw2`, `fw3`, `fw4`.

Run it and observe how `item` is printed for each iteration.

---

## Exercise 2 — Loop over a variable

In the `vars:` section of the play, define a list variable `my_firewalls`
containing the same four firewall names.

Add a second task that loops over `my_firewalls` and prints each name.

> **Question:** What is the difference in the playbook output compared to
> Exercise 1? What is the same?

---

## Exercise 3 — Loop over a dictionary

In `vars:`, add the following dictionary:

```yaml
firewall_dict:
  name: fw1
  location: Munich
  ip_address: 1.1.1.1
  sw_version: R82
```

Add a task that loops over this dictionary and prints both the **key** and
its **value** for each entry.

> **Tip:** Use `loop: "{{ firewall_dict | dict2items }}"`. The key is
> available as `item.key` and the value as `item.value`.

---

## Exercise 4 — Loop over a list of dictionaries

In `vars:`, add the following list of dictionaries:

```yaml
fw_list:
  - { name: fw1, location: Munich,  ip_address: 1.1.1.1,  sw_version: R82    }
  - { name: fw2, location: Munich,  ip_address: 1.1.1.2,  sw_version: R82    }
  - { name: fw3, location: Cologne, ip_address: 10.1.1.1, sw_version: R81.10 }
  - { name: fw4, location: Cologne, ip_address: 10.1.1.2, sw_version: R81.10 }
```

Add a task that loops over `fw_list` and prints each firewall's name,
location, IP address, and software version in a single `msg:`.

Use `loop_control` with `label: " "` to suppress the default item dump
in the task header line.

Expected output per iteration:

```
ok: [localhost] => (item= ) => {
    "msg": "fw1 | Munich | 1.1.1.1 | R82"
}
```

---

---

> **Reference solution available:** A working reference has been deployed to
> `~/ansible/lab1-7/loops_basics_ref.yml`. If you need a starting point:
> ```bash
> cp ~/ansible/lab1-7/loops_basics_ref.yml ~/ansible/lab1-7/loops_basics.yml
> ```

*Next: LAB 1.8 — Loops with CheckPoint*
