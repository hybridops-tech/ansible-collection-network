# `hybridops.network`

Network automation roles for routers and switches used by HybridOps.Studio. The collection focuses on practical workflows: baseline configuration, routing, VLANs, NTP, configuration backups, and lightweight compliance checks.

Role-level usage, variables, and assumptions are documented in each role’s `README.md`.

High-level platform context is maintained at [docs.hybridops.studio](https://docs.hybridops.studio).

## Scope

- Target devices: lab and brownfield networks using supported Ansible network transports (for example `ansible.netcommon.network_cli`).
- Initial validation focuses on Cisco IOS / IOS-XE style devices; vendor-specific logic is isolated per role where applicable.
- Roles aim to be vendor-aware without being vendor-locked.

## Roles

| Role | Purpose |
|------|---------|
| `base_config` | Device baseline (hostname, banners, logging hooks, and common settings). |
| `compliance_check` | Lightweight intent/compliance checks against running configuration. |
| `configure_bgp` | Configure BGP neighbours, ASN, and basic policy primitives. |
| `device_backup` | Capture device configuration backups to a defined backup root. |
| `hsrp_vrrp` | Configure HSRP/VRRP gateway redundancy. |
| `ntp` | Configure NTP servers and time source. |
| `ospf` | Configure OSPF process, areas, and interface participation. |
| `vlan` | Create VLANs and apply access/trunk port configuration. |

## Requirements

- Ansible `ansible-core` 2.15+.
- Python 3.10+ on the control node.
- Network connectivity from the control node to targets over the chosen transport (typically SSH).

Example inventory:

```ini
[edge]
edge01 ansible_host=192.0.2.10
edge02 ansible_host=192.0.2.11

[edge:vars]
ansible_network_os=cisco.ios.ios
ansible_connection=ansible.netcommon.network_cli
ansible_user=netadmin
ansible_password={{ vault_netadmin_password }}
```

## Installation

Install from Ansible Galaxy:

```bash
ansible-galaxy collection install hybridops.network
```

Pin in `collections/requirements.yml`:

```yaml
collections:
  - name: hybridops.network
    version: ">=0.1.0"
```

## Usage

```yaml
- name: Apply network baseline
  hosts: edge
  gather_facts: no
  collections:
    - hybridops.network

  roles:
    - role: hybridops.network.base_config
    - role: hybridops.network.ntp
```

## Examples

### Configure OSPF

```yaml
- name: Configure OSPF on core
  hosts: core
  gather_facts: no
  collections:
    - hybridops.network

  vars:
    ospf_process_id: 100
    ospf_router_id_map:
      core01: 10.0.255.1
      core02: 10.0.255.2
    ospf_networks:
      - { prefix: "10.0.0.0/24", area: 0 }
      - { prefix: "10.0.1.0/24", area: 0 }

  roles:
    - role: hybridops.network.ospf
```

### Nightly configuration backups

```yaml
- name: Nightly device backups
  hosts: all_network_devices
  gather_facts: no
  collections:
    - hybridops.network

  vars:
    backup_root: "/srv/network-backups"

  roles:
    - role: hybridops.network.device_backup
```

## Testing

- Role-local smoke tests are provided under `roles/<role>/tests/` and exercised against a lab inventory.
- Molecule scenarios may be provided where isolated validation is valuable.
- Platform integration tests are executed via HybridOps.Studio pipelines and lab inventories.

## License

- Code: [MIT-0](https://spdx.org/licenses/MIT-0.html)  
- Documentation & diagrams: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)

See the [HybridOps.Studio licensing overview](https://docs.hybridops.studio/briefings/legal/licensing/)
for project-wide licence details, including branding and trademark notes.
