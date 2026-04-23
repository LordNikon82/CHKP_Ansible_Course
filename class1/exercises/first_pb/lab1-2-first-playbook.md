# LAB 1.2 — Your First Playbook

**Goal:** Write and run your first Ansible playbook — first against localhost,
then against your CheckPoint Management API.

Use the inventory you created in LAB 1.1 (`~/ansible/lab1-1/hosts`).

---

## Exercise 1 — Hello World on localhost

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

## Exercise 2 — Disable fact gathering

Add `gather_facts: false` to your play and re-run.

What changes in the output? Why does this matter when working with network devices?

---

---

> **Reference solution available:** A working reference has been deployed to
> `~/ansible/lab1-2/first_pb_ref.yml`. If you need a starting point:
> ```bash
> cp ~/ansible/lab1-2/first_pb_ref.yml ~/ansible/lab1-2/first_pb.yml
> ```

*Next: LAB 1.3 — Variables*
