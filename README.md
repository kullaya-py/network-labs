# Network Labs

Hands-on Cisco networking labs built in **GNS3** and **Cisco Packet Tracer**, covering switching, routing, and network segmentation. Each lab includes the topology, IP/VLAN plan, device configurations, verification output, measured results, and troubleshooting notes.

**Author:** Kullaya Sirisom, B.Sc. Cybersecurity, Khon Kaen University
**Focus:** Network infrastructure and network security

---

## Labs

| # | Lab | Key topics | Tools | Status |
|---|-----|-----------|-------|--------|
| 01 | [Inter-VLAN Routing](./01-inter-vlan-routing) | VLAN, 802.1Q, SVI, router-on-a-stick, CEF | GNS3 | 🚧 In progress |
| 02 | [PVST+ to Rapid-PVST Migration](./02-stp-to-rstp) | STP, RSTP, root bridge, failover measurement | GNS3 | 🚧 In progress |
| 03 | [EtherChannel](./03-etherchannel) | Static, PAgP, LACP, load balancing | GNS3 | 🚧 In progress |
| 04 | [VRF-Lite](./04-vrf-lite) | VRF, overlapping IP space, customer isolation | GNS3 | 🚧 In progress |
| 05 | [OSPF + BGP Dual-Stack](./05-ospf-bgp-dual-stack) | OSPFv2/v3, eBGP, MP-BGP, path manipulation, communities | GNS3 | 🚧 In progress |
| 06 | [OTP XOR over 433 MHz RF](./06-iot-rf-otp) | IoT, RF, one-time pad, key handling | Hardware | 🚧 In progress |

> Status legend: ✅ Complete · 🚧 In progress

---

## Lab Structure

Every lab folder follows the same layout:

```
XX-lab-name/
├── README.md          # Objective, topology, plan, results, lessons learned
├── topology.png       # Topology diagram
├── configs/           # Running configs per device (.txt)
└── verification/      # show commands, ping and traceroute output
```

Each lab README covers:

1. **Objective**: what the lab demonstrates
2. **Topology**: diagram and device list
3. **Addressing plan**: VLAN and IP tables
4. **Key configuration**: the commands that matter, with explanations
5. **Verification**: commands used to prove it works
6. **Results**: measured outcomes
7. **Troubleshooting**: problems I hit and how I fixed them
8. **What I learned**

---

## Environment

| Item | Detail |
|------|--------|
| Emulator | GNS3 (version: _fill in_), Cisco Packet Tracer |
| Router image | Cisco IOSv / IOS XE (_fill in version_) |
| Switch image | Cisco IOSvL2 (_fill in version_) |
| Hosts | VPCS |

> Note: Cisco IOS images are not included in this repository. Results measured in an emulator may differ from physical hardware.

---

## Contact

- Email: kullaya.s@kkumail.com
- GitHub: [kullaya-py](https://github.com/kullaya-py)
