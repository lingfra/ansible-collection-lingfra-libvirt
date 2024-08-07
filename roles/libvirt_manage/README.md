# Ansible Role: libvirt_manage

Manages virtual machines (_domains_) in [libvirt](https://libvirt.org/).


# Features

- Create multiple virtual machines (_domains_) in libvirt from a YAML list
specified in host variables.
- Define default values for all VMs and overwrite them per VM, if necessary.
Following values can be adjusted:
    - Memory of the VM.
    - Count of virtual CPUs
    - Size of virtual Disk.
    - Network: Virtual networking and bridged networking is supported.
- Syntax and validity tests for VM definition from variables. Happens before
defining the VMs to avoid partial deployments.
- Start of an installation image.
- Already existing VMs in the variables are ignored, enabling an idempotent
definition of VMs.


# Limitations

- Does not delete VMs.
- Does not modify existing VM specifications.
- No support for loading cloud images.


# Requirements

- An already working libvirt installation.
- A bootable installation image placed in a storage pool, accessible by libvirt.


# Role Variables


## External Variables

This role does not utilize external variables.


## Public Role Variables

Set defaults for all VMs:

- `libvirt_manage_install_image` (string, **mandatory**), default `null`. Image to
be started after virtual machine creation. It is necessary to set this variable
to an existing installation image, which can be found in `libvirt_manage_storage_pool`.
- `libvirt_manage_memory` (number), default `2048`. Amount of memory in MiB.
- `libvirt_manage_net` (mapping). Specification of the network, which the VM should
be attached to. Defaults to following values:
    - `name` (string), default `default`. Name of the virtual network or bridge.
    - `type` (string), default `network`. Supported types are `network` (libvirt
    virtual host based) and `bridge`. Note that the bridge must exist on the
    hypervisor.
- `libvirt_manage_storage_pool` (string), default `default`. Name of the storage
pool, where the VMs store their virtual disks and where the
`libvirt_manage_install_image` can be accessed.
- `libvirt_manage_vcpu_count` (number), default `2`. Count of virtual CPUs for the
VM.
- `libvirt_manage_virtual_disk_size` (number), default `20`: Size of the new
virtual disk in Gigabyte.

Define the VMs:

- `libvirt_manage_vm_list` (list), default `[]`. List of mappings specifying the VMs
to be managed. This list can be empty, but then the role won't do anything.
Following mapping fields are supported:
    - `name` (string, **mandatory**), no default. The name of the virtual
    machine (libvirt _domain_) to be defined. This name need to be unique.
    Already existing VMs will be skipped.
    - `disk_size` (number), default `libvirt_manage_virtual_disk_size`.
    - `memory` (number), default `libvirt_manage_memory`.
    - `net` (mapping), default `libvirt_manage_net`.
    - `vcpu_count` (number), default `libvirt_manage_vcpu_count`.


## Internal Role Variables

This role does not use internal variables.


# Dependencies

None.


# Example Playbook

You properly want to define some variables beforehand. Assuming the host running
libvirt is named `libvirt01.mydomain.tld` and is already in the inventory. This
example also assumes that a bootable ISO image is placed in the `default`
storage pool, usually at `/var/lib/libvirt/images`.

First define host variables in `host_vars/libvirt01.mydomain.tld/main.yaml`
containing the desired VM configuration:

```yaml
# Host vars for libvirt01.mydomain.tld

# Replace with an actually existing image.
libvirt_manage_install_image: archlinux-2024.08.01-x86_64.iso

# Overwrite role default of 2048. Every VM with no memory attribute will inherit this value.
libvirt_manage_memory: 1024

libvirt_manage_vm_list:
  # web01, web02 and web03 are created with the specified defaults.
  - name: web01
  - name: web02
  - name: web03

  # db01 and db02 overwrite the defaults.
  - name: db01
    memory: 4096
    vcpu_count: 4
    disk_size: 40
  - name: db02
    memory: 2048

  # Bridged networking is also available. Following example assumes a bridge with the
  # name br0 is defined on the host. This will fail if the bridge does not exist!
  - name: public01
    net:
      type: bridge
      name: br0
```

With the variables set the role can be executed:

```yaml
- name: Create virtual machines
  hosts: libvirt01.mydomain.tld
  tasks:
    - name: Include role libvirt_manage
      ansible.builtin.include_role:
        name: libvirt_manage
```

After playbook execution the VMs should be created and booted.

Further automation can be accomplished with the [libvirt dynamic inventory
plugin](https://docs.ansible.com/ansible/latest/collections/community/libvirt/libvirt_inventory.html)
and the `refresh_inventory` action from the
[ansible.builtin.meta](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/meta_module.html)
module.


# License

[UNLICENSE](./LICENSE)


# Author Information

lingling
