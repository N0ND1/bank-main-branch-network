# Configs

`target-state/` holds one paste-ready file per device, taken from Appendix A of the [Master Document](../docs/Bank_Main_Branch_Network_Design_Master.pdf) (Rev 1.0, 30 Sep 2026).

- These are the **target design**, not `show running-config` exports from the live lab. Where the live lab differs, see [Known issues](../README.md#known-issues).
- Lines marked `! VERIFY` use interface IDs that weren't confirmed in a live extract.
- Comments sit on their own lines because IOS rejects inline `!` comments.
- No passwords, enable secrets, SNMP communities or keys are in these files. The target design doesn't set any (the .pkt hasn't been checked; see the publishing notes). If you add them in your own copy, don't commit them.

| File | Device | Built by |
|---|---|---|
| `Router0.txt` | Edge router + DHCP server | Innocent Mbatha |
| `Router1.txt`, `Bank-Edge.txt`, `ISP.txt`, `HQ-Edge.txt` | WAN path to HQ | Innocent Mbatha |
| `L3SW1.txt`, `L3SW2.txt` | Distribution pair | Nondumiso Mbuyazi (VLANs, trunks, SVIs, Po1) + Innocent Mbatha (HSRP, helpers, routes) |
| `ASW1.txt` … `ASW6.txt` | Access switches | Nondumiso Mbuyazi |
| `servers-and-hq-switch.txt` | Server0, Switch0, branch DNS server (GUI settings) | Innocent Mbatha |
