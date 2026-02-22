# Ansible Leaf-Spine Fabric Automation

## Purpose
This repository automates a Cumulus-based leaf-spine fabric with phased deployment and validation:
- Phase 1: interface bring-up + LLDP checks
- Phase 2: loopbacks
- Phase 3: OSPF underlay
- Phase 4: BGP fabric
- Phase 5: server bond configuration
- Phase 6: EVPN/VXLAN services
- Validation phases: 3.5, 4.5, 6.5

The design intent is:
- Single ASN on spines
- Different ASN per leaf
- Leaf-spine topology
- Dual-homed (multihomed) servers via bond/LACP

## Architectural Constraints (Required State)
The playbooks assume these conditions are true.

### Naming and Inventory Contract
- Spines are named `spineN` and live in `[spines]`.
- Leafs are named `leafN` and live in `[leafs]`.
- Servers are named `serverN` and live in `[servers]`.
- `N` must be numeric because loopback/ASN derivation depends on hostname index extraction.

### Port/Topology Contract
- Leaf uplinks to spines are assumed on high ports derived by spine count:
  - leaf side: `swp64`, `swp63`, ... (descending)
- Spine downlinks to leafs are assumed on high ports derived by leaf count:
  - spine side: `swp64`, `swp63`, ... (descending)
- Server-facing connectivity is kept on low-end ports by design.
  - current default access port on each leaf: `leaf_server_port` (default `swp3`)
- Server bond member interfaces are defined per server (default `eth1`, `eth2`).
- `eth0` on servers is treated as management and not changed by the fabric server-bond playbook.

### MLAG/VTEP Derivation Contract
- MLAG pair ID is derived as `ceil(leaf_index / 2)`:
  - `leaf1/leaf2` pair 1, `leaf3/leaf4` pair 2, etc.
- Anycast VTEP is derived from `evpn_vtep_base + mlag_pair_id + evpn_vtep_offset`.

## User Intervention (Manual Inputs)
These are expected operator inputs and are not auto-generated.

### `inventory.ini`
- Device management IPs
- Credentials / connection vars
- Server gateway (`server_bond_gateway`)

### `group_vars/all.yml`
- Fabric prefixes: `leaf_prefix`, `spine_prefix`
- ASN policy: `spine_asn`, `leaf_asn_base`
- EVPN service values (manual-by-design): `evpn_vlan_id`, `evpn_vni`, `evpn_anycast_gw`, `evpn_bridge`

### `group_vars/leafs.yml`
- VTEP derivation knobs: `evpn_vtep_base`, `evpn_vtep_offset`
- Server-facing leaf access port: `leaf_server_port`

### `group_vars/servers.yml`
- Server bond members per host
- Server bond IPs per host
- Bond parameters (`mode`, `lacp-rate`, monitor interval)

## What Scales Automatically
- Device loopback IPs derived from hostname index and shared prefixes.
- Leaf/spine BGP neighbor generation derived from inventory groups.
- EVPN AF activation derived from inventory groups.
- EVPN validation (Phase 6.5) spine/leaf neighbor checks derived from inventory groups (no fixed spine/leaf counts).

## Current Deliberate Constraints
- The BGP multipath command in Phase 4 uses `multipaths ibgp` per current lab requirement.
- Underlay interface derivation is intentionally tied to the porting model above for simplicity.
- Plaintext credentials in inventory are currently accepted for lab use.

## Run Order
```bash
ansible-playbook -i inventory.ini playbooks/Phase1-Interface-Bringup-LLDP.yml
ansible-playbook -i inventory.ini playbooks/Phase2-Loopbacks.yml
ansible-playbook -i inventory.ini playbooks/Phase3-OSPFv2-Underlay.yml
ansible-playbook -i inventory.ini validation-playbooks/Phase3.5-Validation.yml
ansible-playbook -i inventory.ini playbooks/Phase4-eBGP-Fabric.yml
ansible-playbook -i inventory.ini validation-playbooks/Phase4.5-BGP-Validation.yml
ansible-playbook -i inventory.ini playbooks/server-configuration/Phase5-Server-Bonds.yml
ansible-playbook -i inventory.ini playbooks/Phase6-EVPN-VXLAN.yml
ansible-playbook -i inventory.ini validation-playbooks/Phase6.5-EVPN-Validation.yml
```

## Quick Pre-Checks Before Running
- Inventory hostnames follow `spineN`/`leafN`/`serverN`.
- Ports match the topology contract above.
- VLAN/VNI/anycast gateway values are set intentionally.
- Server bond IPs and gateway are correct for the service subnet.
