<!-- BEGIN_ANSIBLE_DOCS -->
# Ansible Role: lingfra.libvirt.libvirt_install

Installs and configures libvirt.

The motivation behind this role is to install libvirt, while respecting to set
the `nocow` file system attribute when using btrfs.

## Features

- Install [libvirt](https://libvirt.org/) and dependencies.
- Optionally set `nocow` file attribute to mitigate performance issues with CoW
file systems like [btrfs](https://btrfs.readthedocs.io/en/latest/index.html)
(activated by default).
- Add users to `libvirt` group to manage the daemon.
- Comprehensive checks to prevent partial deployments.
- [Tagged](#tags) tasks to enable easy testing and debugging.
- Idempotent design.

## Limitations

- Package names are taken from host specific role variables, which are loaded in
the beginning of the role execution. Currently only Arch Linux is supported. If
support for another distribution is desired, you are encouraged to create a file
under [vars](./vars) and add the necessary packages to the lists. PRs are
welcome.
- As setting the `nocow` attribute on folders only effects newly created files
in btrfs, this role will fail if the libvirt data directory already exists, but
is missing the `nocow` attribute. Either set
`libvirt_install_data_dir_set_nocow` to `false` or resolve the issue manually.

## Requirements

- User with administrative permissions to install packages.
- Target host need to have network connection to a package mirror to install
packages.
- Specified users in `libvirt_install_mgm_users` need to exist. The role will
fail if they don't.

## Quick Start

This role has sane defaults and does not need much configuration. However, you
want to set `libvirt_install_data_dir_set_nocow` to `false` if you are not using
btrfs as file system. If you don't want to use privilege escalation (e. g. with
sudo) to manage libvirt you should append users to `libvirt_install_mgm_users`.
See [Example Playbook](#example-playbook) for a quick start and [Role
Arguments](#role-arguments) for an overview of all options.

## Supported Platforms

Not specified.

## Role Arguments

### Entrypoint `main` - Main entry point for libvirt_install role

This is the main entrypoint for the libvirt_install role.

#### `libvirt_install_data_dir_set_nocow`

Type: `bool`, Default: `True`.

Whether `nocow` (`C`) file system attribute should be set on libvirt data directory or not. Having this set to `true` also triggers a check on a potentially already existing libvirt data directory. If the data directory already exist and haven't set the `nocow` attribute the role fails.

#### `libvirt_install_libvirt_group_users`

Type: `list of 'str'`, Default: `[]`.

List of users, which should be allowed to manage the libvirt daemon. These are added to the `libvirt` group. Expected is a flat list of strings, reflecting existing users on the system. If it is not desired to add any users to the `libvirt` group, the list can safely be empty.

### Entrypoint `external` - Expected facts or global variables

This role depends on gathered facts by Ansible. It is the default behavior of an
Ansible play to gather facts and need no further configuration. Following
Ansible facts are utilized:

- `ansible_distribution`
- `ansible_distribution_major_version`

No custom external variables are needed or utilized by this role.

*This is a synthetic entry point, which is used for custom argument validation. You playbook will fail if you use this entry point.*

### Entrypoint `internal` - General internal role variables

This is a synthetic entry point, which is used for custom argument validation. You playbook will fail if you use this entry point.

#### `libvirt_install_libvirt_data_dir`

Type: `str`, **required**.

Directory where libvirt data is stored. This path is needed in multiple tasks, therefore the main reason behind with variable is to avoid multiple occurrences of hardcoded paths in tasks.

### Entrypoint `internal_os` - Host specific internal role variables

**Host variables** are sourced at the beginning of the role. They need to be named after the `ansible_distribution` variable.

This is a synthetic entry point, which is used for custom argument validation. You playbook will fail if you use this entry point.

#### `libvirt_install_pkgs_absent`

Type: `list of 'str'`, **required**.

Contains a flat list of strings reflecting package names, which are conflicting with libvirt and therefore must not be installed. Can safely be empty if no packages need removal.

#### `libvirt_install_pkgs_present`

Type: `list of 'str'`, **required**.

Contains a flat list of strings reflecting package names, which are the libvirt package itself and dependencies to use it properly (e. g. hypervisor like QEMU and networking support).

## Dependencies

None.

## Example Playbook

```yaml
- hosts: all
  tasks:
    - name: Import role lingfra.libvirt.libvirt_install
      ansible.builtin.import_role:
        name: lingfra.libvirt.libvirt_install
      vars:
        libvirt_install_libvirt_data_dir: # required, type: str
        libvirt_install_pkgs_absent: # required, type: list of 'str'
        libvirt_install_pkgs_present: # required, type: list of 'str'
```

The following example configuration and playbook assumes a host
`hypervisor.mydomain.tld` is present in the inventory. It is further assumed
that this host have ext4 as file system and therefore do not need the `nocow`
flag.

**Host configuration (`host_vars/hypervisor.mydomain.tld/main.yaml`)**:

```yaml
# Disable 'nocow' attribute, e. g. if the file system is ext4.
libvirt_install_data_dir_set_nocow: false

# Allow users 'alice' and 'bob' to manage libvirt VMs.
libvirt_install_libvirt_group_users:
  - alice
  - bob
```

**Playbook (`playbook.yaml`)**:

```yaml
- name: Install libvirt
  hosts: hypervisor.mydomain.tld
  gather_facts: true
  tasks:
    - name: Include role lingfra.libvirt.libvirt_install
      ansible.builtin.include_role:
        name: lingfra.libvirt.libvirt_install
```

## Tags

To simplify the testing of this role, all tasks are tagged. Check Ansible
documentation of
[tags](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_tags.html)
for reference. Following tags are available:

- `always`: The special tag `always` is used to enforce inclusion of OS specific
variables at the beginning of the role.
- `checks`: Added to all tasks, which perform checks or gather information for
subsequent checks. This tag contains a superset of tasks having a tag beginning
with `checks-`.
- `checks-syntax`: Added only to tasks performing simple syntax checks on
variables, e. g. if they are defined and have the correct data type.
- `checks-syntax-os`: Added to tasks checking host specific internal role
variables. Is a divergent set to `checks-syntax`.
- `checks-host-config`: Added to tasks, which check if host configuration does
comply with the configuration in variables.
- `configure`: Added to tasks, which actually perform changes on the target host
or prepare for them.

## Project Information

This role is part of the Ansible collection [lingfra.libvirt][collection_url].
For more information, see the collections [README](../../README.md) file.

[collection_url]: https://github.com/lingfra/ansible-collection-lingfra-libvirt
<!-- END_ANSIBLE_DOCS -->
