# TKT-2104: VIP reachable, but no DHCP leases (relay path)

**Type:** lab ticket (Packet Tracer NOC drill) · **Status:** Resolved · **Owner:** Innocent (Nkosiyethu) Mbatha

## Fault
After INC-2101 brought the Finance gateway back, Finance (e.g. PC19) and Tellers (VLAN 50) clients stayed on APIPA or never got a lease. The pattern: **the VIP answers pings, but DHCP Discover never becomes an Offer.**

## Diagnose (in this order)
1. From the distribution switch: `ping 10.10.255.1` (Router0 edge). Prove the L3 path to the DHCP server before rewriting any pools.
2. `show run interface Vlan20`: is `ip helper-address 10.10.255.1` there? (On both Dist switches, since either can be Active.)
3. On Router0: does the pool exist, and is `default-router` the VIP `.1` (not `.2`/`.3`)?
4. Only after path + helper + pool: renew the client and check `show ip dhcp binding`.

**Cause:** no relay path to the DHCP server. The helpers were missing on the Dist SVIs, and the Dist needed the edge /30 to reach Router0.

## Fix
- Edge /30 between Router0 and **L3SW1**: Router0 Gi0/0/1 `10.10.255.1` ↔ L3SW1 Gi1/0/1 `10.10.255.2`.
- `ip helper-address 10.10.255.1` on the Dist SVIs.
- One Router0 pool per VLAN, VIP as `default-router`, `.1–.10` excluded.

```ios
! Dist SVI (pattern)
interface Vlan20
 ip helper-address 10.10.255.1

! Router0 pool (pattern)
ip dhcp excluded-address 10.10.20.1 10.10.20.10
ip dhcp pool VLAN20_FINANCE
 network 10.10.20.0 255.255.255.0
 default-router 10.10.20.1
```

## Verify
- Dist reaches `10.10.255.1`
- Client gets IP, mask, gateway = VIP, DNS
- Router0 `show ip dhcp binding` shows the client's MAC

Evidence: [`../screenshots/03-Router0-dhcp-bindings-2026-09-16.webp`](../screenshots/03-Router0-dhcp-bindings-2026-09-16.webp) (bindings across VLANs incl. Finance 10.10.20.x). Path diagram: [`../diagrams/dhcp-relay-path.png`](../diagrams/dhcp-relay-path.png).

## Not covered by this ticket
DHCP while L3SW2 is Active is unproven (helpers mainly on L3SW1). That's **TKT-2111**, still open.
