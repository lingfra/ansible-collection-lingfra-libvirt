# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.0.0] - 2024-12-31

- Added role libvirt_install
  - Install libvirt and dependencies
  - Optionally set `nocow` attribute
  - Add users to `libvirt` group to manage the daemon
- Added role libvirt_guests
  - Configure libvirt-guests.service
- Added role libvirt_manage
  - Create virtual machines from variables
  - Define default values and overwrite them per VM
- Added playbook libvirt_setup
  - Includes the roles libvirt_install and libvirt_guests

[Unreleased]: https://github.com/lingfra/ansible-collection-lingfra-libvirt/compare/v1.0.0..HEAD
[1.0.0]: https://github.com/lingfra/ansible-collection-lingfra-libvirt/releases/tag/v1.0.0
