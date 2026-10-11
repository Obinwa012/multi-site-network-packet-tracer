# Multi-Site Secure Network: Cisco Packet Tracer

![image](docs/screenshots/https://github.com/Obinwa012/multi-site-network-packet-tracer/blob/main/docs/screenshots/Plan.webp)

A secure, multi-site network built in Cisco Packet Tracer connecting a **Main Restaurant** site and a **Grocery Store** site over a dedicated WAN link. Uses router-on-a-stick inter-VLAN routing, per-VLAN DHCP, static WAN routing, and extended ACLs to isolate guest/vendor traffic from all sensitive internal subnets.

## Project Overview and Work Accomplished

- **Multi-site architecture.** Established a secure two-site network in Cisco Packet Tracer. The Main Restaurant site and the Grocery Store site are linked by a dedicated WAN link (10.0.0.0/30) between two Cisco 2911 routers.
- **Router-on-a-stick routing.** Configured 802.1Q trunking and subinterfaces on both routers to support inter-VLAN routing across departments: POS, Staff, Guest/Vendor, IT, Training, plus a payment-system terminal on VLAN 70 (192.168.70.2).
- **Security and guest isolation.** Implemented and refined extended access control lists (ACL 100) applied inbound on the guest/vendor subinterfaces to completely isolate public users from every sensitive internal subnet while keeping general connectivity open.
- **DHCP reservations.** Bound static IP leases for critical POS terminals (192.168.50.10 and 192.168.70.2) directly within the DHCP pools to ensure zero lease expiration conflicts.
- **NAT/PAT configuration.** Configured Network Address Translation on the routers to simulate edge translation for external traffic flows.

## Network Design

### Main Restaurant site

Router: **Main-Router** (2911). Switch: **Main-Switch** (2960-24TT). Trunk on Fa0/1.

| VLAN | Name | Subnet | Gateway (subinterface) | Access ports |
|------|------|--------|------------------------|--------------|
| 10 | RESTAURANT | 192.168.10.0/24 | G0/0.10 192.168.10.1 | Fa0/3 |
| 20 | BAR | 192.168.20.0/24 | G0/0.20 192.168.20.1 | Fa0/4 |
| 30 | HR-MANAGEMENT | 192.168.30.0/24 | G0/0.30 192.168.30.1 | Fa0/5 |
| 40 | TRAINING | 192.168.40.0/24 | G0/0.40 192.168.40.1 | Fa0/2 |
| 60 | GUESTS-VENDORS | 192.168.60.0/24 | G0/0.60 192.168.60.1 | Fa0/6 |
| 70 | PAYMENT-SYSTEM | 192.168.70.0/24 | G0/0.70 192.168.70.1 | Fa0/8 |
| 99 | IT-MANAGEMENT | 192.168.99.0/24 | G0/0.99 192.168.99.1 | Fa0/7 |

### Grocery Store site

Router: **Grocery-Router** (2911). Switch: **Grocery-switch** (2960-24TT). Trunk on Fa0/1.

| VLAN | Name | Subnet | Gateway (subinterface) | Access ports |
|------|------|--------|------------------------|--------------|
| 50 | Grocery-POS | 192.168.50.0/24 | G0/1.50 192.168.50.1 | Fa0/2 |
| 51 | Grocery-Staff | 192.168.51.0/24 | G0/1.51 192.168.51.1 | Fa0/3 |
| 52 | Grocery-Training | 192.168.52.0/24 | G0/1.52 192.168.52.1 | Fa0/4 |
| 53 | Grocery-Guests | 192.168.53.0/24 | G0/1.53 192.168.53.1 | Fa0/5 |
| 54 | Grocery-IT | 192.168.54.0/24 | G0/1.54 192.168.54.1 | Fa0/6 |

### WAN and routing

| Link | Detail |
|------|--------|
| WAN subnet | 10.0.0.0/30 |
| Main-Router G0/1 | 10.0.0.1 |
| Grocery-Router G0/0 | 10.0.0.2 |
| Grocery-Router | Default route `0.0.0.0/0` via 10.0.0.1 |
| Main-Router | Static route `192.168.50.0/24` via 10.0.0.2 |

### Guest isolation ACLs (ACL 100, applied inbound)

- **Grocery-Router, G0/1.53:** denies 192.168.53.0 from reaching 192.168.10.0, 192.168.20.0, 192.168.30.0, 192.168.40.0, 192.168.70.0, 192.168.50.0, 192.168.51.0, and 192.168.52.0; then `permit ip any any`.
- **Main-Router, G0/0.60:** denies 192.168.60.0 from reaching the same destinations; then `permit ip any any`.

## Build Walkthrough

### 1. Main site LAN

The main site starts with Main-Router and Main-Switch serving Restaurant Staff, Bar Staff, HR/Management, Training PC, Guests/Vendors, and IT.

![Main site topology: Main-Router connected to Main-Switch with six PCs and a laptop](docs/screenshots/01-main-site-topology.png)

Switch ports are up and trunking to the router.

![Main-Switch interfaces status showing trunk on Fa0/1 and connected access ports](docs/screenshots/02-main-switch-interfaces-status.png)

The router hands out DHCP leases across the staff VLANs.

![Main-Router DHCP bindings for the main site VLANs](docs/screenshots/03-main-router-dhcp-bindings.png)

### 2. WAN link between sites

Grocery-Router comes up with its WAN-facing interface enabled.

![Grocery-Router WAN interface G0/1 brought up](docs/screenshots/04-grocery-router-wan-interface-up.png)

Main-Router gets 10.0.0.1/30 on G0/1, and the dashed WAN link joins the two routers.

![Main-Router WAN configuration: 10.0.0.1/30 on G0/1](docs/screenshots/05-main-router-wan-config.png)

![Topology showing the WAN link between Main-Router and Grocery-Router](docs/screenshots/06-wan-topology-grocery-added.png)

### 3. Grocery VLANs and DHCP

VLAN 50 is created on Grocery-Router as subinterface G0/1.50 with gateway 192.168.50.1.

![Grocery-Router subinterface G0/1.50 configuration with dot1Q encapsulation for VLAN 50](docs/screenshots/07-grocery-router-vlan50-subinterface.png)

On the switch side, the VLAN is named GROCERY-LAN, Fa0/1 trunks, and Fa0/2 is assigned as an access port.

![Grocery-switch VLAN 50 creation and Fa0/1 trunk configuration](docs/screenshots/08-grocery-switch-vlan50-trunk.png)

The DHCP pool for 192.168.50.0/24 is created with the gateway excluded, and a grocery PC picks up 192.168.50.2 via DHCP.

![Grocery-Router DHCP pool for 192.168.50.0/24](docs/screenshots/09-grocery-router-dhcp-pool.png)

![Grocery PC receiving 192.168.50.2 via DHCP](docs/screenshots/10-grocery-pc-dhcp-client.png)

### 4. Inter-site routing

Before routing was in place, a grocery PC pinging the main site (192.168.40.2) got "Destination host unreachable" from its gateway.

![Failed ping from grocery PC to the main site before routing existed](docs/screenshots/11-grocery-pc-inter-site-ping-unreachable.png)

Routing was fixed with a default route on Grocery-Router via 10.0.0.1, verified with a successful WAN ping, and a static route on Main-Router for 192.168.50.0/24 via 10.0.0.2.

![Grocery-Router default route via 10.0.0.1 and successful WAN ping](docs/screenshots/12-grocery-router-default-route.png)

![Main-Router static route to 192.168.50.0/24 via 10.0.0.2](docs/screenshots/13-main-router-static-route-to-grocery.png)

The full two-site topology with all endpoints:

![Full two-site topology with all endpoints](docs/screenshots/14-full-topology-both-sites.png)

### 5. Grocery VLAN verification

All five grocery VLANs are active on the switch, all five router subinterfaces are up, and DHCP leases exist on every grocery subnet.

![Grocery-switch VLAN brief showing VLANs 50 through 54](docs/screenshots/15-grocery-switch-vlan-brief.png)

![Grocery-Router subinterfaces all up with their gateway addresses](docs/screenshots/16-grocery-router-subinterfaces.png)

![Grocery-Router DHCP bindings across all grocery subnets](docs/screenshots/17-grocery-router-dhcp-bindings.png)

Connectivity was confirmed end to end: Grocery IT reaches the main site IT VLAN and its own staff subnet.

![Grocery IT ping tests to the main site and local staff subnet](docs/screenshots/18-grocery-it-ping-tests.png)

The router ARP table shows live entries across the WAN and all grocery subinterfaces.

![Grocery-Router ARP table with live entries](docs/screenshots/19-grocery-router-arp-table.png)

And Main IT pings Grocery IT at 192.168.54.2 with 0% loss.

![Main IT pinging Grocery IT across the WAN](docs/screenshots/20-main-it-ping-grocery-it.png)

### 6. Guest isolation on the grocery site

The first ACL draft was mistakenly written against the training VLAN (.52).

![ACL 100 mistakenly targeting the training VLAN](docs/screenshots/21-grocery-acl-100-misapplied-to-vlan52.png)

It was removed and rebuilt for the guest/vendor VLAN (.53), then applied inbound on G0/1.53.

![ACL 100 rebuilt for the guest/vendor VLAN 53](docs/screenshots/22-grocery-acl-rebuilt-for-vlan53.png)

![ACL 100 applied inbound on subinterface G0/1.53](docs/screenshots/23-grocery-acl-53-applied-inbound.png)

A grocery guest pinging the POS terminal at 192.168.50.2 is now blocked with "Destination host unreachable".

![Grocery guest blocked from reaching the POS terminal](docs/screenshots/24-grocery-guest-blocked-from-pos.png)

### 7. Guest isolation on the main site

Before the main-site ACL, a main-site guest could still reach the grocery POS.

![Main-site guest successfully reaching the grocery POS before the ACL](docs/screenshots/25-main-guest-reaches-grocery-pos.png)

ACL 100 was then built on Main-Router denying 192.168.60.0 to all sensitive subnets.

![Main-Router ACL 100 denying guest VLAN 60 to internal subnets](docs/screenshots/26-main-router-acl-100-guest-vlan60.png)

Verification shows guests blocked from POS and restaurant subnets while IT stays reachable, and the final ACL covers all internal destinations.

![Main-site guest isolation verified: blocked from POS and restaurant, IT reachable](docs/screenshots/27-main-guest-isolation-verified.png)

![Main-Router full ACL 100 covering all internal destinations](docs/screenshots/28-main-router-acl-100-full.png)

### 8. Final grocery ACL and confirmation

The grocery guest ACL was extended to cover every sensitive subnet on both sites.

![Grocery-Router final ACL 100 covering all sensitive subnets](docs/screenshots/29-grocery-router-acl-53-full.png)

Live tests confirm a grocery guest reaches its own gateway peers but is blocked from restaurant POS and management subnets.

![Grocery guest verification tests](docs/screenshots/30-grocery-guest-verification-tests.png)

The ACL match counters show the deny entries actively filtering traffic.

![ACL 100 match counters showing denied traffic](docs/screenshots/31-grocery-acl-match-counters.png)

Final topology:

![Final two-site topology](docs/screenshots/32-final-topology.png)

### 9. Scope expansion and main-site finishing touches

The payment-system terminal (VLAN 70, gateway 192.168.70.1) and IT management (VLAN 99, gateway 192.168.99.1) subinterfaces were added on Main-Router, and gateway addresses were excluded from DHCP.

![Main-Router subinterfaces for VLANs 60 and 99](docs/screenshots/33-main-router-vlan60-99-subinterfaces.png)

![Main-Router DHCP exclusions for gateway addresses](docs/screenshots/34-main-router-dhcp-exclusions.png)

The main switch VLAN table carries all names and access-port assignments.

![Main-Switch VLAN brief](docs/screenshots/35-main-switch-vlan-brief.png)

The Fa0/1 trunk carries VLANs 1, 10, 20, 30, 40, 60, and 99.

![Main-Switch trunk status on Fa0/1](docs/screenshots/36-main-switch-trunk-config.png)

A main-site PC gets 192.168.40.2 via DHCP and pings its gateway, and the guest (Fa0/6, VLAN 60) and IT (Fa0/7, VLAN 99) access ports are assigned.

![Main-site PC DHCP client receiving 192.168.40.2](docs/screenshots/37-main-pc-dhcp-client.png)

![Main-site PC pinging its gateway](docs/screenshots/38-main-pc-ping-gateway.png)

![Main-Switch guest and IT access port assignments](docs/screenshots/39-main-switch-access-ports.png)

### 10. Final verification with the main-site POS

The final topology includes the main-site POS terminal alongside the existing endpoints.

![Final topology with the main-site POS terminal](docs/screenshots/40-final-topology-with-pos.png)

VLAN 70 is confirmed as PAYMENT-SYSTEM on the main switch.

![Main-Switch VLAN table showing VLAN 70 as PAYMENT-SYSTEM](docs/screenshots/41-main-switch-vlan-payment-system.png)

The POS terminal holds its DHCP lease at 192.168.70.2.

![Main-Router DHCP bindings including the 192.168.70.2 POS lease](docs/screenshots/42-main-router-dhcp-with-pos-lease.png)

The grocery switch VLAN table and the grocery router's final ACL and DHCP state confirm guest isolation remains in place.

![Grocery-switch final VLAN table](docs/screenshots/43-grocery-switch-vlan-brief-final.png)

![Grocery-Router final ACL and DHCP bindings](docs/screenshots/44-grocery-router-acl-dhcp-final.png)

## Challenges Met

- **Unassigned subinterface states.** Newly created subinterfaces initially showed disabled IP processing flags until their respective gateway IP addresses were explicitly assigned.
- **Misaligned VLAN targeting.** The guest security restriction was accidentally applied to the training VLAN (.52) instead of the intended guest/vendor VLAN (.53), requiring careful reversal and re-application (see step 6 above).
- **Scope expansion for new subnets.** As new sensitive endpoints (such as the payment-system terminal on VLAN 70) were introduced, the access control lists required iterative updates to ensure total isolation from public traffic zones.

## Lessons Learnt

- **Interface mapping precision.** Verifying exact subinterface numbers and mapping them accurately to correct VLAN assignments before applying security policies saves troubleshooting time.
- **Layered verification.** Combining status checks (`show ip interface`), access list statistics (`show access-lists`), and live end-to-end ping testing ensures that security boundaries are active and working precisely as intended.

## Packet Tracer file

The complete topology source file is at the repo root: `Store_Network_Recreating.pkt` — open it directly in Cisco Packet Tracer. Screenshots documenting the build live in `docs/screenshots/`.
