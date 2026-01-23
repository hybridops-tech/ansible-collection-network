# hybridops.network – Network Automation Collection

Network automation roles for HybridOps.Studio.  
The collection focuses on practical workflows for routers and switches: baseline configuration, routing, VLANs, NTP, device backups, and basic compliance checks.

The collection is reusable outside HybridOps.Studio and can target any supported lab or brownfield network.

---

## 1. Collection scope

**Collection name:** `hybridops.network`  
**Galaxy namespace/name:** `hybridops.network` (planned)  
**Source repository:** [github.com/hybridops-studio/ansible-collection-network](https://github.com/hybridops-studio/ansible-collection-network)

The collection targets:

- Cisco-style network devices (IOS / IOS-XE / similar) via SSH/NETCONF.  
- Baseline configuration, routing, VLANs, and HA protocols.  
- Repeatable backups and lightweight compliance checks.

It is consumed by the main HybridOps.Studio repository under `deployment/network_config/` playbooks, and can also be installed in other Ansible projects.

---

## 2. Roles

Current roles:

| Role name          | Purpose                                                              |
|--------------------|----------------------------------------------------------------------|
| `base_config`      | Device-wide baseline (hostname, AAA hooks, logging, banners, etc.). |
| `compliance_check` | Run lightweight intent/compliance checks against running config.     |
| `configure_bgp`    | Configure BGP neighbours, ASN, and basic policies.                  |
| `device_backup`    | Take regular text backups of device configuration.                  |
| `hsrp_vrrp`        | Configure HSRP/VRRP gateway redundancy.                             |
| `ntp`              | Configure NTP servers and time source.                              |
| `ospf`             | Configure OSPF areas, interfaces, and router-ids.                   |
| `vlan`             | Create VLANs and assign access/trunk ports.                         |

Planned additions (not yet committed):

- `interfaces_l3` – routed interface and SVI configuration.  
- `logging` – central syslog target and severity levels.  
- `radius_tacacs` – RADIUS/TACACS integration.

---

## 3. Requirements

- Ansible **2.15+**.  
- Python **3.10+** on the control node.  
- Network OS:
  - Initially tested with Cisco IOS / IOS-XE (lab images).

Connection requirements:

- `ansible_connection: ansible.netcommon.network_cli` (or equivalent).  
- Working SSH reachability from the control node.

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

---

## 4. Installation

From any Ansible project that will use the collection:

```bash
ansible-galaxy collection install hybridops.network
```

In `collections/requirements.yml`:

```yaml
collections:
  - name: hybridops.network
    version: "*"
```

Playbook example:

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

---

## 5. Quick examples

### 5.1 Configure OSPF on core routers

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

### 5.2 Nightly configuration backups

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

---

## 6. Relationship to HybridOps.Studio

This collection is maintained as part of the HybridOps.Studio platform.  
The main platform repository consumes it from Galaxy under `deployment/network_config/` and combines it with NetBox, Proxmox SDN, and evidence helpers for end-to-end scenarios.

For a complete view of how these roles fit into the wider platform (NetBox as source of truth, SDN, evidence), see the documentation site at [docs.hybridops.studio](https://docs.hybridops.studio).

---

## 7. Testing and CI

Network roles in this collection are validated at multiple levels:

- **Self-contained tests** under `tests/` (for example `tests/inventory.example.ini` and `tests/smoke.yml`) to exercise a single role such as `base_config` or `vlan` against lab devices.
- **Molecule scenarios** under `molecule/default/` for roles that benefit from container- or lab-based validation, including multi-device or topology-aware tests.
- **Platform integration tests** in the HybridOps.Studio platform repository, where playbooks under `deployment/network_config/` and CI pipelines exercise `hybridops.network` against real lab devices via NetBox inventories.

A repository-level `Makefile` provides a thin wrapper around Molecule:

- `make test` runs `molecule test` for all roles that have a `molecule/default/` scenario.
- `make test ROLE=base_config` runs `molecule test` for a single role.

This keeps individual network roles easy to validate while proving the collection against realistic routing, VLAN, and backup workflows.

---

## 8. Development and contributions

Repository layout (simplified):

```text
ansible_collections/hybridops/network/
├── roles/
│   ├── base_config/
│   ├── compliance_check/
│   ├── configure_bgp/
│   ├── device_backup/
│   ├── hsrp_vrrp/
│   ├── ntp/
│   ├── ospf/
│   └── vlan/
├── plugins/
└── README.md
```

Standard Ansible collection practices apply:

- Each role has its own `defaults/`, `vars/`, `tasks/`, and `README.md`.  
- Integration tests can be added under `tests/` (for example, Molecule scenarios).  

Release process (high level):

1. Update `galaxy.yml` version and changelog.  
2. Tag the repo (`v0.1.0-network`, `v0.2.0-network`, and so on).  
3. Build and publish with:

   ```bash
   ansible-galaxy collection build
   ansible-galaxy collection publish hybridops-network-<version>.tar.gz
   ```

---

## Releases

This collection is versioned with Semantic Versioning and published to Ansible Galaxy as `hybridops.network`.

Contribution guidelines are documented in `CONTRIBUTING.md` in this repository.

For maintainers, the end-to-end release workflow (versioning, changelog updates, build, publish and evidence capture) follows the standard HybridOps.Studio collections process described in ADR-0606:

- [ADR-0606 – Ansible collections release process](https://docs.hybridops.studio/adr/ADR-0606-ansible-collections-release-process/)

---

## 9. Licence

- Code in this collection: **MIT-0**.  
- Documentation in this repository: **CC BY 4.0**.

See the main HybridOps.Studio repository for project-wide licence details.
