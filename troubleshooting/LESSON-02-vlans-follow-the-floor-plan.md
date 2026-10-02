# LESSON-02: VLANs follow the floor plan

**Type:** lesson learned during the design (not an incident)
**Joint lesson:** Nondumiso Mbuyazi and Innocent (Nkosiyethu) Mbatha. The access ports and floor-plan integration are Nondumiso's part of the build ([Master Document](../docs/Bank_Main_Branch_Network_Design_Master.pdf) §1.4).

## Symptom / what we got wrong
Our first topology had every VLAN listed and configured on every access switch, and we hadn't looked at the floor plan. Each department ended up spread over whichever switches had free ports, with no link to the room it was in or the wiring closet near it.

The earlier port table shows this. Nondumiso's [command book](../docs/L2-L3-Command-Book_Nondumiso-Mbuyazi.pdf) (p.4, "Differences vs earlier port table") summarises it:

| Switch | Before (earlier port table) |
|---|---|
| ASW1 | 50 (Fa0/1–8), 80 (Fa0/9–11), 60 (Fa0/12) |
| ASW2 | 60 (Fa0/1–6), 20 (Fa0/7–10), 70 laptops |
| ASW3 | 10, 30, 20, 70 (ports not recorded) |
| ASW4 | 20 + IP Phone1, 30, 70 |
| ASW5 | 80 (PC42–46), 20 laptops |
| ASW6 | 90 servers, 80 / 40 |
| VLAN 100 | All APs on a seventh switch, ASW7 |

Finance (VLAN 20) alone sat on four switches: ASW2, ASW3, ASW4 and ASW5. That table has no room or IDF column at all.

## Why it mattered
This is design reasoning, not a measured result:

- **Cabling to the wrong closet.** The floor plan puts Finance (Room 5) beside IDF A ([floor plan](../diagrams/floor-plan-ground.png); Master Document §2.3). With the switches in the IDFs as drawn, the old map would cable Finance desks to switches in other closets, or give a desk a port whose VLAN doesn't match its room.
- **Troubleshooting has to start from a room.** A caller says "Finance is down", not "ASW3 Fa0/7". With the old map, room → closet → switch → port → VLAN had no fixed answer. Now it does: Room 5 → IDF A → ASW1 Fa0/1–3 / ASW2 Fa0/1–3 → VLAN 20 → gateway 10.10.20.1.
- **A closet fault should match a known set of rooms.** Each IDF now serves set rooms (Master Document §2.3). Under the old map, one switch carried bits of several unrelated departments.
- **Trunk scope.** If each closet only serves certain VLANs, its trunks only need those VLANs (see "Not changed yet" below).

## What we changed
We didn't rebuild the whole network. We remapped the access layer so each department's VLAN sits on the access switches in the IDF that serves its area of the floor plan.

After, by room. Rooms, VLANs and IDFs are from Master Document §2.2–2.3 (p.7). Ports are from Master Document §7.2 (p.13), which matches [`configs/target-state/ASW1–6`](../configs/target-state/).

| Room | Department | VLAN | IDF | Access switch ports |
|---|---|---|---|---|
| 2 | Customer Service | 60 | IDF A | ASW1 Fa0/15–17 · ASW2 Fa0/13–15 |
| 3 | Management Office | 10 | IDF C | ASW5 Fa0/6–9 · ASW6 Fa0/1–3 |
| 4 | Tellers | 50 | IDF C | ASW5 Fa0/1–5 · ASW6 Fa0/4–5 |
| 5 | Finance | 20 | IDF A | ASW1 Fa0/1–3 · ASW2 Fa0/1–3 |
| 6 | HR | 30 | IDF A | ASW1 Fa0/4–5 · ASW2 Fa0/4–6 |
| 7 | Loans | 70 | IDF B | ASW3 Fa0/1–4 · ASW4 Fa0/1–3 |
| 8 | IT | 40 | IDF A + IDF B | ASW1 Fa0/6–9 · ASW2 Fa0/7–9 · ASW3 Fa0/5–6 · ASW4 Fa0/4–6 |
| 9 + 14 | Security Office + Security Control Room | 80 | IDF C | ASW5 Fa0/10–11, Fa0/14–15 · ASW6 Fa0/6–8 |
| 12 + 13 | Server Room (MDF) + Storage / Records | 90 | IDF C | ASW5 Fa0/12–13 · ASW6 Fa0/9–11 |
| Guest Wi-Fi (APs) | – | 100 | IDF A + IDF C | ASW2 Fa0/10–12 · ASW6 Fa0/12–13 |
| 1, 10, 11, 15 | Public Lobby, Staff Room, Network Room, ATM Lobby | none yet | – | Open items V7 / V9 (Master Document §2.3) |

