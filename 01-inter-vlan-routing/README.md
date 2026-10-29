# Lab 01: Inter-VLAN Routing (SVI + Router-on-a-Stick)

## Objective

Route traffic between four VLANs using **both** inter-VLAN routing methods in one topology, then prove the forwarding path with routing, CEF, and adjacency tables.

- VLAN 10 and 20 are routed by a **Layer 3 switch (SVIs)**
- VLAN 30 and 40 are routed by a **router-on-a-stick (802.1Q sub-interfaces)**
- The two routing points are connected through a transit VLAN (99) with static routes

---

## Topology

![Topology](./topology.png)

```
                 R1 (IOSv)  ── router-on-a-stick for VLAN 30, 40
                 Gi0/0
                   │  trunk (30, 40, 99)
                 Gi0/0
                 DSW1 (IOSvL2) ── SVIs for VLAN 10, 20, 99
            Gi0/1 ╱        ╲ Gi0/2
         trunk   ╱          ╲   trunk (10, 20, 30, 40)
           Gi0/0           Gi0/0
           ASW1             ASW2
       Gi0/1  Gi0/2     Gi0/1  Gi0/2
         │      │         │      │
        PC1    PC2       PC3    PC4
      VLAN10  VLAN30   VLAN20  VLAN40
```

| Device | Role | Image |
|--------|------|-------|
| R1 | Router-on-a-stick | Cisco IOSv (_version_) |
| DSW1 | Distribution / Layer 3 switch | Cisco IOSvL2 (_version_) |
| ASW1, ASW2 | Access switches | Cisco IOSvL2 (_version_) |
| PC1–PC4 | Hosts | VPCS |

---

## Addressing Plan

### VLANs

| VLAN | Name | Subnet | Gateway | Routed by |
|------|------|--------|---------|-----------|
| 10 | STAFF | 192.168.10.0/24 | 192.168.10.1 | DSW1 (SVI) |
| 20 | SERVER | 192.168.20.0/24 | 192.168.20.1 | DSW1 (SVI) |
| 30 | GUEST | 192.168.30.0/24 | 192.168.30.1 | R1 (sub-interface) |
| 40 | VOICE | 192.168.40.0/24 | 192.168.40.1 | R1 (sub-interface) |
| 99 | TRANSIT | 10.0.99.0/30 | n/a | DSW1 .1 ↔ R1 .2 |
| 999 | NATIVE-UNUSED | n/a | n/a | Native VLAN on trunks |

### Hosts

| Host | Switch port | VLAN | IP address | Gateway |
|------|-------------|------|------------|---------|
| PC1 | ASW1 Gi0/1 | 10 | 192.168.10.10/24 | 192.168.10.1 |
| PC2 | ASW1 Gi0/2 | 30 | 192.168.30.10/24 | 192.168.30.1 |
| PC3 | ASW2 Gi0/1 | 20 | 192.168.20.10/24 | 192.168.20.1 |
| PC4 | ASW2 Gi0/2 | 40 | 192.168.40.10/24 | 192.168.40.1 |

---

## Key Configuration

Full running configs are in [`configs/`](./configs).

### DSW1 (Layer 3 switch)

```
hostname DSW1
ip routing
!
vlan 10
 name STAFF
vlan 20
 name SERVER
vlan 30
 name GUEST
vlan 40
 name VOICE
vlan 99
 name TRANSIT
vlan 999
 name NATIVE-UNUSED
!
interface GigabitEthernet0/0
 description TRUNK-TO-R1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 30,40,99
!
interface GigabitEthernet0/1
 description TRUNK-TO-ASW1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,20,30,40
!
interface GigabitEthernet0/2
 description TRUNK-TO-ASW2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,20,30,40
!
interface Vlan10
 ip address 192.168.10.1 255.255.255.0
 no shutdown
interface Vlan20
 ip address 192.168.20.1 255.255.255.0
 no shutdown
interface Vlan99
 ip address 10.0.99.1 255.255.255.252
 no shutdown
!
ip route 192.168.30.0 255.255.255.0 10.0.99.2
ip route 192.168.40.0 255.255.255.0 10.0.99.2
```

