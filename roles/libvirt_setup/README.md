# Ansible Role: libvirt_setup

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

- `libvirt_setup_guests_BYPASS_CACHE` (number, optional), default `0`
- `libvirt_setup_guests_ON_BOOT` (string, optional), default `start`
- `libvirt_setup_guests_ON_SHUTDOWN` (string, optional), default `suspend`
- `libvirt_setup_guests_PARALLEL_SHUTDOWN` (number, optional), default `0`
- `libvirt_setup_guests_PERSISTENT_ONLY` (string, optional), default `default`
- `libvirt_setup_guests_SHUTDOWN_TIMEOUT` (number, optional), default `300`
- `libvirt_setup_guests_START_DELAY` (number, optional), default `0`
- `libvirt_setup_guests_SYNC_TIME` (number, optional), default `0`


## Internal Role Variables

None.


# Dependencies

None.


# Example Playbook

```yaml
- hosts: all
  tasks:
    - name: Include role libvirt_setup
      ansible.builtin.include_role:
        name: libvirt_setup
```


# License

[UNLICENSE](./LICENSE)


# Author Information

lingling
