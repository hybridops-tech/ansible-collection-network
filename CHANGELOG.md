# Changelog

All notable changes to the `hybridops.network` collection will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/)
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.1.0] - 2026-04-03

### Added

- First public Galaxy release of the narrowed `hybridops.network` collection.
- `wan_edge` for legacy strongSwan and FRR WAN simulation and transitional Linux edge estates.
- `wan_validate` for convergence checks across IPsec, BGP, routes, and end-to-end reachability.
- A repeatable three-VM WAN simulator fixture for HyOps-backed smoke validation.

### Changed

- Narrowed the collection to the maintained Linux WAN simulator and validation path.
- Removed legacy device roles that are no longer part of the promoted release surface:
  - `base_config`
  - `compliance_check`
  - `configure_bgp`
  - `device_backup`
  - `hsrp_vrrp`
  - `ntp`
  - `ospf`
  - `vlan`
- Aligned CI and Galaxy release automation with the shared HybridOps collection harness.

[Unreleased]: https://github.com/hybridops-tech/ansible-collection-network/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/hybridops-tech/ansible-collection-network/releases/tag/v0.1.0
