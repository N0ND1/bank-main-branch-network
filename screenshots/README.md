# Screenshots

Live Packet Tracer CLI captures from the 16 Sep 2026 lab file.

| File | Shows | Used for |
|---|---|---|
| `01-L3SW1-hsrp-svis-2026-09-16.webp` | L3SW1: HSRP state changes to Active for groups 10–100; `show ip interface brief \| include Vlan` with Vlan10–100 .2 up/up | INC-2101 |
| `02-L3SW2-svis-po1-2026-09-16.webp` | L3SW2: Port-channel1 10.255.255.2 up; SVIs .3 up/up. **Note:** Vlan60 shows 10.10.60.1, the open TKT-2109 gap | INC-2101 pair · known issue 1 |
| `03-Router0-dhcp-bindings-2026-09-16.webp` | Router0 `show ip dhcp binding`: leases across VLANs incl. Finance 10.10.20.x | TKT-2104 |
| `04-dns-server-records-2026-09-16.webp` | Branch DNS server 10.10.90.10: DNS service on, A records | Supporting (DNS) |

MAC addresses in the captures are Packet Tracer simulated devices.
