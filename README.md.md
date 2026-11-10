# Shopping Mall Network – Cisco Packet Tracer Case Study

**Project:** Design and Implementation of a Secure Shopping Mall Network  
**Simulator:** Cisco Packet Tracer  
**Main devices:** Cisco 2811 Mall Router, Cisco 1941 ISP Router, Cisco 2960 switches, mall server, PCs and wireless access point.

> **Project status note:** This README distinguishes configurations that have been demonstrated in the supplied Packet Tracer screenshots from features that are planned or still need testing. Do not describe a feature as successfully implemented until its verification output or screenshot has been added to this repository.

## 1. Project overview

The case study designs a small, understandable network for a shopping mall with 250 shops, a food court, cinema, parking facilities and security operations. The network uses logical separation so that each department can have its own IP network and access policy. A mall router provides inter-VLAN routing, while an ISP router represents the external network.

The Packet Tracer file uses representative PCs for the departments. It does **not** simulate 250 individual shop terminals; the 250-shop requirement is represented in the addressing and capacity plan.

### Business requirements

- Provide network access for shop POS/billing terminals.
- Separate food court, cinema, parking and security traffic.
- Provide access to central mall services.
- Provide wireless access for authorized users and, in a later stage, isolated public guest Wi-Fi.
- Support controlled access between departments.
- Provide a simulated connection from the mall to an ISP/external network.

## 2. Topology and devices

The logical topology contains:

- **Mall-R1 (Cisco 2811):** default gateways for the department VLANs and routing toward the ISP.
- **ISP-Router (Cisco 1941):** represents the service-provider side of the simulated WAN link.
- **Cisco 2960 switches:** connect the end devices and carry multiple VLANs over an 802.1Q trunk.
- **MALL-SERVER:** intended to host central services such as DNS/HTTP (DNS service must be enabled and tested before it is claimed as complete).
- **Department PCs:** representative endpoints for shops, food court, cinema, parking and security.
- **Wireless access point and laptop:** demonstrate wireless connectivity; SSID/encryption and guest isolation must be verified separately.

Add the final topology screenshot to `screenshots/topology.png` and reference it here:

`![Shopping mall network topology](screenshots/topology.png)`

## 3. Current IP addressing and VLAN design (observed configuration)

The supplied router and switch outputs show five VLANs and five router subinterfaces. The observed subnets are currently all `/24`; therefore, this current table is **not yet a VLSM implementation**.

| VLAN | Department | Network currently configured | Mask | Default gateway |
|---:|---|---|---|---|
| 10 | Shops / POS | `192.168.10.0/24` | `255.255.255.0` | `192.168.10.1` |
| 20 | Food Court | `192.168.20.0/24` | `255.255.255.0` | `192.168.20.1` |
| 30 | Cinema | `192.168.30.0/24` | `255.255.255.0` | `192.168.30.1` |
| 40 | Parking | `192.168.40.0/24` | `255.255.255.0` | `192.168.40.1` |
| 50 | Security | `192.168.50.0/24` | `255.255.255.0` | `192.168.50.1` |
| WAN | Mall–ISP link | `203.0.113.0/30` | `255.255.255.252` | Mall `203.0.113.2`; ISP `203.0.113.1` |

The mall router uses router-on-a-stick: subinterfaces `GigabitEthernet0/0.10`, `.20`, `.30`, `.40` and `.50` act as the VLAN gateways. The switch trunk output showed VLANs 10, 20, 30, 40 and 50 allowed and in the forwarding state on the observed trunk.

## 4. Protocols, algorithms and why they are useful in this mall

This section explains the networking concepts used or intended for the case study. The **status** column is important: it prevents planned features from being mistaken for completed work.

