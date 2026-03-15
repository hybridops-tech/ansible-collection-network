# wan_validate

Read-only convergence checks for `wan_edge`.

`wan_validate` proves that the Linux WAN simulator has actually converged. It does not configure services. Use it after `hybridops.network.wan_edge` or as a standalone health check on an already-configured node.

## What It Checks

- expected IPsec CHILD_SAs are installed
- no CHILD_SAs are stuck in transient states
- BGP neighbors are reachable and Established
- expected routes are present in the BGP table
- expected routes are present in the kernel FIB when requested
- optional end-to-end reachability checks succeed from the routed loopback source

## Inputs

`wan_validate` consumes the `wan_edge_*` topology variables from `wan_edge` and adds its own controls.

| Variable | Default | Description |
|----------|---------|-------------|
| `wan_validate_expect_routes` | derived from `wan_edge_bgp.import_allow` | Prefixes expected in the BGP table |
| `wan_validate_expect_bgp_neighbors` | derived from `wan_edge_bgp` | Explicit neighbor list override |
| `wan_validate_ping_targets` | `[]` | IPs to ping after convergence |
| `wan_validate_ping_source` | first `wan_edge_loopbacks` address | Explicit source IP override |
| `wan_validate_ping_count` | `3` | Ping attempts per target |
| `wan_validate_assert_kernel_routes` | `false` | Also verify routes in the kernel routing table |
| `wan_validate_require_accepted_prefixes` | `false` | Require each neighbor to report accepted prefixes |

## Example

```yaml
- name: Validate WAN tunnels
  hosts: edge
  roles:
    - role: hybridops.network.wan_validate
```

Example validation overrides:

```yaml
wan_validate_ping_targets:
  - "10.70.0.1"
  - "10.80.0.1"

wan_validate_assert_kernel_routes: true
```

If you want the committed simulator topology to carry the reachability targets,
set `wan_edge_validation_ping_targets` beside the `wan_edge_*` variables and let
`wan_validate` derive from it.

## Role Pairing

The normal maintainer path is:

```yaml
- name: Configure WAN
  hosts: wansim_gcp:wansim_edge:wansim_azure
  roles:
    - role: hybridops.network.wan_edge

- name: Validate WAN
  hosts: wansim
  roles:
    - role: hybridops.network.wan_validate
```

The committed `wan_edge` smoke already follows that pattern, so a successful `wan_edge` smoke run proves this role too.

For a standalone `wan_validate` run, reuse the committed WAN simulator inventory from
`roles/wan_edge/tests/inventories/wansim/` rather than maintaining a second copy.

## License

- Code: [MIT-0](https://spdx.org/licenses/MIT-0.html)
- Documentation: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)
