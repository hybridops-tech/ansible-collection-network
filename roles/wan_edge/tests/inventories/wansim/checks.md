# WAN Simulator Troubleshooting

Quick reference for the three-node `wan_edge` simulator.

## Topology

| Role | Hostname | Mgmt IP | Underlay IP | Loopback | ASN |
|------|----------|---------|-------------|----------|-----|
| On-prem spoke | `edge-sim` | `10.10.0.132` | `10.50.0.142` | `10.110.0.1/24` | `65010` |
| Transit hub | `gcp-sim` | `10.10.0.151` | `10.50.0.149` | `10.70.0.1/20` | `64514` |
| Azure spoke | `azure-sim` | `10.10.0.181` | `10.50.0.132` | `10.80.0.1/24` | `65020` |

Additional alias IPs:

- `gcp-sim`: `10.50.0.150`, `10.50.0.151`, `10.50.0.152`
- `azure-sim`: `10.50.0.133`

## Provisioning Path

The committed HyOps VM fixture is:

- [wan_sim_vms.quicktest.inputs.yml](../../fixtures/wan_sim_vms.quicktest.inputs.yml)

Apply it with:

```bash
hyops apply --env quickTest \
  --module platform/onprem/platform-vm \
  --state-instance wan_sim_vms \
  --inputs roles/wan_edge/tests/fixtures/wan_sim_vms.quicktest.inputs.yml
```

## Basic Checks

Confirm the underlay interface addresses:

```bash
ssh opsadmin@10.10.0.132 'ip -4 addr show eth1 | grep inet'
ssh opsadmin@10.10.0.151 'ip -4 addr show eth1 | grep inet'
ssh opsadmin@10.10.0.181 'ip -4 addr show eth1 | grep inet'
```

Expected primary underlay addresses:

- `edge-sim`: `10.50.0.142/24`
- `gcp-sim`: `10.50.0.149/24`
- `azure-sim`: `10.50.0.132/24`

## Tunnel Plan

### edge-sim -> gcp-sim

| Tunnel | Local IP | Peer IP | Inside Local | Inside Peer | Key |
|--------|----------|---------|--------------|-------------|-----|
| `tunnel_a` | `10.50.0.142` | `10.50.0.149` | `169.254.10.1/30` | `169.254.10.2` | `10` |
| `tunnel_b` | `10.50.0.142` | `10.50.0.150` | `169.254.10.5/30` | `169.254.10.6` | `11` |

### gcp-sim -> azure-sim

| Tunnel | Local IP | Peer IP | Inside Local | Inside Peer | Key |
|--------|----------|---------|--------------|-------------|-----|
| `azure_a` | `10.50.0.151` | `10.50.0.132` | `169.254.20.1/30` | `169.254.20.2` | `20` |
| `azure_b` | `10.50.0.152` | `10.50.0.133` | `169.254.20.5/30` | `169.254.20.6` | `21` |

## Operational Commands

### IPsec

```bash
swanctl --list-sas
swanctl --list-sas | grep -c "INSTALLED, TUNNEL"
```

### BGP

```bash
vtysh -c "show bgp summary"
vtysh -c "show bgp ipv4 unicast"
vtysh -c "show ip route 10.110.0.0/24"
```

### Routing

```bash
ip route show proto bgp
ip route get 10.110.0.1
ip route show table all | grep 169.254
```

### Reachability

```bash
ping -c 2 -I 10.80.0.1 10.110.0.1
ping -c 2 -I 10.110.0.1 10.70.0.1
```

## Common Failures

| Symptom | Check | Fix |
|---------|-------|-----|
| `bgpd` not running | `grep bgpd /etc/frr/daemons` | Re-run role or set `bgpd=yes` and restart FRR |
| Tunnels stay in `CONNECTING` | `swanctl --list-sas` | Check PSK and underlay peer IPs |
| Transit routing missing | `vtysh -c "show ip route <prefix>"` | Ensure `wan_edge_bgp_transit` and `wan_edge_bgp.import_allow` match the topology |
| Ping succeeds only from underlay | Run ping with `-I <loopback>` | Traffic selectors are built around the routed loopbacks, not the VTI inside IPs |