Before → after, by department. "Before" is from the command book p.4; "after" is from the table above.

| VLAN | Department | Before: switches | After: switches (IDF) |
|---|---|---|---|
| 10 | Management | ASW3 | ASW5, ASW6 (C) |
| 20 | Finance | ASW2, ASW3, ASW4, ASW5 | ASW1, ASW2 (A) |
| 30 | HR | ASW3, ASW4 | ASW1, ASW2 (A) |
| 40 | IT | ASW6 | ASW1, ASW2 (A) + ASW3, ASW4 (B) |
| 50 | Tellers | ASW1 | ASW5, ASW6 (C) |
| 60 | Customer Service | ASW1, ASW2 | ASW1, ASW2 (A) |
| 70 | Loans | ASW2, ASW3, ASW4 | ASW3, ASW4 (B) |
| 80 | Security | ASW1, ASW5, ASW6 | ASW5, ASW6 (C) |
| 90 | Storage / Servers | ASW6 | ASW5, ASW6 (C) |
| 100 | Guest / APs | ASW7 | ASW2 (A), ASW6 (C) |

Each IDF holds one switch pair, and every access switch is dual-homed (Gi0/1 to L3SW1, Gi0/2 to L3SW2). So a department in one closet is split across two switches, not across the building.

### Not changed yet
- **Every access switch still has the full VLAN database, and every uplink trunk still allows all of them** (`switchport trunk allowed vlan 10,20,30,40,50,60,70,80,90,99,100` in every ASW config). So the remap changed where users plug in, not which VLANs reach each closet. A VLAN's broadcasts still flood to every access switch.
- The next step is pruning each closet's trunks to the VLANs it serves:
  - IDF A: 20, 30, 40, 60, 100
  - IDF B: 40, 70
  - IDF C: 10, 50, 80, 90, 100

  Check this before applying it. Po1 is routed, so the HSRP hellos between L3SW1 and L3SW2 cross the access trunks (Master Document §8.1). Each VLAN has to stay on the trunks of at least one IDF.

## Lesson
Look at the floor plan before configuring the access layer. Where people sit decides which closet serves them, and that decides which switch and VLAN their port belongs to. Many lab builds put every VLAN on whichever switch has a free port. We picked the switch that physically serves each area and configured that department's VLAN there.

## How to verify
On each access switch (ASW1–ASW6):

```ios
show vlan brief
show interfaces trunk
show cdp neighbors
```

- `show vlan brief`: all VLANs (10–100 and 99) will be listed, because the database is on every switch. Check the **Ports** column: each port range should be in the VLAN shown in the table above. For example, on ASW1 Fa0/1–3 should be in VLAN 20 and Fa0/15–17 in VLAN 60.
- `show interfaces trunk`: Gi0/1 and Gi0/2 trunking, native VLAN 99. Today the allowed list is the full set on every switch. After pruning it should match that switch's IDF.
- `show cdp neighbors`: Gi0/1 should reach L3SW1 and Gi0/2 should reach L3SW2, on Gi1/0/3 (ASW1) through Gi1/0/8 (ASW6). That confirms each switch is cabled into the closet it's documented in.
- From one PC per room: DHCP gives an address in that room's subnet (test plan T05/T06).

## Proof
These are tests T01 (trunks) and T03 (VLANs / access ports) in the [test plan](../docs/test-plan.md). **Not run yet; captures pending.** The command book (p.10) also notes that the running `.pkt` still needs checking against these port assignments.
