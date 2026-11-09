# Project 05 – DHCP and DNS Configuration

## Project Overview

This project expands the previous VLAN and Inter-VLAN Routing lab by implementing centralized DHCP and DNS services in a small office network using Cisco Packet Tracer.

The objective is to allow devices in separate VLANs to automatically obtain their IP configurations while using a centralized DNS server for hostname resolution.

## Network Topology

**Devices used:**
- 1 Cisco 1941 Router
- 2 Cisco 2960 Switches
- 4 PCs
- 1 Server-PT (DHCP and DNS)

**Network connections:**
- R1 G0/0 → SW1 Gi0/2
- SW1 Gi0/1 → SW2 Gi0/1
- Server → SW1 Fa0/3
- IT-PC01 → SW1 Fa0/1
- HR-PC01 → SW1 Fa0/2
- IT-PC02 → SW2 Fa0/1
- HR-PC02 → SW2 Fa0/2

## VLAN Configuration

| VLAN | Department | Network |
|---|---|---|
| 10 | IT | 192.168.10.0/24 |
| 20 | HR | 192.168.20.0/24 |

Both switches were configured with VLAN 10 and VLAN 20.

Access ports were assigned to their corresponding VLANs, while the inter-switch connection and router connection were configured as trunk links carrying VLANs 10 and 20.

## IP Addressing

### Router – R1

| Interface | IP Address |
|---|---|
| G0/0.10 | 192.168.10.1/24 |
| G0/0.20 | 192.168.20.1/24 |

### DHCP/DNS Server

| Setting | Configuration |
|---|---|
| Device Name | SRV-DNS-DHCP |
| IP Address | 192.168.10.10 |
| Subnet Mask | 255.255.255.0 |
| Default Gateway | 192.168.10.1 |
| Preferred DNS | 192.168.10.10 |

The server uses a static IP address to ensure that DHCP and DNS services remain reachable at a consistent address.

## DHCP Configuration

Two DHCP pools were created on the server.

| Setting | IT_POOL | HR_POOL |
|---|---|---|
| Default Gateway | 192.168.10.1 | 192.168.20.1 |
| DNS Server | 192.168.10.10 | 192.168.10.10 |
| Starting IP | 192.168.10.100 | 192.168.20.100 |
| Subnet Mask | 255.255.255.0 | 255.255.255.0 |

Since the DHCP server is located in VLAN 10, a DHCP relay configuration was required on the router's VLAN 20 subinterface.

```cisco
interface gigabitEthernet 0/0.20
ip helper-address 192.168.10.10
```

This allows DHCP broadcast requests from VLAN 20 to be forwarded to the centralized DHCP server.

## DNS Configuration

The DNS service was enabled on the server.

An A record was created:

| Hostname | IP Address |
|---|---|
| srv-dns-dhcp.bytebuddy.local | 192.168.10.10 |

This allows network clients to resolve the server's hostname into its corresponding IP address.

## Verification and Testing

The following tests were performed:

- Verified that IT clients received DHCP addresses from the IT pool.
- Verified that HR clients received DHCP addresses from the HR pool.
- Confirmed that clients received the correct subnet mask, default gateway, and DNS server.
- Tested DNS hostname resolution using `nslookup`.
- Tested connectivity between IT and HR clients.
- Verified connectivity to the DHCP/DNS server.

### DHCP Lease Results

| Device | Assigned IP Address | VLAN |
|---|---|---|
| IT-PC01 | 192.168.10.100 | 10 |
| IT-PC02 | 192.168.10.101 | 10 |
| HR-PC01 | 192.168.20.100 | 20 |
| HR-PC02 | 192.168.20.101 | 20 |

### DNS Verification

Command used:

```cmd
nslookup srv-dns-dhcp.bytebuddy.local
```

Expected result:

```text
Name:    srv-dns-dhcp.bytebuddy.local
Address: 192.168.10.10
```

DNS resolution was successfully verified from an HR client, demonstrating that clients in a different VLAN could access the centralized DNS service.

## Troubleshooting Notes

During testing, the following issues were encountered:

1. **DNS resolution initially failed:** The first hostname lookup returned a non-existent domain response. Subsequent DNS lookup and hostname ping tests succeeded.
2. **Incorrect ping destination:** An initial test used `192.168.10.11` for IT-PC02. The actual DHCP lease was `192.168.10.101`, explaining the failed ping attempt.
3. **DHCP relay requirement:** The router's `ip helper-address` configuration was necessary to forward DHCP requests from VLAN 20 to the server in VLAN 10.

These troubleshooting steps helped reinforce the importance of checking actual DHCP leases, verifying DNS records, and understanding how DHCP relay works across VLANs.

## Skills Practiced

- VLAN creation and port assignment
- 802.1Q trunk configuration
- Router-on-a-stick inter-VLAN routing
- Centralized DHCP configuration
- DHCP relay using `ip helper-address`
- DNS service configuration
- Static IP addressing for servers
- Network connectivity and service verification
- Troubleshooting DHCP and DNS issues

## Project Outcome

**Status: Completed**

Successfully implemented centralized DHCP and DNS services for a two-VLAN office network.

All four client PCs obtained their IP configurations automatically, DNS hostname resolution was verified, and communication between IT and HR clients was successfully tested.

This project demonstrates the integration of switching, routing, DHCP, and DNS services in a small enterprise-style network.

---