> No SVIs exist for VLAN 30 and 40 on DSW1. DSW1 only switches those frames at Layer 2 toward R1.

### R1 (router-on-a-stick)

```
hostname R1
!
interface GigabitEthernet0/0
 no ip address
 no shutdown
!
interface GigabitEthernet0/0.30
 description GUEST
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
!
interface GigabitEthernet0/0.40
 description VOICE
 encapsulation dot1Q 40
 ip address 192.168.40.1 255.255.255.0
!
interface GigabitEthernet0/0.99
 description TRANSIT-TO-DSW1
 encapsulation dot1Q 99
 ip address 10.0.99.2 255.255.255.252
!
ip route 192.168.10.0 255.255.255.0 10.0.99.1
ip route 192.168.20.0 255.255.255.0 10.0.99.1
```

### ASW1 (ASW2 is the same, with PC3 in VLAN 20 and PC4 in VLAN 40)

```
hostname ASW1
!
vlan 10
 name STAFF
vlan 20
 name SERVER
vlan 30
 name GUEST
vlan 40
 name VOICE
vlan 999
 name NATIVE-UNUSED
!
interface GigabitEthernet0/0
 description TRUNK-TO-DSW1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,20,30,40
!
interface GigabitEthernet0/1
 description PC1
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
!
interface GigabitEthernet0/2
 description PC2
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
```

### VPCS

```
PC1> ip 192.168.10.10/24 192.168.10.1
PC2> ip 192.168.30.10/24 192.168.30.1
PC3> ip 192.168.20.10/24 192.168.20.1
PC4> ip 192.168.40.10/24 192.168.40.1
```

---

## Verification

Raw output is saved in [`verification/`](./verification).

| What to prove | Command | Device |
|---------------|---------|--------|
| VLANs and access ports | `show vlan brief` | ASW1, ASW2, DSW1 |
| Trunks, allowed and native VLANs | `show interfaces trunk` | all switches |
| SVIs are up | `show ip interface brief` | DSW1 |
| Sub-interfaces and tags | `show vlans` | R1 |
| Routing tables | `show ip route` | DSW1, R1 |
| CEF entries | `show ip cef 192.168.30.0` | DSW1 |
| Layer 2 rewrite info | `show adjacency detail` | DSW1, R1 |
| End-to-end reachability | `ping`, `trace` | PC1–PC4 |

---

## Results

### Ping matrix

| From \ To | PC1 (V10) | PC2 (V30) | PC3 (V20) | PC4 (V40) |
|-----------|-----------|-----------|-----------|-----------|
| PC1 (V10) | n/a | _ | _ | _ |
| PC2 (V30) | _ | n/a | _ | _ |
| PC3 (V20) | _ | _ | n/a | _ |
| PC4 (V40) | _ | _ | _ | n/a |

> Fill in ✅ / ❌ from actual results.

### Traceroute paths

| Path | Expected hops | Observed |
|------|---------------|----------|
| PC1 → PC3 (SVI only) | 192.168.10.1 → PC3 | _ |
| PC2 → PC4 (router-on-a-stick only) | 192.168.30.1 → PC4 | _ |
| PC1 → PC2 (SVI → transit → R1) | 192.168.10.1 → 10.0.99.2 → PC2 | _ |

---

## Troubleshooting

_Record real problems found during the lab: symptom, cause, fix, and the command that revealed it._

| Symptom | Cause | Fix | Found with |
|---------|-------|-----|------------|
| _ | _ | _ | _ |

---

## What I Learned

- _Difference between SVI and router-on-a-stick in scalability and bandwidth_
- _How CEF and the adjacency table are used to forward packets_
- _Why the native VLAN is moved to an unused VLAN_