| Protocol / technique | Algorithm or operating method | Mall use | Where configured / how to verify | Current evidence status |
|---|---|---|---|---|
| VLAN (802.1Q switching) | Logical Layer-2 segmentation; switches learn source MAC addresses and forward frames using their MAC table. 802.1Q adds VLAN tags to trunk traffic. | Separates POS, food, cinema, parking and security devices, reducing unnecessary cross-department traffic. | 2960 switch CLI: `show vlan brief`, `show interfaces trunk`. | **Demonstrated.** VLANs 10/20/30/40/50 were shown active; trunk carried all five. |
| Inter-VLAN routing (router-on-a-stick) | Router receives tagged VLAN traffic on one physical link and routes it between subinterfaces. | Allows permitted communication between departments, servers and the ISP while keeping separate VLANs. | Mall-R1 CLI: `show ip interface brief`, `show ip route`. | **Demonstrated.** Five `.10`–`.50` subinterfaces and gateway IPs were shown up/up. |
| DHCP | DORA sequence: Discover, Offer, Request, Acknowledgement. | Automatically assigns IP settings to POS and department devices instead of manually configuring every PC. | Mall-R1 CLI: `show ip dhcp pool`, `show ip dhcp binding`, `show running-config`. | **Configured.** DHCP pools and excluded gateway addresses were shown; verify all pool masks and leases after any subnet change. |
| ACL (extended IP access list) | Ordered, first-match packet filtering using source/destination IP and protocol. | Example policy: restrict security/CCTV network access to POS devices while allowing other traffic as explicitly permitted. | Mall-R1 CLI: `show access-lists`, `show running-config`; also verify the ACL is applied to the intended interface and direction. | **ACL entries demonstrated.** ACL 100 showed a deny from Security to Shops followed by `permit ip any any`; interface application/direction still needs verification. |
| OSPF | Link-state routing; routers exchange link-state information and run Dijkstra's Shortest Path First (SPF) algorithm to calculate routes. | Mall and ISP routers automatically learn department networks. This supports route updates when the mall network changes. | Mall-R1 and ISP-Router CLI: `show ip ospf neighbor`, `show ip route ospf`, `show ip protocols`. | **Working and demonstrated.** ISP showed neighbour `1.1.1.1` in `FULL/BDR` state and five OSPF routes for the mall VLANs. |
| RIP v2 | Distance-vector routing based on the Bellman-Ford approach; uses hop count as its metric (maximum usable distance is 15 hops). | Could be used as a simpler dynamic-routing alternative in a small network. | Both routers: `show ip protocols`, `show ip route rip`. | **Not part of the current final routing design.** RIP was tried earlier and then replaced by the working OSPF configuration on the Mall Router. Do not claim RIP is currently working. |
| NAT/PAT | Network address translation; PAT also tracks transport-layer ports so multiple inside hosts can share one outside IPv4 address. | Allows many private mall devices to share the mall router's ISP-facing address when accessing external services. | Mall-R1 CLI: `show ip nat statistics`, `show ip nat translations`, `show running-config`. | **Not yet verified.** Do not claim it works until an inside-to-outside test creates translations. |
| DNS | Client resolves a hostname using a DNS query; an A record maps a name to an IPv4 address. | Staff can use a name such as `mall.local` rather than memorizing the mall server IP. | MALL-SERVER GUI: Services → DNS; clients: `ping mall.local` or `nslookup mall.local` if supported. | **Not yet verified.** Server is present, but DNS service and records were not shown as enabled/tested. |
| Wireless LAN (IEEE 802.11) | Wireless clients associate with an access point using an SSID and configured authentication/encryption. | Provides wireless access for mall staff and, in a separate isolated network, guests. | Access Point GUI: Config/GUI wireless settings; verify SSID, WPA2-PSK and client association. | **Access point and wireless laptop are present.** Security mode and successful wireless tests still need evidence. |
| VLSM | Variable Length Subnet Masking allocates different prefix lengths according to host requirements. | A larger subnet can be reserved for 250 shops/POS terminals while smaller subnets serve parking or other departments. | Design the subnet table first; then update router subinterfaces, DHCP pools, ACL wildcards and endpoint masks. Verify with `show ip interface brief`, `show ip route` and `show ip dhcp pool`. | **Not implemented in the supplied current configuration.** All five existing VLANs were shown using `/24`. |
| ICMP testing | Ping sends ICMP Echo Request and receives Echo Reply; traceroute increases TTL to reveal routing hops. | Confirms that a POS device reaches its gateway and that remote network paths work. | PC → Desktop → Command Prompt: `ping <IP>` and `tracert <IP>`. | **Gateway ping demonstrated.** Shop-PC1 pinged `192.168.10.1` with 4 replies and 0% loss. Traceroute is pending. |

