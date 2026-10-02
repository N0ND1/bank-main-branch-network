# INC-2101: Finance VLAN 20, Limited/No connectivity (HSRP gateway)

**Type:** lab incident (Packet Tracer NOC drill) · **Status:** Resolved · **Owner:** Innocent (Nkosiyethu) Mbatha

## Fault
Finance floor PCs (VLAN 20) showed Limited/No connectivity and the banking apps were down. **HR next door (VLAN 30) still worked**, so this wasn't a campus-wide outage.

## Diagnose
1. Compared a broken Finance host's IP, mask, gateway and DNS with a working HR host. Same pattern, different VLAN.
2. Because HR was healthy and both VLANs share the same access and distribution switches, I checked the distribution layer for VLAN 20 before touching access ports: SVI up/up, `show standby brief`, VIP present.
3. Confirmed clients must use the VIP `10.10.20.1`, not a physical SVI address (.2 or .3).

**Cause (as recorded in the master document):** HSRP group 20 was stuck in Init. Both distribution switches had priority 110, so there was no deterministic Active, and the VLAN 20 hellos weren't reaching the peer because the trunks weren't carrying VLAN 20.

## Fix
```ios
! L3SW1 - Active
interface Vlan20
 ip address 10.10.20.2 255.255.255.0
 standby 20 ip 10.10.20.1
 standby 20 priority 110
 standby 20 preempt

! L3SW2 - Standby
interface Vlan20
 ip address 10.10.20.3 255.255.255.0
 standby 20 ip 10.10.20.1
 standby 20 priority 100
 standby 20 preempt
```
Trunks restored with native VLAN 99 and VLAN 20 in the allowed list.

## Verify
- `show standby brief`: group 20 Active on L3SW1, Standby on L3SW2
- Finance PC pings `10.10.20.1`
- HR still healthy (cross-VLAN ping)

Evidence: [`../screenshots/01-L3SW1-hsrp-svis-2026-09-16.webp`](../screenshots/01-L3SW1-hsrp-svis-2026-09-16.webp) (HSRP state changes to Active, SVIs up/up) and [`../screenshots/02-L3SW2-svis-po1-2026-09-16.webp`](../screenshots/02-L3SW2-svis-po1-2026-09-16.webp).

## Not covered by this ticket
Failover (shut the Active and watch L3SW2 take over) hasn't been tested yet. That's **TKT-2105**, still open. See [open-tickets.md](open-tickets.md).
