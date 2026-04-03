# `hybridops.network`

Linux WAN simulation and validation roles for lab and brownfield environments.

This collection is intentionally narrow. The primary WAN and site-extension path uses VyOS-backed roles and modules in `hybridops.common` and `hybridops-core`. `hybridops.network` is useful for:

- three-node WAN simulation
- academy and workshop lab exercises
- brownfield Linux strongSwan/FRR estates that are not yet on the VyOS path

Public docs live at <https://docs.hybridops.tech>. The main website is <https://hybridops.tech>.

## Roles

| Role | Purpose |
|------|---------|
| `wan_edge` | Configure a Linux WAN node with strongSwan, VTI interfaces, and FRR eBGP. |
| `wan_validate` | Validate WAN convergence across IPsec, BGP, routes, and end-to-end reachability. |

## Installation

Install from Ansible Galaxy:

```bash
ansible-galaxy collection install hybridops.network
```

Pin from a `collections/requirements.yml` file:

```yaml
collections:
  - name: hybridops.network
    version: ">=0.1.0"
```

## Testing

Run role tests with HybridOps:

```bash
hyops test role hybridops.network.wan_edge \
  --env quickTest \
  --workspace-root /path/to/hybridops-collections-src/collections/dev/workspace \
  --inventory-file /path/to/ansible-collection-network/roles/wan_edge/tests/inventories/wansim/hosts.ini
```

To stand up the three-VM WAN simulator first:

```bash
hyops apply --env quickTest \
  --module platform/onprem/platform-vm \
  --state-instance wan_sim_vms \
  --inputs roles/wan_edge/tests/fixtures/wan_sim_vms.quicktest.inputs.yml
```

That fixture provisions:

- `edge-sim` on `10.10.0.132` with WAN underlay `10.50.0.142`
- `gcp-sim` on `10.10.0.151` with WAN underlay `10.50.0.149`
- `azure-sim` on `10.10.0.181` with WAN underlay `10.50.0.132`

`wan_edge` smoke includes `wan_validate`, so the full role test proves both configuration and convergence.

## Scope boundary

This collection covers Linux-based WAN simulation and brownfield paths. For the HybridOps WAN stack, use the VyOS roles and modules from `hybridops.common` and `hybridops-core`.

## License

- Code: [MIT-0](https://spdx.org/licenses/MIT-0.html)
- Documentation: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)
