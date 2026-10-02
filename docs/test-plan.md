# Test plan (T01–T18)

From the Master Document §9. **Results are blank on purpose**: a result goes in only after the test has been run on the live `.pkt`, with a screenshot reference. Any difference from the expected result is a finding.

| # | Area | Test / command | Expected result | Result / screenshot |
|---|---|---|---|---|
| T01 | Trunks | `show interfaces trunk` (ASW1–6, L3SW1/2) | ASW Gi0/1–2 and Dist Gi1/0/3–8 trunking, native 99, same allowed list both ends | |
| T02 | Po1 | `show etherchannel summary` (L3SW1/2) | Po1(RU), LACP, Gi1/0/23 and Gi1/0/24 (P); ping 10.255.255.2 from L3SW1 replies | |
| T03 | VLANs / access | `show vlan brief` (all switches) | VLANs 10–100 + 99 with the agreed names; each access range in its VLAN | |
| T04 | HSRP state | `show standby brief` (L3SW1, L3SW2) | 10 groups; L3SW1 Active pri 110 P; L3SW2 Standby pri 100 P; VIP .1; no Vlan99 | |
| T05 | DHCP lease | Finance PC → IP Configuration → DHCP | 10.10.20.11–.254, /24, GW 10.10.20.1, DNS 10.10.90.10 | |
| T06 | DHCP leases (all VLANs) | One host per VLAN 10–100 → DHCP; Router0 `show ip dhcp binding` | A binding in every pool; none inside an excluded range | |
| T07 | Gateway | Finance PC `ping 10.10.20.1` | Replies from the VIP | |
| T08 | Inter-VLAN | Finance PC ping an HR host (VLAN 30) | Replies, routed on the Active Dist | |
| T09 | Edge | Finance PC `ping 10.10.255.1` | Replies from Router0 | |
| T10 | Routes | `show ip route` on each hop | Entries exactly as in the README routing table; next hops on connected /30s | |
| T11 | HQ reachability | Finance PC `ping 172.16.10.10` | Replies from Server0 | |
| T12 | Path | Finance PC `tracert 172.16.10.10` | 10.10.20.2 → 10.10.255.1 → 10.10.255.6 → 10.255.1.2 → 10.255.2.2 → 10.255.3.2 → 172.16.10.10 | |
| T13 | DNS | PC `nslookup router0` | Server 10.10.90.10 answers 10.10.255.1 | |
| T14 | HQ web | PC Web Browser `http://172.16.10.10` | Server0 page loads | |
| T15 | HSRP failover | PC `ping -t 10.10.20.1` + `ping -t 172.16.10.10`; L3SW1 `interface Vlan20` → `shutdown` | L3SW2 Active for group 20 after ≈10 s; VIP pings resume; HQ pings resume via L3SW2 → Po1 → L3SW1 (TKT-2115) | |
| T16 | DHCP in failover | During T15: renew a Finance PC | New lease via the L3SW2 helper (TKT-2111) | |
| T17 | Preempt | L3SW1 `interface Vlan20` → `no shutdown` | L3SW1 back to Active (110, preempt); L3SW2 Standby; pings continue | |
| T18 | BPDU Guard | Connect a switch to an ASW Fa0/x port | Port goes err-disabled; recover with `shutdown` / `no shutdown` | |
