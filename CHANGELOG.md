# Changelog

All notable changes to the `hybridops.network` collection will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/)
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

- No unreleased changes yet.

## [0.1.0] - 2026-01-02

### Added

- Initial publication of the `hybridops.network` collection.
- Network roles for lab and brownfield devices:
  - `base_config`
  - `compliance_check`
  - `configure_bgp`
  - `device_backup`
  - `hsrp_vrrp`
  - `ntp`
  - `ospf`
  - `vlan`
- Support for Cisco-style devices (IOS / IOS-XE) via `network_cli`.
- `galaxy.yml` metadata for namespace `hybridops` and collection name `network`.

[Unreleased]: https://github.com/hybridops-studio/ansible-collection-network/compare/v0.1.0-network...HEAD
[0.1.0]: https://github.com/hybridops-studio/ansible-collection-network/releases/tag/v0.1.0-network