### 4.1 OSPF configuration evidenced in the project

The Mall Router configuration used OSPF process 1, router ID `1.1.1.1`, area 0, and advertised the five `/24` VLAN networks plus the WAN `/30` link. The ISP Router used router ID `2.2.2.2` and participated on the WAN link.

**Mall-R1 (Cisco 2811)** — use the actual WAN interface name shown in the Packet Tracer device if it differs from `GigabitEthernet0/1`:

```cisco
router ospf 1
 router-id 1.1.1.1
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.20.0 0.0.0.255 area 0
 network 192.168.30.0 0.0.0.255 area 0
 network 192.168.40.0 0.0.0.255 area 0
 network 192.168.50.0 0.0.0.255 area 0
 network 203.0.113.0 0.0.0.3 area 0
 passive-interface default
 no passive-interface GigabitEthernet0/1
```

**ISP-Router (Cisco 1941)**:

```cisco
router ospf 1
 router-id 2.2.2.2
 network 203.0.113.0 0.0.0.3 area 0
```

The supplied verification output showed the ISP router learning these OSPF routes:

```text
O 192.168.10.0/24
O 192.168.20.0/24
O 192.168.30.0/24
O 192.168.40.0/24
O 192.168.50.0/24
```

The neighbour output showed neighbour ID `1.1.1.1`, state `FULL/BDR`, and address `203.0.113.2`. This is the evidence to use in the implementation/results chapter.

### 4.2 DHCP gateway reservations

The following gateway exclusions were entered on Mall-R1:

```cisco
ip dhcp excluded-address 192.168.10.1
ip dhcp excluded-address 192.168.20.1
ip dhcp excluded-address 192.168.30.1
ip dhcp excluded-address 192.168.40.1
ip dhcp excluded-address 192.168.50.1
```

A server address should also be excluded if it is statically assigned from a DHCP-managed subnet. Confirm the server's actual IP before documenting that reservation.

### 4.3 ACL 100 observed in the configuration

The ACL output previously showed the following policy (the wildcard masks below correspond to the observed `/24` networks):

```cisco
access-list 100 deny ip 192.168.50.0 0.0.0.255 192.168.10.0 0.0.0.255
access-list 100 permit ip any any
```

This denies IP traffic from the Security subnet to the Shops/POS subnet, then permits other IP traffic. **The ACL's attachment to an interface and direction must be checked before claiming the policy is actively enforced.**

## 5. Verified test results

| Test | Source | Destination | Observed result | Meaning |
|---|---|---|---|---|
| POS-to-gateway ping | Shop-PC1 | `192.168.10.1` | 4 sent, 4 received, 0% loss | Shop PC can reach its VLAN gateway. |
| Mall-to-ISP physical/IP link | Mall-R1 `203.0.113.2` | ISP `203.0.113.1` | Both connected interfaces displayed up/up in the supplied screenshots; ping result should be captured separately if used as a test claim. | WAN link interfaces are active. |
| OSPF neighbour | ISP-Router | Mall-R1 `1.1.1.1` | `FULL/BDR` | OSPF adjacency established. |
| OSPF learned routes | ISP-Router | Five mall VLAN networks | Five `O` routes displayed | ISP learned mall VLAN routes through OSPF. |

