# Testing & Evidence — Completed Packet Tracer Lab

This table records results **visible in the uploaded screenshots**, not assumptions based only on running-config. All listed files are included under `screenshots/`. In particular, a **blocked** guest-to-internal ping is an intentional and successful access-control test.

## Reproduce tests

1. Open [`COFFEE.pkt`](COFFEE.pkt) in Cisco Packet Tracer.
2. On network devices, use the **CLI** tab. On PCs/laptops, use **Desktop → Command Prompt** or **Desktop → IP Configuration**.
3. Console and enable lab passwords: `Cisco123`. Switch SSH: username `admin`, password `Cisco123`.
4. Run these commands; allow for initial ARP convergence if a first ping times out.

## Verification matrix

| ID | Source | Exact command / action | Observed result | Evidence | Verdict |
|---|---|---|---|---|---|
| 01 | Switch | `show vlan brief` | VLANs 10/20/30/99; access ports assigned | [01-switch-vlans.png](screenshots/01-switch-vlans.png) | PASS (configuration) |
| 02 | Switch | `show interfaces trunk` | `Gi0/1` 802.1Q trunk, allowed/forwarding 10,20,30,99 | [02-switch-trunk.png](screenshots/02-switch-trunk.png) | PASS |
| 03 | Switch | `show running-config` (Vlan99 area) | `192.168.99.2/24` and `ip default-gateway 192.168.99.1` | [03-switch-management.png](screenshots/03-switch-management.png) | PASS (configuration) |
| 04 | Router | `show ip interface brief` | `Gi0/1.10`, `.20`, `.30`, `.99` up/up | [04-router-subinterfaces.png](screenshots/04-router-subinterfaces.png) | PASS |
| 05 | Router | `show ip dhcp binding` | Leases for `.10.21`, `.20.21`, `.30.21`, `.30.22` | [05-router-dhcp-bindings.png](screenshots/05-router-dhcp-bindings.png) | PASS |
| 06 | Router | `show access-lists`; `show ip interface GigabitEthernet0/1.30` | `GUEST_RESTRICTIONS` exists and is applied inbound | [06-router-guest-acl.png](screenshots/06-router-guest-acl.png) | PASS (configuration) |
| 07 | Office PC1 | `ipconfig`; `ping 192.168.10.1` | `192.168.10.21/24`, gateway `.10.1`, 4/4 replies, 0% loss | [07-office-gateway-ping.png](screenshots/07-office-gateway-ping.png) | PASS |
| 08 | POS PC2 | `ipconfig`; `ping 192.168.20.1` | `192.168.20.21/24`, gateway `.20.1`, 4/4 replies, 0% loss | [08-pos-gateway-ping.png](screenshots/08-pos-gateway-ping.png) | PASS |
| 09a | Guest laptop | Desktop → IP Configuration → DHCP | `192.168.30.22/24`, gateway `.30.1`, DNS `8.8.8.8` | [09-guest-DHCP_CONF.png](screenshots/09-guest-DHCP_CONF.png) | PASS |
| 09b | Guest laptop | `ping 192.168.30.1` | 4/4 replies, 0% loss | [09-guest-gateway-ping.png](screenshots/09-guest-gateway-ping.png) | PASS |
| 10 | Guest laptop | `ping 192.168.10.1`; `ping 192.168.20.1`; `ping 192.168.99.1` | Destination host unreachable from `.30.1` for each internal gateway (user stopped pings early) | [10-guest-isolation.png](screenshots/10-guest-isolation.png) | PASS — expected denial |
| 11 | Office PC1 | `ping 192.168.20.21` | 4/4 replies, 0% loss, confirms office → POS | [11-office-to-pos-ping.png](screenshots/11-office-to-pos-ping.png) | PASS |
| 12 | Office PC | `ssh -l admin 192.168.99.2` | Authenticated console shows `CoffeeShop-SW#` | [12-switch-ssh.png](screenshots/12-switch-ssh.png) | PASS |
| 13 | Switch | `ping 192.168.99.1` | `!!!!!`, 5/5 replies, 100% | [13-switch-management-ping.png](screenshots/13-switch-management-ping.png) | PASS |

## Interpreting the guest security test

1. First verify a guest can communicate with its **own** gateway (`09-guest-gateway-ping.png`). That succeeds.
2. Then verify a guest cannot ping the **office, POS or management gateways** (`10-guest-isolation.png`). They report `Destination host unreachable` from `192.168.30.1`.
3. Confirm the named extended ACL is bound inbound to router `Gi0/1.30` (`06-router-guest-acl.png`).

This is strong evidence that the router is isolating guest-originated traffic from the specified internal destinations. The deny-rule packet counters are **not visibly incremented** in the ACL screenshot, so don't claim that the screenshot records deny counter hits. The ACL does not block office-initiated access to the POS network; see successful test 11.

## Limitations

- **No working WAN/internet demonstration:** router `Gi0/0` is administratively down. No web browsing, external DNS reachability or NAT test is evidenced.
- **SSID/Wi-Fi password unspecified:** screenshots show a guest laptop obtained DHCP, but do not document access-point security settings.
- **Security scope:** this is a classroom/simulation design. The use of `Cisco123` is intentionally public only for the lab; do not reuse in deployed equipment.
- Gateway ping results are **not equivalent** to successful printer-specific connectivity tests; those are out of scope of the supplied screenshots.
