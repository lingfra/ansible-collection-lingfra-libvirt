# Ansible Role: libvirt

Installs and configures libvirt.


# Features

- Installs libvirt and dependencies.
- Configures libvirt-guests. Templates configuration file and manages service.


# Limitations

- Packages are hardcoded and cannot be modified.


# Requirements

None.


# Role Variables


## External Variables

None.


## Public Role Variables

The configuration parameters for libvirt-guests configuration file can be set.
Refer to libvirt-guests(8) for explanation of values. Following variables are
available:

- `libvirt_guests_BYPASS_CACHE` (number, optional), default `0`
- `libvirt_guests_ON_BOOT` (string, optional), default `start`
- `libvirt_guests_ON_SHUTDOWN` (string, optional), default `suspend`
- `libvirt_guests_PARALLEL_SHUTDOWN` (number, optional), default `0`
- `libvirt_guests_PERSISTENT_ONLY` (number, optional), default `0`
- `libvirt_guests_SHUTDOWN_TIMEOUT` (number, optional), default `300`
- `libvirt_guests_START_DELAY` (number, optional), default `0`
- `libvirt_guests_SYNC_TIME` (number, optional), default `0`


## Internal Role Variables

None.


# Dependencies

None.


# Example Playbook

```yaml
- hosts: all
  tasks:
    - name: Include role libvirt
      ansible.builtin.include_role:
        name: libvirt
```


# License

[UNLICENSE](./LICENSE)


# Author Information

lingling