Do not invent NAT, DNS, wireless, VLSM or traceroute results. Add them only after they have been tested and screenshots have been saved.

## 6. Features still to implement or verify

1. **VLSM:** choose and document different subnet sizes. The previously discussed proposal was Shops `/23`, Food `/26`, Cinema `/26`, Parking `/27`, Security `/26`. This is a proposed redesign, not the current running configuration. Update DHCP pools, endpoint masks, ACL wildcard masks and router subinterfaces together if adopting it.
2. **NAT/PAT:** configure inside/outside interfaces, an inside source ACL and PAT overload on the WAN interface. Test by accessing/pinging a reachable external test server and checking the NAT translation table.
3. **DNS:** configure a static server IP, enable Services → DNS, add an A record (for example `mall.local`), give clients the DNS server address, and test name resolution.
4. **Wireless security:** configure an SSID and WPA2-PSK; test client association. Guest Wi-Fi should be separated from POS and security networks if implemented.
5. **Traceroute and ACL tests:** test both permitted and denied communication and record the actual result.

## 7. Evidence and screenshot checklist

Save screenshots in the `screenshots/` folder with clear filenames. Suggested evidence:

| Filename | Evidence to capture | Packet Tracer location/command |
|---|---|---|
| `topology.png` | Full logical topology with labels | Logical workspace |
| `interfaces.png` | Router subinterfaces and WAN interfaces | Mall-R1: `show ip interface brief` |
| `vlans.png` | VLAN membership | Switch: `show vlan brief` |
| `trunk.png` | VLAN trunk status | Switch: `show interfaces trunk` |
| `dhcp-pools.png` | DHCP pool network and lease information | Mall-R1: `show ip dhcp pool` |
| `dhcp-bindings.png` | Active DHCP leases | Mall-R1: `show ip dhcp binding` |
| `acl.png` | ACL entries and counters | Mall-R1: `show access-lists` |
| `ospf-neighbor.png` | OSPF FULL adjacency | Both routers: `show ip ospf neighbor` |
| `ospf-routes.png` | OSPF-learned routes | ISP-Router: `show ip route ospf` |
| `nat-statistics.png` | NAT configuration/counters | Mall-R1: `show ip nat statistics` |
| `nat-translations.png` | Active translations | Mall-R1: `show ip nat translations` |
| `dns-service.png` | DNS service and A record | MALL-SERVER → Services → DNS |
| `dns-test.png` | Hostname resolution test | PC command prompt |
| `wireless-security.png` | SSID and authentication | Access Point configuration |
| `ping-gateway.png` | Successful POS gateway ping | Shop-PC1 command prompt |
| `traceroute.png` | Route path to a remote test host | PC command prompt: `tracert <IP>` |

## 8. Suggested GitHub repository structure

```text
Shopping-Mall-Network-Project/
├── Shopping Mall.pkt
├── README.md
├── report/
│   └── Computer-Networks-Case-Study-Report.docx
├── screenshots/
│   ├── topology.png
│   ├── interfaces.png
│   ├── vlans.png
│   ├── trunk.png
│   ├── dhcp-pools.png
│   ├── acl.png
│   ├── ospf-neighbor.png
│   └── ospf-routes.png
└── configs/
    ├── mall-router-ospf.txt
    └── isp-router-ospf.txt
```

Only add files that exist and correspond to the current project version. Remove any private information or passwords before publishing the repository.

## 9. Conclusion

The current Packet Tracer implementation demonstrates departmental VLAN segmentation, router-on-a-stick inter-VLAN routing, DHCP pools, an extended ACL configuration, a working Mall-to-ISP link, and OSPF route exchange. A successful ping from Shop-PC1 to its gateway confirms basic POS connectivity. NAT/PAT, DNS, VLSM, wireless security and traceroute must be implemented or verified before they can be reported as completed. OSPF is the active dynamic-routing design in the evidence supplied; RIP was considered earlier but is not part of the current working configuration.
