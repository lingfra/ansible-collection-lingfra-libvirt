<!-- BEGIN_ANSIBLE_DOCS -->
# Ansible Role: lingfra.libvirt.libvirt_manage

Manages domains in libvirt.

## Features

Manages virtual machines (_domains_) in [libvirt](https://libvirt.org/).

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

## Limitations

- Does not delete VMs.
- Does not modify existing VM specifications.
- No support for loading cloud images.

## Supported Platforms

Not specified.

## Entrypoint `main` - Main entry point for libvirt_manage role

This is the main entrypoint for the libvirt_manage role.

### Options for `main`

#### `libvirt_manage_install_image`

Type: `str`, **required**.

Image to be started after virtual machine creation. It is necessary to set this variable to an existing installation image, which can be found in `libvirt_manage_storage_pool`.

#### `libvirt_manage_memory`

Type: `int`, Default: `2048`.

Amount of memory in MiB.

#### `libvirt_manage_net`

Type: dict of `libvirt_manage_net` [options](#options-for-main--libvirt_manage_net), Default: `{'name': 'default', 'type': 'network'}`.

Specification of the network, which the VM should be attached to.

#### `libvirt_manage_storage_pool`

Type: `str`, Default: `default`.

Name of the storage pool, where the VMs store their virtual disks and where the `libvirt_manage_install_image` can be accessed.

#### `libvirt_manage_vcpu_count`

Type: `int`, Default: `2`.

Count of virtual CPUs for the VM.

#### `libvirt_manage_virtual_disk_size`

Type: `int`, Default: `20`.

Size of the new virtual disk in Gigabyte.

#### `libvirt_manage_virtual_disk_type`

Type: `str`, Default: `qcow2`.

Type of the new virtual disk.

_Choices:_ `qcow2`, `raw`

#### `libvirt_manage_vm_list`

Type: list of dicts of `libvirt_manage_vm_list` [options](#options-for-main--libvirt_manage_vm_list), Default: `[]`.

List of mappings specifying the VMs to be managed. This list can be empty, but then the role won't do anything.

### Options for `main` > `libvirt_manage_net`

- **`name`** (Type: `str`, **required**).

  Name of the virtual network or bridge.

- **`type`** (Type: `str`, **required**).

  Supported types are `network` (libvirt virtual host based) and `bridge`. Note that the bridge must exist on the hypervisor.

  _Choices:_ `network`, `bridge`

### Options for `main` > `libvirt_manage_vm_list`

- **`name`** (Type: `str`, **required**).

  The name of the virtual machine (libvirt _domain_) to be defined. This name need to be unique. Already existing VMs will be skipped.

- `disk_size` (Type: `int`, optional).

  Overwrites `libvirt_manage_virtual_disk_size` on a per-VM basis.

- `memory` (Type: `int`, optional).

  Overwrites `libvirt_manage_memory` on a per-VM basis.

- `net` (Type: `dict`, optional).

  Overwrites `libvirt_manage_net` on a per-VM basis. See [`libvirt_manage_net`](#libvirt_manage_net) for required dict options.

- `vcpu_count` (Type: `int`, optional).

  Overwrites `libvirt_manage_vcpu_count` on a per-VM basis.

## Dependencies

None.

## Example Playbook

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
    - name: Include role lingfra.libvirt.libvirt_manage
      ansible.builtin.include_role:
        name: lingfra.libvirt.libvirt_manage
```

After playbook execution the VMs should be created and booted.

Further automation can be accomplished with the [libvirt dynamic inventory
plugin](https://docs.ansible.com/ansible/latest/collections/community/libvirt/libvirt_inventory.html)
and the `refresh_inventory` action from the
[ansible.builtin.meta](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/meta_module.html)
module.

## Project Information

This role is part of the Ansible collection [lingfra.libvirt][collection_url].
For more information, see the collections [README](../../README.md) file.

[collection_url]: https://github.com/lingfra/ansible-collection-lingfra-libvirt
<!-- END_ANSIBLE_DOCS -->
