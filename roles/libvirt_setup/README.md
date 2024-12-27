# Ansible Role: lingfra.libvirt.libvirt_setup

Installs and configures [libvirt](https://libvirt.org/) for KVM/QEMU.

The initial motivation behind this role was to install libvirt, while respecting
to set the `nocow` file system attribute when using btrfs. But this role was
further elaborated to also manage the libvirt-guests.service configuration.

## Features

- Install [libvirt](https://libvirt.org/) and dependencies.
- Configure libvirt-guests for power management coordination between host and
guests. Templates configuration file and manages service.
- Optionally set `nocow` file attribute to mitigate performance issues with CoW file
systems like [btrfs](https://btrfs.readthedocs.io/en/latest/index.html)
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
- Restarting `libvirt-guests.service` lead to power cycling the VMs. As this is
may not desired, restarting the service is deactivated and instead, a debug
message is printed by the handler. Nevertheless, the changes will take effect at
the next host reboot. Restarting the service can be activated by setting
`libvirt_setup_guests_service_allow_restart` to `true`. **Important Note**: When
`libvirt_setup_guests_ON_BOOT` is set to `ignore`, the VMs will be shutdown when
the service stop, but not booted again when the service starts.
- As setting the `nocow` attribute on folders only effects newly created files
in btrfs, this role will fail if the libvirt data directory already exists, but
is missing the `nocow` attribute. Either set `libvirt_setup_data_dir_set_nocow`
to `false` or resolve the issue manually.

## Requirements

- User with administrative permissions to install packages.
- Target host need to have network connection to a package mirror to install
packages.
- Specified users in `libvirt_setup_mgm_users` need to exist. The role will fail
if they don't.

## Role Variables

**Quick Start**: This role has sane defaults and does not need much
configuration. However, you want to set `libvirt_setup_data_dir_set_nocow` to
`false` if you are not using btrfs as file system. If you don't want to use
privilege escalation (e. g. with sudo) to manage libvirt you should append users
to `libvirt_setup_mgm_users`. See [Example Playbook](#example-playbook) for a
quick start.

### External Variables

This role depends on gathered facts by Ansible. It is the default behavior of an
Ansible play to gather facts and need no further configuration. Following
Ansible facts are utilized:

- `ansible_distribution`
- `ansible_distribution_major_version`

No custom external variables are needed or utilized by this role.

### Public Role Variables

**libvirt configuration**:

- `libvirt_setup_data_dir_set_nocow`, default `true`: Whether `nocow` (`C`) file
system attribute should be set on libvirt data directory or not. Having this set
to `true` also triggers a check on a potentially already existing libvirt data
directory. If the data directory already exist and haven't set the `nocow`
attribute the role fails.
- `libvirt_setup_mgm_users` (list), default `[]`. List of users, which should be
allowed to manage the libvirt daemon. These are added to the `libvirt` group.
Expected is a flat list of strings, reflecting existing users on the system. If
it is not desired to add any users to the `libvirt` group, the list can safely
be empty.

**libvirt-guests configuration**:

The configuration parameters for libvirt-guests configuration file can be set.
Refer to
**[libvirt-guests(8)](https://man.archlinux.org/man/extra/libvirt/libvirt-guests.8.en)**
for explanation of values. If a variable is set to `null` or not defined, only a
comment is written to the configuration file. Consequently libvirt-guests uses
its default option. Following variables are available:

- `libvirt_setup_guests_BYPASS_CACHE` (number), default `null`.
- `libvirt_setup_guests_ON_BOOT` (string), default `null`.
- `libvirt_setup_guests_ON_SHUTDOWN` (string), default `null`.
- `libvirt_setup_guests_PARALLEL_SHUTDOWN` (number), default `null`.
- `libvirt_setup_guests_PERSISTENT_ONLY` (string), default `null`.
- `libvirt_setup_guests_SHUTDOWN_TIMEOUT` (number), default `null`.
- `libvirt_setup_guests_START_DELAY` (number), default `null`.
- `libvirt_setup_guests_SYNC_TIME` (number), default `null`.

**libvirt-guests service management**:

- `libvirt_setup_guests_service_allow_restart`, default `false`: Whether a
restart of libvirt-guests.service is allowed or not. See
[Limitations](#limitations) for an explanation on implications of setting this
to `true`.

### Internal Role Variables

**General**:

- `libvirt_setup_libvirt_data_dir` (string), value `/var/lib/libvirt`. Directory
where libvirt data is stored. This path is needed in multiple tasks, therefore
the main reason behind with variable is to avoid multiple occurrences of
hardcoded paths in tasks.

**Host variables** are sourced at the beginning of the role. They need to be
named after the `ansible_distribution` variable. Following variables are
expected:

- `libvirt_setup_pkgs_absent` (list). Contains a flat list of strings reflecting
package names, which are conflicting with libvirt and therefore must not be
installed. Can safely be empty if no packages need removal.
- `libvirt_setup_pkgs_present` (list). Contains a flat list of strings reflecting
package names, which are the libvirt package itself and dependencies to use it
properly (e. g. hypervisor like QEMU and networking support).

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

## Dependencies

None.

## Example Playbook

The following example configuration and playbook assumes a host
`hypervisor.mydomain.tld` is present in the inventory. It is further assumed
that this host have ext4 as file system and therefore do not need the
`nocow` flag.

**Host configuration (`host_vars/hypervisor.mydomain.tld/main.yaml`)**:

```yaml
# Disable 'nocow' attribute, e. g. if the file system is ext4.
libvirt_setup_data_dir_set_nocow: false

# Allow users 'alice' and 'bob' to manage libvirt VMs.
libvirt_setup_mgm_users:
  - alice
  - bob

# Gracefully shutdown VMs (instead of suspend) when the host is powered off.
libvirt_setup_guests_ON_SHUTDOWN: shutdown

# With 'ignore' libvirt-guests does no action on host boot. Instead, only VMs
# marked with autostart will be started by libvirtd on host boot.
libvirt_setup_guests_ON_BOOT: ignore
```

**Playbook (`playbook.yaml`)**:

```yaml
- name: Install and configure libvirt
  hosts: hypervisor.mydomain.tld
  gather_facts: true
  tasks:
    - name: Include role lingfra.libvirt.libvirt_setup
      ansible.builtin.include_role:
        name: lingfra.libvirt.libvirt_setup
```

### Pre-Check Variable Syntax

In a larger playbook it may desirable to check the variable syntax in an early
stage, to prevent aborts in the middle of the playbook run. This role provides
the syntax checks in a separate file
([tasks/checks-syntax.yaml](./tasks/checks-syntax.yaml)) for this case.

```yaml
- name: Install and configure libvirt
  hosts: hypervisor.mydomain.tld
  pre_tasks:
    - name: Include lingfra.libvirt.libvirt_setup role variable syntax checks
      ansible.builtin.include_role:
        name: lingfra.libvirt.libvirt_setup
        tasks_from: checks-syntax.yaml
  tasks:
#     - name: Do some import stuff
#       ....

    - name: Include role lingfra.libvirt.libvirt_setup
      ansible.builtin.include_role:
        name: lingfra.libvirt.libvirt_setup
```

The tasks [check-host-config.yaml](./tasks/checks-host-config.yaml) are also
available for early inclusion or imports. But as they check that the users in
`libvirt_setup_mgm_users` are present on the system, they need to be placed on a
position where the users are already provisioned.

## Project Information

This role is part of the Ansible collection [lingfra.libvirt][libvirt].
For more information, see the collections [README](../../README.md) file.

[libvirt]: https://codeberg.org/lingfra/ansible-collection-lingfra-libvirt
