# Open tickets

These are open lab/NOC-drill tickets. They're listed so the project is honest about what's proven. None of them is claimed as a result.

| Ticket | Problem (last recorded) | Next action | Status |
|---|---|---|---|
| TKT-2105 | Finance VLAN 20 HSRP failover not yet proven | Run T15–T17; capture `show standby brief` on both switches before/after | Open |
| TKT-2109 | VLAN 60: L3SW2 Vlan60 owns 10.10.60.1 as its real IP, no `standby 60` | L3SW2 `ip address 10.10.60.3 255.255.255.0` + `standby 60 ip 10.10.60.1`, priority 100, preempt; re-run T04 | Open |
| TKT-2111 | Helpers mainly on L3SW1; DHCP during Active loss unproven | Helpers on every L3SW2 SVI + L3SW2 default; T16 | Open |
| TKT-2113 | VLAN 80 hosts get a lease but can't use VIP 10.10.80.1 (default-router / VIP mismatch) | Pool VLAN80_SECURITY `default-router 10.10.80.1`; check group 80; T05 on VLAN 80 | Open |
| TKT-2114 | Bank PCs can't reach Server0 172.16.10.10 (Dist had no outside route) | Dist defaults applied in the design; prove with T11 (ping) and T12 (tracert) | Diagnosed, proof pending |
| TKT-2115 | HQ reachable only while L3SW1 is Active; L3SW2 has no working outside route | L3SW2 `ip route 0.0.0.0 0.0.0.0 10.255.255.1`; T15 with HQ ping; also check the return path while L3SW1 Vlan20 is shut | Open |

Other clean-up from the 16 Sep live extract (no ticket): VLAN 30 pool missing `dns-server`, duplicate `vlan40_IT` pool, Router0 Gi0/0/1 description says L3SW2 (peer is L3SW1), DNS record `inno@bank.com` needs renaming.
