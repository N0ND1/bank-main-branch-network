# LESSON-01: `%Invalid next hop address (it's this router)`

**Type:** lesson learned while building the static routes (not an incident)

## What happened
Adding the default route on a distribution switch returned:

```
%Invalid next hop address (it's this router)
```

## Why
The next hop was an address the switch owns itself (its own SVI .2/.3, or 10.10.255.2 on L3SW1). IOS won't route to itself.

## Fix
Point at a neighbour on a connected subnet:

```ios
! L3SW1 - connected to Router0 on 10.10.255.0/30
ip route 0.0.0.0 0.0.0.0 10.10.255.1

! L3SW2 - no link in 10.10.255.0/30, so go via L3SW1's Po1 address
ip route 0.0.0.0 0.0.0.0 10.255.255.1
```

## Proof
`show ip route` should show `S* 0.0.0.0/0 via 10.10.255.1` and gateway of last resort 10.10.255.1 on L3SW1. **Capture still pending.**
