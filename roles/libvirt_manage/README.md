# Ansible Role: libvirt_manage

Manages virtual machines (_domains_) in [libvirt](https://libvirt.org/).


# Features


# Limitations


# Requirements

- An already working libvirt installation.


# Role Variables


## External Variables


## Public Role Variables

- `libvirt_manage_vm_list` (list), default `[]`. List of mappings specifying the VMs
to be managed. Following mapping fields are supported:
    - `name` (string, mandatory).
    - `disk_size` (number, optional).
    - `memory` (number, optional).
    - `net` (string, optional).
    - `vcpu_count` (number, optional).

## Internal Role Variables


# Dependencies

None.


# Example Playbook

```yaml
- hosts: all
  tasks:
    - name: Include role libvirt_manage
      ansible.builtin.include_role:
        name: libvirt_manage
```


# License

[UNLICENSE](./LICENSE)


# Author Information

lingling
