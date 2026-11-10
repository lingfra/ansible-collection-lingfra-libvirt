<!-- BEGIN_ANSIBLE_DOCS -->
# Ansible Role: lingfra.libvirt.libvirt_guests

Configures libvirt-guests.service.

## Supported Platforms

Not specified.

## Entrypoint `main` - Main entry point for libvirt_guests role

Configure libvirt-guests for power management coordination between host and
guests. Templates configuration file and manages service.

Restarting `libvirt-guests.service` lead to power cycling the VMs. As this is
may not desired, restarting the service is deactivated and instead, a debug
message is printed by the handler. Nevertheless, the changes will take effect at
the next host reboot. Restarting the service can be activated by setting
`libvirt_guests_service_allow_restart` to `true`. **Important Note**: When
`libvirt_guests_conf_on_boot` is set to `ignore`, the VMs will be shutdown when
the service stop, but not booted again when the service starts.

### Options for `main`

#### `libvirt_guests_conf_bypass_cache`

Type: `int`, optional.

Refer to [**libvirt-guests(8)**](https://man.archlinux.org/man/extra/libvirt/libvirt-guests.8.en) variable `BYPASS_CACHE`.

#### `libvirt_guests_conf_on_boot`

Type: `str`, optional.

Refer to [**libvirt-guests(8)**](https://man.archlinux.org/man/extra/libvirt/libvirt-guests.8.en) variable `ON_BOOT`.

_Choices:_ `ignore`, `start`

#### `libvirt_guests_conf_on_shutdown`

Type: `str`, optional.

Refer to [**libvirt-guests(8)**](https://man.archlinux.org/man/extra/libvirt/libvirt-guests.8.en) variable `ON_SHUTDOWN`.

_Choices:_ `shutdown`, `suspend`

#### `libvirt_guests_conf_parallel_shotdown`

Type: `int`, optional.

Refer to [**libvirt-guests(8)**](https://man.archlinux.org/man/extra/libvirt/libvirt-guests.8.en) variable `PARALLEL_SHUTDOWN`.

#### `libvirt_guests_conf_persistent_only`

Type: `str`, optional.

Refer to [**libvirt-guests(8)**](https://man.archlinux.org/man/extra/libvirt/libvirt-guests.8.en) variable `PERSISTENT_ONLY`.

_Choices:_ `default`, `false`, `true`

#### `libvirt_guests_conf_shutdown_timeout`

Type: `int`, optional.

Refer to [**libvirt-guests(8)**](https://man.archlinux.org/man/extra/libvirt/libvirt-guests.8.en) variable `SHUTDOWN_TIMEOUT`.

#### `libvirt_guests_conf_start_delay`

Type: `int`, optional.

Refer to [**libvirt-guests(8)**](https://man.archlinux.org/man/extra/libvirt/libvirt-guests.8.en) variable `START_DELAY`.

#### `libvirt_guests_conf_sync_time`

Type: `int`, optional.

Refer to [**libvirt-guests(8)**](https://man.archlinux.org/man/extra/libvirt/libvirt-guests.8.en) variable `SYNC_TIME`.

#### `libvirt_guests_service_allow_restart`

Type: `bool`, Default: `False`.

Whether a restart of libvirt-guests.service is allowed or not. See the description for an explanation on implications of setting this to `true`.

## Dependencies

None.

## Tags

To simplify the testing of this role, all tasks are tagged. Check Ansible
documentation of
[tags](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_tags.html)
for reference. Following tags are available:

- `configure`: Added to tasks, which actually perform changes on the target host
or prepare for them.

## Project Information

This role is part of the Ansible collection [lingfra.libvirt][collection_url].
For more information, see the collections [README](../../README.md) file.

[collection_url]: https://github.com/lingfra/ansible-collection-lingfra-libvirt
<!-- END_ANSIBLE_DOCS -->
