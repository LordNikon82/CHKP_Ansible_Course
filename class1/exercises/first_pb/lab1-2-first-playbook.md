# LAB 1.2a — Your First Playbook

**Goal:** Write and run your first Ansible playbook — first against localhost,
then against your CheckPoint Management API.

Use the inventory you created in LAB 1.1 (`~/ansible/lab1-1/hosts`).

---

## Exercise 1a — Hello World on localhost

Create a playbook `~/ansible/lab1-2/first_pb.yml` with:

- A single **play** targeting the `local` group
- A single **task** using the `debug` module to print `"Hello from Ansible"`

Run it:

```bash
ansible-playbook -i ~/ansible/lab1-1/hosts ~/ansible/lab1-2/first_pb.yml
```

Expected output shape:

```
PLAY [First playbook] ******************************************

TASK [Gathering Facts] *****************************************
ok: [localhost]

TASK [Print hello] *********************************************
ok: [localhost] => {
    "msg": "Hello from Ansible"
}

PLAY RECAP *****************************************************
localhost : ok=2  changed=0  unreachable=0  failed=0
```

---

## Exercise 1b — Disable fact gathering

Add `gather_facts: false` to your play and re-run.

What changes in the output? Why does this matter when working with network devices?

---

## Exercise 1c — First call to the CheckPoint API

Add a **second play** to the same playbook that:

- Targets the `mgmt` group
- Has `gather_facts: false`
- Uses the module `check_point.mgmt.cp_mgmt_show_api_versions` to query
  the Management API

> **Tip:** This module needs no arguments — it returns the supported API
> version list from your gateway.

Run the playbook again. The second play should connect to your CheckPoint
gateway and return a JSON response with the supported API versions.

Expected shape for the second play:

```
PLAY [Query CheckPoint API] ************************************

TASK [Show API versions] ***************************************
ok: [lab_mgmt] => {
    "ansible_facts": { ... },
    "changed": false,
    "current-version": "1.9.1",
    "supported-versions": ["1", "1.1", ...]
}
```

---

## Exercise 1d — Print a specific value

Add a `debug` task after the API call to print only the value of
`current-version` from the module output.

> **Tip:** Save the module result with `register`, then reference it in `debug`.

---

*Next: LAB 1.2b — Variables*
