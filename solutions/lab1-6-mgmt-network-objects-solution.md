# LAB 1.6 — Management Network Objects Solution (Trainer Reference)

---

## Complete playbook — `~/ansible/lab1-6/mgmt_net_objects.yml`

```yaml
---
- name: Network objects
  hosts: mgmt
  gather_facts: false

  tasks:
    - name: Retrieve network objects
      check_point.mgmt.cp_mgmt_network_facts:
      register: net_result

    - name: Print full result
      ansible.builtin.debug:
        var: net_result

    - name: Extract first network object
      ansible.builtin.set_fact:
        first_object: "{{ net_result.ansible_facts.checkpoint_networks.objects[0] }}"

    - name: Extract name, subnet and mask from first object
      ansible.builtin.set_fact:
        obj_name:   "{{ first_object.name }}"
        obj_subnet: "{{ first_object.subnet4 }}"
        obj_mask:   "{{ first_object['subnet-mask'] }}"

    - name: Print object info
      ansible.builtin.debug:
        msg: "Name: {{ obj_name }}  Network: {{ obj_subnet }}/{{ obj_mask }}"
```

---

## Bonus — create objects and loop over all IPv4 objects

Add the following at the top of the task list (before `cp_mgmt_network_facts`),
plus a handler section:

```yaml
  vars:
    lab_networks:
      - { name: "Lab Net 1", subnet: "10.10.1.0", mask_length: 24 }
      - { name: "Lab Net 2", subnet: "10.10.2.0", mask_length: 24 }
      - { name: "Lab Net 3", subnet: "10.10.3.0", mask_length: 24 }

  handlers:
    - name: publish
      check_point.mgmt.cp_mgmt_publish:

  tasks:
    - name: Create lab network objects
      check_point.mgmt.cp_mgmt_network:
        name: "{{ item.name }}"
        subnet: "{{ item.subnet }}"
        mask_length: "{{ item.mask_length }}"
        state: present
      loop: "{{ lab_networks }}"
      notify: publish

    # ... existing tasks follow (retrieve, extract, print) ...
```

Then add the loop task at the end:

```yaml
    - name: Print all IPv4 network objects
      ansible.builtin.debug:
        msg: "{{ item.name }}  {{ item.subnet4 }}/{{ item['subnet-mask'] }}"
      loop: "{{ net_result.ansible_facts.checkpoint_networks.objects }}"
      loop_control:
        label: "{{ item.name }}"
      when: item.subnet4 is defined
```

> **Why `when: item.subnet4 is defined`:** The gateway includes IPv6-only built-in
> objects that have no `subnet4` field — accessing it would throw an undefined error.

---

## Expected output

```
TASK [Retrieve network objects] ********************************
ok: [lab_mgmt]

TASK [Extract first network object] ****************************
ok: [lab_mgmt]

TASK [Extract name, subnet and mask from first object] *********
ok: [lab_mgmt]

TASK [Print object info] ***************************************
ok: [lab_mgmt] => {
    "msg": "Name: <object_name>  Network: <subnet>/<mask>"
}
```

> The exact object name depends on what's pre-configured on the gateway.
> On a fresh standalone, common defaults include `All_Internet`, `LocalMachine`,
> or RFC 1918 network objects. Tell students to use whichever first object
> their gateway returns — the exercise is about the data extraction pattern.

---

## Key teaching points

| Point | Explanation |
|-------|-------------|
| `ansible_facts` nesting | `cp_mgmt_network_facts` stores results under `ansible_facts.checkpoint_networks.objects` — always `register` + print first |
| `item['subnet-mask']` vs `item.subnet-mask` | Hyphenated keys can't use dot notation in Jinja2 — must use bracket syntax |
| `loop_control.label` | Replaces the default `item` in loop output with a readable identifier; keeps long JSON blobs out of the task summary |
| `[0]` indexing | Valid in Jinja2 for list access; equivalent to Python list indexing |
