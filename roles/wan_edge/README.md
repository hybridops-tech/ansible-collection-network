# wan_edge

Legacy Linux WAN compatibility role for strongSwan and FRR.

Use this role only when you need a Linux-based WAN simulator or a transitional strongSwan/FRR edge. The product-standard HybridOps WAN path now lives in `hybridops.common` with the VyOS-backed modules and blueprints.

## What It Does

`wan_edge` configures a Linux host to behave as a route-based WAN node:

- installs strongSwan and FRR
- creates VTI interfaces for each tunnel
- loads `swanctl` IPsec configuration
- enables FRR eBGP peering over the tunnel inside addresses
- assigns loopbacks and optional alias IPs used by the simulator topology

## Requirements

- Debian 11+ or Ubuntu 22.04+
- root or sudo access
- management connectivity to the target host
- a dedicated underlay interface for the WAN-facing IPs

## Variable Contract

The public contract is role-prefixed.

### Required

| Variable | Description |
|----------|-------------|
| `wan_edge_public_local_ip` | Primary underlay IP for the local node |
| `wan_edge_public_peer_ip` | Default underlay peer IP |
| `wan_edge_tunnels` | Tunnel definitions |
| `wan_edge_ipsec.psk` | Default IKE pre-shared key |
| `wan_edge_bgp.local_as` | Local BGP ASN |
| `wan_edge_bgp.router_id` | Local BGP router ID |
| `wan_edge_bgp.advertise` | Prefixes to advertise |
| `wan_edge_bgp.import_allow` | Prefixes expected from the peer |
| `wan_edge_bgp.export_allow` | Prefixes permitted for export |

### Tunnel Definition

```yaml
wan_edge_tunnels:
  - name: tunnel_a
    ifname: vti10
    key: 10
    mark: 10
    inside_local: "169.254.10.1/30"
    inside_peer: "169.254.10.2"
    peer_public_ip: "203.0.113.1"
    local_public_ip: "198.51.100.1"
```

### Common Optional Inputs

| Variable | Default | Description |
|----------|---------|-------------|
| `wan_edge_pub_if` | auto-detected | Underlay interface carrying the public IPs |
| `wan_edge_public_aliases` | `[]` | Additional underlay IPs for multi-tunnel peers |
| `wan_edge_loopbacks` | `[]` | Routed loopback prefixes used for reachability checks |
| `wan_edge_bgp.peer_groups` | `[]` | Hub/transit peer-group definitions |
| `wan_edge_bgp_transit` | `false` | Enables hub-style transit behavior |
| `wan_edge_wait_for_tunnels` | `false` | Waits for CHILD_SAs before role exit |

## Example

```yaml
- name: Configure WAN edge
  hosts: edge
  roles:
    - role: hybridops.network.wan_edge
```

Example group variables:

```yaml
wan_edge_pub_if: "eth1"
wan_edge_public_local_ip: "198.51.100.10"
wan_edge_public_peer_ip: "203.0.113.10"

wan_edge_loopbacks:
  - "10.110.0.1/24"

wan_edge_tunnels:
  - name: tunnel_a
    ifname: vti10
    key: 10
    mark: 10
    inside_local: "169.254.10.1/30"
    inside_peer: "169.254.10.2"

  - name: tunnel_b
    ifname: vti11
    key: 11
    mark: 11
    inside_local: "169.254.10.5/30"
    inside_peer: "169.254.10.6"
    peer_public_ip: "203.0.113.11"

wan_edge_ipsec:
  psk: "{{ vault_wan_edge_ipsec_psk }}"
  start_action: "start"
  proposals: "aes256-sha256-modp2048"
  esp_proposals: "aes256-sha256"

wan_edge_bgp:
  local_as: 65010
  peer_as: 64514
  router_id: "10.110.0.1"
  neighbors:
    - "169.254.10.2"
    - "169.254.10.6"
  advertise:
    - "10.110.0.0/24"
  import_allow:
    - "10.70.0.0/20"
  export_allow:
    - "10.110.0.0/24"
```

## Testing

This role ships with a repeatable three-node simulator:

- `edge-sim` as the on-prem spoke
- `gcp-sim` as the transit hub
- `azure-sim` as the second spoke

Provision the simulator VMs through HyOps:

```bash
hyops apply --env quickTest \
  --module platform/onprem/platform-vm \
  --state-instance wan_sim_vms \
  --inputs roles/wan_edge/tests/fixtures/wan_sim_vms.quicktest.inputs.yml
```

Then run the full role smoke:

```bash
hyops test role hybridops.network.wan_edge \
  --env quickTest \
  --workspace-root /path/to/hybridops-collections-src/collections/dev/workspace \
  --inventory-file /path/to/ansible-collection-network/roles/wan_edge/tests/inventories/wansim/hosts.ini
```

That smoke includes `hybridops.network.wan_validate`, so it proves both configuration and convergence.

Troubleshooting reference:

- [checks.md](tests/inventories/wansim/checks.md)

## License

- Code: [MIT-0](https://spdx.org/licenses/MIT-0.html)
- Documentation: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)
